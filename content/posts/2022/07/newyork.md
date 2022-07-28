+++
title = "New York City Airbnb Open Data Analysis"
date = 2022-07-28T14:23:00+08:00
lastmod = 2022-07-28T14:24:54+08:00
tags = ["数据分析"]
categories = ["Julia"]
draft = false
toc = true
+++

## New York City Airbnb Open Data Analysis {#new-york-city-airbnb-open-data-analysis}

以下流程参考自 <https://www.kaggle.com/code/chirag9073/airbnb-analysis-visualization-and-prediction> <br/>


### 导入库 {#导入库}

```julia
using MLJFlux, Flux, MLJ, DataFrames, CSV, StatsBase
using WordCloud
```


### 加载数据 {#加载数据}

```julia
origindata = CSV.read("data/newyork-city-airbnb-open-data/AB_NYC_2019.csv", DataFrame)

```


### 观察数据 {#观察数据}

你可以像教程那样 <br/>

{{< figure src="/ox-hugo/2022-07-26_18-23-18_screenshot.png" >}} <br/>

{{< figure src="/ox-hugo/2022-07-26_18-25-50_screenshot.png" >}} <br/>

也可以，像我一样，用 **excel** 打开 csv 文件 <br/>

![](/ox-hugo/2022-07-26_18-27-06_screenshot.png) <br/>
我写了一个表格，记录我观察到的结果 <br/>

| column                         | missing count | type     | type coerce                                  | fill/drop |
|--------------------------------|---------------|----------|----------------------------------------------|-----------|
| id                             | 0             | Int      | Count =&gt; Continuous                       | None      |
| name                           | 16            | String?  | Multiclass                                   | Drop      |
| host_id                        | 0             | Int      | Count =&gt; Continuous                       | None      |
| host_name                      | 21            | String?  | Multiclass                                   | Drop      |
| neighbourhood_group            | 0             | String15 | Multiclass =&gt; Count =&gt; Continuous      | None      |
| neighbourhood                  | 0             | String31 | Multiclass =&gt; Count =&gt; Continuous      | None      |
| latitude                       | 0             | Float64  | Continuous                                   | None      |
| longitude                      | 0             | Float64  | Continuous =&gt; Multiclass =&gt; Continuous | None      |
| room_type                      | 0             | String15 | Multiclass =&gt; Count =&gt; Continuous      | None      |
| price                          | 0             | Int      | Count =&gt; Continuous                       | None      |
| minimum_nights                 | 0             | Int      | Count =&gt; Continuous                       | None      |
| number_of_reviews              | 0             | Int      | Count =&gt; Continuous                       | None      |
| last_review                    | 10052         | Date?    | Date =&gt; Count =&gt; Continuous ?          | Drop      |
| reviews_per_month              | 10052         | Float64? | Continuous                                   | Drop      |
| calculated_host_listings_count | 0             | Int      | Count =&gt; Continuous                       | None      |
| availability_365               | 0             | Int      | Count =&gt; Continuous                       | None      |

你可以用这段代码来观察 missing 的数据量 <br/>

```julia
for column in names(origindata)
  _count = count(ismissing, origindata[!, column])
  println("$column: missing $_count data")
end
```


### 数据清洗 {#数据清洗}

基于上述数据观察，我们这样确定清洗流程， <br/>
首先我们选择抛弃的特征 <br/>

```julia
featureSelector = FeatureSelector(
  features = [:id, :name, :host_name, :last_review],
  ignore = true
)
```

**:last_review** 字段已被抛弃，有相似的字段 **:reviews_per_month** 存在过多缺失值，这里决定丢弃缺失的行 <br/>

```julia
dropMissing(dataframe::DataFrame) = begin
  dropmissing(dataframe, :reviews_per_month)
end
```

**:longitude** 字段我们发现，他的数值在 -74, -75 上下，我们把他记为 1 和 2 <br/>

```julia
processLongitude(dataframe::DataFrame) = begin
  dataframe[!, :longitude] = map(floor, dataframe[!, :longitude])
  array = unique(dataframe[!, :longitude])
  dict = Dict{Float64, Float64}()
  for (index, value) in Iterators.enumerate(array)
    dict[value] = index
  end

  dataframe[!, :longitude] = map(x -> dict[x], dataframe[!, :longitude])
  return dataframe
end
```

**:neighbourhood_group** 字段有多个重复的值，我们将其进行编码 <br/>

```julia
processNeighbourhoodGroup(dataframe::DataFrame) = begin
  array = unique(dataframe[!, :neighbourhood_group])
  dict = Dict{String, Int}()
  for (index, value) in Iterators.enumerate(array)
    dict[value] = index
  end

  dataframe[!, :neighbourhood_group] = map(x -> dict[x], dataframe[!, :neighbourhood_group])

  return dataframe
end
```

**:neighbourhood** 和 **:room_type** 也是类似的 <br/>

```julia
processNeighbourhood(dataframe::DataFrame) = begin
  array = unique(dataframe[!, :neighbourhood])

  dict = Dict{String, Int}()
  for (index, value) in Iterators.enumerate(array)
    dict[value] = index
  end

  dataframe[!, :neighbourhood] = map(x -> dict[x], dataframe[!, :neighbourhood])

  return dataframe
end

processRoomType(dataframe::DataFrame) = begin
  array = unique(dataframe[!, :room_type])
  dict = Dict{String, Int}()
  for (index, value) in Iterators.enumerate(array)
    dict[value] = index
  end

  dataframe[!, :room_type] = map(x -> dict[x], dataframe[!, :room_type])

  return dataframe
end
```

别忘了将科学类型 **Count** 改为 科学类型 **Continuous** <br/>

```julia
coerceCount(dataframe::DataFrame) = begin
  coerce(dataframe, Count => Continuous)
end
```

最后转换数据 <br/>

```julia
transformModel = Pipeline(
  featureSelector,
  dropMissing,
  processLongitude,
  processNeighbourhoodGroup,
  processNeighbourhood,
  processRoomType,
  coerceCount
)

transformMachine = machine(transformModel, origindata)
fit!(transformMachine)
transformedData = MLJ.transform(transformMachine, origindata)
```


### 数据可视化 {#数据可视化}

别忘了导入库和设置 plot 后端 <br/>

```julia
using Plots, StatsPlots
plotly()

```


#### Plotting all neighbourhood group {#plotting-all-neighbourhood-group}

```julia
let 
  counts = countmap(origindata[!, :neighbourhood_group])
  bar(collect(keys(counts)), collect(values(counts)),
      title = "Neighbourhood Group") |> display
end
```

{{< figure src="/ox-hugo/2022-07-26_18-43-21_screenshot.png" >}} <br/>


#### Plotting neighbourhood {#plotting-neighbourhood}

```julia
let
  counts = countmap(origindata[!, :neighbourhood])
  bar(collect(keys(counts)), collect(values(counts)),
      xrotation = -90,
      xticks = :all,
      size = (1920, 1680),
      title = "Neighbourhood") |> display
end

```

{{< figure src="/ox-hugo/2022-07-26_18-44-25_screenshot.png" >}} <br/>


#### Plotting room type {#plotting-room-type}

```julia
let 
  counts = countmap(origindata[!, :room_type])
  bar(collect(keys(counts)), collect(values(counts))) |> display
end

```

{{< figure src="/ox-hugo/2022-07-26_18-45-28_screenshot.png" >}} <br/>


#### Plotting relation between neighbourhood_group and availability_365 of room {#plotting-relation-between-neighbourhood-group-and-availability-365-of-room}

```julia
let
  x = origindata[!, :neighbourhood_group]
  y = origindata[!, :availability_365]
  boxplot(x, y) |> display
end

```

{{< figure src="/ox-hugo/2022-07-26_18-46-29_screenshot.png" >}} <br/>


#### Plotting map of neighbourhood_group {#plotting-map-of-neighbourhood-group}

```julia
let
  array = unique(origindata[!, :neighbourhood_group])
  colors = [:red, :green, :blue, :black, :yellow]
  dict = Dict{String, Symbol}()

  for (index, value) in Iterators.enumerate(array)
    dict[value] = colors[index]
  end

  markercolors = map(x -> dict[x], origindata[!, :neighbourhood_group])
  scatter(origindata[!, :longitude], origindata[!, :latitude],
	  markercolor = markercolors,
	  size = figuresize) |> display
end

```

{{< figure src="/ox-hugo/2022-07-26_18-47-39_screenshot.png" >}} <br/>


#### Plotting map of neighbourhood {#plotting-map-of-neighbourhood}

```julia
let
  array = unique(origindata[!, :room_type])
  colors = [:red, :green, :blue]
  dict = Dict{String, Symbol}()
  for (index, value) in Iterators.enumerate(array)
    dict[value] = colors[index]
  end

  markercolors = map(x -> dict[x], origindata[!, :room_type])
  scatter(origindata[!, :longitude], origindata[!, :latitude],
	  markercolor = markercolors,
	  size = (1980, 1600)) |> display
end
```

{{< figure src="/ox-hugo/2022-07-26_18-52-20_screenshot.png" >}} <br/>


#### Plotting availability of room {#plotting-availability-of-room}

```julia
let
  mapcolor(number::Number) = begin
    if number >= 0 && number < 150
      return :red
    elseif number >= 150 && number < 300
      return :green
    elseif number >= 300 && number < 450
      return :blue
    else
      return :black
    end
  end

  markercolors = map(mapcolor, origindata[!, :availability_365])
  scatter(origindata[!, :longitude], origindata[!, :latitude],
	  markercolor = markercolors,
	  size = figuresize |> display
end
```

{{< figure src="/ox-hugo/2022-07-26_18-54-09_screenshot.png" >}} <br/>


#### Word Cloud {#word-cloud}

```julia
using WordCloud
wc = wordcloud(origindata[!, :neighbourhood]) |> generate!
paint(wc, "/home/steiner/Downloads/neighbourhood.png")
```

{{< figure src="/ox-hugo/2022-07-26_18-55-04_screenshot.png" >}} <br/>