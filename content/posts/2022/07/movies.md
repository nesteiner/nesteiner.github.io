+++
title = "数据可视化 电影数据分析"
date = 2022-07-28T14:23:00+08:00
lastmod = 2022-07-28T14:24:44+08:00
tags = ["数据分析"]
categories = ["Julia"]
draft = false
toc = true
+++

## 数据可视化 电影数据分析 {#数据可视化-电影数据分析}


### 说明 {#说明}

这次的文章参考自 <https://zhuanlan.zhihu.com/p/35453189> <br/>


### 提出问题 {#提出问题}

简单分析制作一部电影时，应考虑哪些客观因素才能使电影成功? 为客户提供有效的数据一句，以便作出更准确 <br/>
的决策 <br/>
本次数据分析报告主要围绕一下几点进行分析 <br/>

1.  问题一：电影类型如何随时间发生变化？ <br/>
    -   电影数量上的对比 <br/>
    -   电影收入的对比 <br/>

2.  问题二：影响电影收入的客观因素有哪些？ **这个不会，先放一放** <br/>

3.  问题三：两家电影公司Universal Pictures 和 Paramount Pictures之间的对比。 <br/>
    -   电影发行数量上的对比 <br/>
    -   电影产生的利润对比 <br/>

4.  问题四：改编电影和原创电影之间的对比。 <br/>
    -   电影发行数量上的对比 <br/>
    -   电影产生的利润对比 <br/>


### 理解数据 {#理解数据}

重点关注的变量有： <br/>

-   `imdb_id` IMDB 标识号 <br/>
-   `popularity` 在 Movie Database 上的相对页面查看次数 <br/>
-   `budget` 预算（美元） <br/>
-   `revenue` 收入（美元） <br/>
-   `original_title` 电影名称 <br/>
-   `cast` 演员列表，按 | 分隔，最多 5 名演员 <br/>
-   `homepage` 电影首页的 URL <br/>
-   `director` 导演列表，按 | 分隔，最多 5 名导演 <br/>
-   `tagline` 电影的标语 <br/>
-   `keywords` 与电影相关的关键字，按 | 分隔，最多 5 个关键字 <br/>
-   `overview` 剧情摘要 <br/>
-   `runtime` 电影时长 <br/>
-   `genres` 风格列表，按 | 分隔，最多 5 种风格 <br/>
-   `production_companies` 制作公司列表，按 | 分隔，最多 5 家公司 <br/>
-   `release_date` 首次上映日期 <br/>
-   `vote_count` 评分次数 <br/>
-   `vote_average` 平均评分 <br/>
-   `release_year` 发行年份 <br/>
-   `budget_adj` 根据通货膨胀调整的预算（2010 年，美元） <br/>
-   `revenue_adj` 根据通货膨胀调整的收入（2010 年，美元） <br/>


### 导入数据 {#导入数据}

```julia
using MLJFlux, Flux, MLJ, DataFrames, CSV, StatsBase, Dates
import JSON
creditsdata = CSV.read("data/movies/tmdb_5000_credits.csv", DataFrame)
moviesdata = CSV.read("data/movies/tmdb_5000_movies.csv", DataFrame)

select!(creditsdata, Not(:title))
fulldata = hcat(creditsdata, moviesdata)

```

使用 excel 查看数据 <br/>

{{< figure src="/ox-hugo/2022-07-28_13-45-25_screenshot.png" >}} <br/>

查看缺失值情况 <br/>

```julia
for column in names(fulldata)
  missingcount = count(ismissing, fulldata[!, column])
  println("$column: \t $missingcount")
end
```

发现只有以下字段有缺失值 <br/>

-   `release_date` <br/>
-   `runtime` <br/>


### 数据清洗 {#数据清洗}


#### 选择子集 {#选择子集}

```julia
columns = [:id, :title, :vote_average, :production_companies, :genres,
	   :release_date, :keywords, :runtime, :budget, :revenue, :vote_count, :popularity]
fulldata = select(fulldata, columns)

```


#### 缺失值处理 {#缺失值处理}

```julia
fillReleaseDate(dataframe::DataFrame) = begin
  mapdate(date::Union{Missing, Date}) = begin
    if ismissing(date)
      return Date("2014-06-01")
    else
      return date
    end
  end

  dataframe[!, :release_date] = map(mapdate, dataframe[!, :release_date])
  return dataframe
end

fillRuntime(dataframe::DataFrame) = begin
  meanvalue = mean(skipmissing(dataframe[!, :runtime]))
  mapruntime(runtime::Union{Missing, Float64}) = begin
    if ismissing(runtime)
      return meanvalue
    else
      return runtime
    end
  end

  dataframe[!, :runtime] = map(mapruntime, dataframe[!, :runtime])
  return dataframe
end

```


#### 数据类型转换 {#数据类型转换}

<!--list-separator-->

-  在时间序列中提取年份

    ```julia
    generateReleaseYear(dataframe::DataFrame) = begin
      dataframe[!, :release_year] = map(year, dataframe[!, :release_date])
      return dataframe
    end
    
    ```

<!--list-separator-->

-  将电影类型添加到列，需进行one-hot编码

    ```julia
    function generateGenreType(dataframe::DataFrame)
      len = first(size(dataframe))
    
      for column in genrelist
        dataframe[!, column] = zeros(len)
        for row in eachrow(dataframe)
          if contains(row.genres, column)
    	row[column] = 1
          end
        end
      end
    
      return dataframe
    end
    ```

<!--list-separator-->

-  用年份索引

    ```julia
    function sortByReleaseYear(dataframe::DataFrame)
      sort(dataframe, [:release_year])
    end
    
    ```


#### 数据转换 {#数据转换}

```julia
featureSelector = FeatureSelector(
  features = [:release_date],
  ignore = true
)

transformModel = Pipeline(
  fillReleaseDate,
  fillRuntime,
  generateReleaseYear,
  featureSelector,
  generateGenreType,
  sortByReleaseYear
  # generateName
)

transformMachine = machine(transformModel, fulldata)

fit!(transformMachine)
transformedData = MLJ.transform(transformMachine, fulldata)

```


### 问题分析 {#问题分析}


#### 电影类型随时间变化 {#电影类型随时间变化}

<!--list-separator-->

-  提取电影类型

    ```julia
    function fetchGenreList(dataframe::DataFrame)
      mapfn(array::Vector{Any}) = map(x -> x["name"], array)
      genrelist = Set{String}()
      jsons = map(JSON.parse, dataframe[!, :genres])
      for json in jsons
        names = mapfn(json)
        for name in names
          push!(genrelist, name)
        end
      end
      return genrelist
    end
    
    const genrelist = fetchGenreList(fulldata)
    
    ```

<!--list-separator-->

-  对每个类型的电影按年份求和

    ```julia
    function groupByReleaseYear(dataframe::DataFrame)
      dataframes = groupby(dataframe, :release_year)
      years = Int[]
      counts = Int[]
      for _dataframe in dataframes
        year = first(_dataframe.release_year)
        count = first(size(_dataframe))
    
        push!(years, year)
        push!(counts, count)
      end
    
      bar(years, counts, xticks = :all, size = figuresize) |> display
    end
    
    groupByReleaseYear(transformedData)
    
    ```
    
    {{< figure src="/ox-hugo/2022-07-28_14-05-28_screenshot.png" >}} <br/>

<!--list-separator-->

-  汇总各电影类型的总量

    ```julia
    function groupByEachGenre(dataframe::DataFrame)
    
      record = Dict{String, Int}()
      for genre in genrelist
        record[genre] = 0
      end
    
      for row in eachrow(dataframe)
        for genre in genrelist
          record[genre] += row[genre]
        end
      end
    
      xs = collect(keys(record))
      ys = collect(values(record))
    
      bar(xs, ys, xticks = :all, size = figuresize) |> display
    end
    
    groupByEachGenre(transformedData)
    
    ```
    
    {{< figure src="/ox-hugo/2022-07-28_14-07-31_screenshot.png" >}} <br/>

<!--list-separator-->

-  电影类型随时间的变化

    ```julia
    function plotGenreAndTime(dataframe::DataFrame)
      columns = ["Drama","Comedy","Thriller","Action","Romance","Adventure",
    	     "Crime","Science Fiction","Horror","Family", "release_year"]
      _dataframes = groupby(select(dataframe, columns), :release_year)
      # p = plot()
    
      # record: Dict{year, Dict{Name, Count}}
      record = Dict{Int, Dict{String, Int}}()
      for _dataframe in _dataframes
        # years
        # counts
        year = first(_dataframe.release_year)
        record[year] = Dict{String, Int}()
        for column in columns[columns .!= "release_year"]
          record[year][column] = reduce(+, _dataframe[!, column])
        end
      end
    
      _years = collect(keys(record))
      _countmaps = collect(values(record))
      indexs = sortperm(_years)
    
      years = _years[indexs]
      countmaps = _countmaps[indexs]
    
      p = plot(size = figuresize, xticks = :all)
      for column in columns[columns .!= "release_year"]
        counts = map(x -> x[column], countmaps)
        plot!(p, years, counts, label = column, xticks = :all)
      end
    
      plot(p) |> display
    end
    
    plotGenreAndTime(transformedData)
    
    ```
    
    {{< figure src="/ox-hugo/2022-07-28_14-08-19_screenshot.png" >}} <br/>


#### Universal Pictures和Paramount Pictures之间的对比 {#universal-pictures和paramount-pictures之间的对比}

<!--list-separator-->

-  电影发行量对比

    ```julia
    function plotCompareTotal(dataframe::DataFrame)
      dataframe[!, "Universal Pictures"] = map(s -> contains(s, "Universal Pictures") ? 1 : 0, dataframe[!, :production_companies])
      dataframe[!, "Paramount Pictures"] = map(s -> contains(s, "Paramount Pictures") ? 1 : 0, dataframe[!, :production_companies])
    
      universalTotal = reduce(+, dataframe[!, "Universal Pictures"])
      paramountTotal = reduce(+, dataframe[!, "Paramount Pictures"])
      total = universalTotal + paramountTotal
    
      xs = ["Universal Pictures", "Paramount Pictures"]
      ys = [universalTotal / total, paramountTotal / total]
      pie(xs, ys, aspect_ratio = 1.0) |> display
    
      companyDifference = groupby(select(dataframe, vcat(xs, "release_year")), :release_year)
      # record: Dict{Year, Dict{Company, Int}}
      record = Dict{Int, Dict{String, Int}}()
      for _dataframe in companyDifference
        year = first(_dataframe.release_year)
        record[year] = Dict{String, Int}()
        for column in xs
          count = reduce(+, _dataframe[!, column])
          record[year][column] = count
        end
      end
    
      _years = collect(keys(record))
      _countmaps = collect(values(record))
      indexs = sortperm(_years)
    
      years = _years[indexs]
      countmaps = _countmaps[indexs]
    
      p = plot(size = figuresize)
      for column in xs
        counts = map(x -> x[column], countmaps)
        plot!(p, years, counts, label = column, xticks = :all)
      end
    
      plot(p) |> display
    end
    
    plotCompareTotal(transformedData)
    ```
    
    {{< figure src="/ox-hugo/2022-07-28_14-11-31_screenshot.png" >}} <br/>
    
    {{< figure src="/ox-hugo/2022-07-28_14-11-46_screenshot.png" >}} <br/>

<!--list-separator-->

-  利润对比

    ```julia
    function plotCompareProfit(dataframe::DataFrame)
      dataframe[!, :profit] = dataframe[!, :revenue] .- dataframe[!, :budget]
      dataframe[!, "Universal Profit"] = dataframe[!, "Universal Pictures"] .* dataframe[!, :profit]
      dataframe[!, "Paramount Profit"] = dataframe[!, "Paramount Pictures"] .* dataframe[!, :profit]
    
      universalProfit = reduce(+, dataframe[!, "Universal Profit"])
      paramountProfit = reduce(+, dataframe[!, "Paramount Profit"])
      totalProfit = universalProfit + paramountProfit
    
      xs = ["Universal Profit", "Paramount Profit"]
      ys = [universalProfit / totalProfit, paramountProfit / totalProfit]
      pie(xs, ys) |> display
    
      companyDifference = groupby(select(dataframe, vcat(xs, "release_year")), :release_year)
      # record: Dict{Year, Dict{Company, Number}}
      record = Dict{Int, Dict{String, Number}}()
      for _dataframe in companyDifference
        year = first(_dataframe.release_year)
        record[year] = Dict{String, Number}()
        for column in xs
          profit = reduce(+, _dataframe[!, column])
          record[year][column] = profit
        end
      end
    
      _years = collect(keys(record))
      _profitmaps = collect(values(record))
      indexs = sortperm(_years)
    
      years = _years[indexs]
      profitmaps = _profitmaps[indexs]
    
      p = plot(size = figuresize)
      for column in xs
        profits = map(x -> x[column], profitmaps)
        plot!(p, years, profits, label = column, xticks = :all)
      end
    
      plot(p) |> display
    end
    
    plotCompareProfit(transformedData)
    ```
    
    {{< figure src="/ox-hugo/2022-07-28_14-13-34_screenshot.png" >}} <br/>
    
    {{< figure src="/ox-hugo/2022-07-28_14-13-49_screenshot.png" >}} <br/>


#### 改编电影和原创电影的对比 {#改编电影和原创电影的对比}

<!--list-separator-->

-  数量对比

    ```julia
    function plotCompareOriginal(dataframe::DataFrame)
      column = "is original"
      dataframe[!, column] = map(x -> contains(x, "based on novel") ? 0 : 1, dataframe[!, :keywords])
    
      keycount = countmap(dataframe[!, column])
      total = keycount[0] + keycount[1]
      xs = ["is original", "not original"]
      ys = [keycount[1] / total, keycount[0] / total]
    
      pie(xs, ys) |> display
    end
    
    plotCompareOriginal(transformedData)
    
    ```
    
    {{< figure src="/ox-hugo/2022-07-28_14-17-41_screenshot.png" >}} <br/>
    
    {{< figure src="/ox-hugo/2022-07-28_14-18-02_screenshot.png" >}} <br/>

<!--list-separator-->

-  平均利润对比

    ```julia
    function plotCompareProfit(dataframe::DataFrame)
      column = "is original"
      (notoriginalDataframe, originalDataframe) = groupby(select(dataframe, [column, "profit"]), column)
      # record: Dict{is original, profit}
      originalProfit = reduce(+, originalDataframe[!, :profit])
      notoriginalProfit = reduce(+, notoriginalDataframe[!, :profit])
      originalCount = first(size(originalDataframe))
      notoriginalCount = first(size(notoriginalDataframe))
      bar(xs, [originalProfit / originalCount, notoriginalProfit / notoriginalCount], size = figuresize) |> display
    end
    
    plotCompareProfit(transformedData)
    ```
    
    {{< figure src="/ox-hugo/2022-07-28_14-19-02_screenshot.png" >}} <br/>