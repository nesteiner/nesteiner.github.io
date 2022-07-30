+++
title = "电信用户流失分析"
date = 2022-07-30T21:43:00+08:00
lastmod = 2022-07-30T21:44:11+08:00
tags = ["数据分析"]
categories = ["Julia"]
draft = false
toc = true
+++

## 电信用户流失分析 {#电信用户流失分析}


### 说明 {#说明}

这次文章的参考来自 <https://zhuanlan.zhihu.com/p/68397317> <br/>


### 提出问题 {#提出问题}

关于用户留存有这样一个观点，如果将用户流失率降低5%，公司利润将提升25%-85% <br/>
如今高居不下的获客成本让电信运营商遭遇“天花板”，甚至陷入获客难的窘境 <br/>
随着市场饱和度上升，电信运营商亟待解决增加用户黏性，延长用户生命周期的问题 <br/>
因此，电信用户流失分析与预测至关重要。 <br/>

数据集来自 [kaggle](https://link.zhihu.com/?target=https%3A//www.kaggle.com/blastchar/telco-customer-churn) <br/>


### 理解数据 {#理解数据}

| 字段名           | 数据类型 | 字段描述    |
|---------------|------|---------|
| customerID       | String  | 顾客ID      |
| gender           | String  | 客户性别    |
| SeniorCitizen    | Integer | 客户是否为老年人 |
| Partner          | String  | 客户是否有合作伙伴 |
| Dependents       | String  | 客户是否有家属 |
| tenure           | Integer | 客户在公司停留的月数 |
| PhoneService     | String  | 客户是否有电话服务 |
| MultipleLines    | String  | 客户是否有多条路线 |
| InternetService  | String  | 客户的互联网服务提供商 |
| OnlineSecurity   | String  | 客户是否具有在线安全性 |
| OnlineBackup     | String  | 客户是否有在线备份 |
| DeviceProtection | String  | 客户是否有设备保护 |
| TechSupport      | String  | 客户是否有技术支持 |
| StreamingTV      | String  | 客户是否有流媒体电视 |
| StreamingMovies  | String  | 客户是否与流媒体电影 |
| Contract         | String  | 客户的合同期限 |
| PaperlessBilling | String  | 客户是否有无纸化账单 |
| PaymentMethod    | String  | 客户的支付方式 |
| MonthlyCharges   | Integer | 每月向客户收取的金额 |
| TotalCharges     | Integer | 向客户收取的总金额 |
| Churn            | String  | 客户是否流失 |


### 数据清洗 {#数据清洗}

首先导入数据 <br/>

```julia
origindata = CSV.read("data/telco-customer-churn/data.csv", DataFrame)
```

查看数据类型，发现 `TotalCharges` 为字符串，把他改为浮点型数据，但是发现他有几个空的数据，是 " " <br/>
我们定义处理函数 <br/>

```julia
function transformTotalCharges(dataframe::DataFrame)
  indexs = dataframe[!, :TotalCharges] .== " "
  dataframe[!, :TotalCharges][indexs] .= string.(dataframe[!, :MonthlyCharges][indexs])

  dataframe[!, :TotalCharges] = map(x -> parse(Float64, x), dataframe[!, :TotalCharges])
  dataframe[!, :tenure][indexs] .= 1
  return dataframe
end
```

解释下最后几行代码， `TotalCharges` 为空的行中，其 `tenure` 入网时长为 0 个月，这样我们推测是当月新入用户 <br/>
根据一般禁烟，用户即使在注册的当月流失，也需缴纳当月费用，因此将这种用户入网市场改为1，将总消费额填充为月消费额 <br/>


### 可视化分析 {#可视化分析}

首先我们查看流失用户和占比 <br/>

```julia
function plotChurn(dataframe::DataFrame)
  counts = countmap(dataframe[!, :Churn])

  yescount = counts["Yes"]
  nocount = counts["No"]
  total = yescount + nocount

  xs = ["Yes", "No"]
  ys1 = [yescount / total, nocount / total]
  ys2 = [yescount, nocount]
  pie(xs, ys1, aspect_ratio = :equal) |> display

  bar(xs, ys2, size = figuresize) |> display 
end

plotChurn(origindata)

```

{{< figure src="/ox-hugo/2022-07-30_21-13-22_screenshot.png" >}} <br/>

{{< figure src="/ox-hugo/2022-07-30_21-13-42_screenshot.png" >}} <br/>

属于不平衡数据集，流失用户占比达 26.54% <br/>


#### 用户属性分析 {#用户属性分析}

```julia
function plotPercentages(dataframe::DataFrame, feature::Symbol, ymatrix::Matrix{Float64})
  columns = [feature, :Churn]
  groupDataframe = groupby(select(dataframe, columns), feature)
  xs = []

  let
    index = 1
    for _dataframe in groupDataframe
      x = first(_dataframe[!, feature])
      push!(xs, x)

      yescount = count(isequal("Yes"), _dataframe[!, :Churn])
      nocount = count(isequal("No"), _dataframe[!, :Churn])
      total = yescount + nocount

      ymatrix[index, :] = [yescount / total, nocount / total]
      index += 1
    end
  end
  groupedbar(ymatrix, xticks = (1:length(xs), xs), label = ["Yes" "No"]) |> display
end
```

```javascript
plotPercentages(origindata, :SeniorCitizen, ones((2, 2)))
```

{{< figure src="/ox-hugo/2022-07-30_21-16-00_screenshot.png" >}} <br/>

```julia
plotPercentages(origindata, :gender, ones((2, 2)))
```

{{< figure src="/ox-hugo/2022-07-30_21-16-23_screenshot.png" >}} <br/>

```julia
plotPercentages(origindata, :Partner, ones((2, 2)))
```

{{< figure src="/ox-hugo/2022-07-30_21-16-39_screenshot.png" >}} <br/>

```julia
plotPercentages(origindata, :Dependents, ones((2, 2)))
```

{{< figure src="/ox-hugo/2022-07-30_21-16-55_screenshot.png" >}} <br/>

```julia
density(origindata.tenure, group = origindata.Churn, size = figuresize) |> display
```

{{< figure src="/ox-hugo/2022-07-30_21-17-38_screenshot.png" >}} <br/>

可以看出 <br/>

-   用户流失与性别基本无关 <br/>
-   年老用户流失占比显著高于年轻用户 <br/>


#### 服务属性分析 {#服务属性分析}

```julia
plotPercentages(origindata, :MultipleLines, ones((3, 2)))
plotPercentages(origindata, :InternetService, ones((3, 2)))
```

{{< figure src="/ox-hugo/2022-07-30_21-20-52_screenshot.png" >}} <br/>

{{< figure src="/ox-hugo/2022-07-30_21-21-11_screenshot.png" >}} <br/>

```julia
function plotPaperlessBillingChurn(dataframe::DataFrame)
  columns = [:PaperlessBilling, :Contract, :Churn]
  groupDataframe = groupby(select(dataframe, columns), :PaperlessBilling)
  array = unique(dataframe[!, :Contract])
  for _dataframe in groupDataframe
    _dataframe = filter(row -> row.Churn == "Yes", _dataframe)
    paperlessbilling = first(_dataframe[!, :PaperlessBilling])

    churn1 = count(isequal(array[1]), _dataframe[!, :Contract])
    churn2 = count(isequal(array[2]), _dataframe[!, :Contract])
    churn3 = count(isequal(array[3]), _dataframe[!, :Contract])

    total = churn1 + churn2 + churn3
    ys = [churn1 / total, churn2 / total, churn3 / total]
    bar(array, ys, title = "PaperlessBilling = $paperlessbilling") |> display
  end

end

plotPaperlessBillingChurn(origindata)

```

{{< figure src="/ox-hugo/2022-07-30_21-21-56_screenshot.png" >}} <br/>

```julia
function plotNumberOfCustomer(dataframe::DataFrame)
  columns = ["PhoneService", "MultipleLines", "OnlineSecurity", "OnlineBackup", "DeviceProtection", "TechSupport", "StreamingTV", "StreamingMovies"]
  ymatrix = ones((length(columns), 3))

  index = 1
  for column in columns
    _dataframe = select(filter(row -> row.InternetService != "No", dataframe), [column, "Churn"])

    count1 = count(isequal("Yes"), _dataframe[!, column])
    count2  = count(isequal("No"),  _dataframe[!, column])
    if column != "MultipleLines"
      ymatrix[index, :] = [count1, count2, 0]
    else
      ymatrix[index, :] = [count1, count2, count(isequal("No phone service"), _dataframe[!, column])]
    end
    index += 1

  end

  groupedbar(ymatrix, xticks = (1:length(columns), columns), label = ["Has Service" "No Service" "No Service"], size = figuresize) |> display
end

plotNumberOfCustomer(origindata)

```

{{< figure src="/ox-hugo/2022-07-30_21-22-28_screenshot.png" >}} <br/>

```julia
function plotNumberOfChurnCustomer(dataframe::DataFrame)
  columns = ["PhoneService", "MultipleLines", "OnlineSecurity", "OnlineBackup", "DeviceProtection", "TechSupport", "StreamingTV", "StreamingMovies"]
  ymatrix = ones((length(columns), 2))

  index = 1
  for column in columns
    _dataframe = select(filter(row -> row.InternetService != "No" && row.Churn == "Yes", dataframe), [column, "Churn"])
    # has service but churn
    yescount = count(isequal("Yes"), _dataframe[!, column])
    # has no service but churn
    nocount = count(isequal("No"), _dataframe[!, column])
    ymatrix[index, :] = [yescount, nocount]
    index += 1
  end

  groupedbar(ymatrix, xticks = (1:length(columns), columns), label = ["Has Service" "No Service"], size = figuresize) |> display
end

plotNumberOfChurnCustomer(origindata)

```

{{< figure src="/ox-hugo/2022-07-30_21-23-00_screenshot.png" >}} <br/>

可以看出 <br/>

-   电话服务整体对用户流失影响较小 <br/>
-   单光纤用户的流失占比较高 <br/>
-   光纤用户绑定了安全，备份，保护，技术支持服务的流失率较低 <br/>
-   光纤用户附加流媒体电视，电影服务的流失率占比较高 <br/>


#### 合同属性分析 {#合同属性分析}

```julia
plotPercentages(origindata, :PaymentMethod, ones((4, 2)))
```

{{< figure src="/ox-hugo/2022-07-30_21-23-49_screenshot.png" >}} <br/>

```julia
density(origindata.MonthlyCharges, group = origindata.Churn, size = figuresize) |> display
```

{{< figure src="/ox-hugo/2022-07-30_21-24-21_screenshot.png" >}} <br/>

```julia
density(origindata.TotalCharges, group = origindata.Churn, size = figuresize) |> display
```

{{< figure src="/ox-hugo/2022-07-30_21-24-50_screenshot.png" >}} <br/>

可以看出 <br/>

-   采用电子支票支付的用户流失率最高，推测该方式的使用体验较为一般 <br/>
-   签订合同方式对客户流失率影响为 按月签订 &gt; 按一年签订 &gt; 按两年签订，证明长期合同最能保留客户 <br/>
-   月消费额大约在 70 - 110 之间用户流失率较高 <br/>
-   长期来看，用户总消费越高，流失率月底，符合一般经验 <br/>