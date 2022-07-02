+++
title = "有监督回归"
date = 2022-07-02T14:33:00+08:00
lastmod = 2022-07-02T14:46:20+08:00
tags = ["有监督回归"]
categories = ["Julia"]
draft = false
toc = true
+++

## 算法介绍 {#算法介绍}

这里简要介绍下各算法和他们之间的关系，详细理解请百度一下 <br/>


### 基本算法 {#基本算法}

最小二乘法是众多机器学习算法中极为重要的一种基础算法 <br/>

单纯的最小二乘法对于包含噪声的学习过程经常有过拟合的弱点，这往往是由于学习模型对于训练样本而言过于复杂 <br/>


### l2 约束 {#l2-约束}

由此，引入带有约束条件的最小二乘法 ---- Ridge 回归 <br/>

带有约束条件的最小二乘法和交叉验证法的组合，在实际应用中是非常有效的回归方法 <br/>
然而，当参数特别多的时候，求解各参数以及学习得到的函数的输出值的过程，都需要耗费大量的时间 <br/>


### l1 约束 {#l1-约束}

由此，引入可以吧大部分参数都置为0的稀疏学习算法 <br/>
因为大部分参数都变成了0，所以就可以快速地求解各参数以及学习得到的函数的输出值 <br/>


### l1 + l2 约束 {#l1-plus-l2-约束}

虽然 l1 约束的最小二乘学习法是非常有用的学习方法，但是在实际应用中，经常会遇到些许限制 <br/>

-   在 Lasso 回归求解路径中，对于 N×P 的设计矩阵来说，最多只能选出 min(N,p) 个变量 <br/>
    当 p&gt;N 的时候，最多只能选出N个预测变量．因此，对于 p∼N 的情况，Lasso方法不能够很好的选出真实的模型． <br/>
-   如果预测变量具有群组效应，则用Lasso回 归时，只能选出其中的一个预测变量 <br/>
-   对于通常的 N&gt;P 的情形，如果预测变量中 存在很强的共线性，Lasso的预测表现受控于岭回归 <br/>

基于以上几点Lasso回归的局限性，Zou和 Hastie在2005年提出了弹性网回归方法，回归系 数表达式为 <br/>

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">

\\(\hat \beta^{ridge} =\mathop{\arg\min}\_{\beta}  \\{\sum \limits \_{i=1}^{N}(y\_i-\beta\_0-\sum\limits\_{j=1}^px\_{ij}\beta\_j)^2+\lambda\sum \limits\_{j=1}^{p}|\beta\_{j}|+\lambda\sum \limits\_{j=1}^{p}\beta\_{j}^2\\}\\) <br/>

</div>


## MLJLinearModels 使用 {#mljlinearmodels-使用}


### Ridge {#ridge}

\\(J = \frac{1}{n}\sum\_{i = 1}^n (f( x\_i) - y\_i)^2 + \lambda \\|w\\|\_2^2\tag{1}\\) <br/>

**RidgeRegressor** <br/>

```julia
RidgeRegression()
RidgeRegression(λ; lambda, fit_intercept, penalize_intercept, scale_penalty_with_samples)
```


### Lasso {#lasso}

\\(J = \frac{1}{n}\sum\_{i = 1}^n (f( x\_i) - y\_i)^2 + \lambda \\|w\\|\_1\tag{2}\\) <br/>

**LassoRegressor** <br/>

```julia
LassoRegression()
LassoRegression(λ; lambda, fit_intercept, penalize_intercept, scale_penalty_with_samples)
```


### Elastic-Net {#elastic-net}

\\(\smash{\min\_{w}}\sum\_{i=1}^m(y\_i-\sum\_{j=1}^dx\_{ij}w\_j)^2 + \lambda\sum\_{j=1}^d|w\_j|+\lambda \sum\_{j=1}^dw\_j^2 \tag{3}\\) <br/>
**ElasticNetRegression** <br/>

```julia
ElasticNetRegression()
ElasticNetRegression(λ)
ElasticNetRegression(λ, γ; lambda, gamma, fit_intercept, penalize_intercept, scale_penalty_with_samples)
```


### 说明 {#说明}

其实可以不用管 <br/>

-   `fit_intercept` <br/>
-   `penalize_intercept` <br/>

我也不知道这两个是干什么的，就先别管他们了 <br/>
总之，只用设置 `lambda` 就行了 <br/>


## 实例 波士顿房价预测 {#实例-波士顿房价预测}


### 数据准备 {#数据准备}

竞赛数据来自 <https://www.kaggle.com/competitions/house-prices-advanced-regression-techniques/overview> <br/>

```julia
using MLJ, CSV, StableRNGs, MLJLinearModels, Plots
import DataFrames: DataFrame, select, describe
using Statistics


dataTrain = CSV.read("data/train.csv", DataFrame)
dataTest = CSV.read("data/test.csv", DataFrame)
```


### 观察各项主要特征与房价售价的关系 {#观察各项主要特征与房价售价的关系}


#### [存疑]分析 SalePrice {#存疑-分析-saleprice}

```julia
julia> describe(dataTrain[!, :SalePrice])
Summary Stats:
Length:         1460
Missing Count:  0
Mean:           180921.195890
Minimum:        34900.000000
1st Quartile:   129975.000000
Median:         163000.000000
3rd Quartile:   214000.000000
Maximum:        755000.000000
Type:           Int64
```

通过上面的结果可以知道 **SalePrice** 没有无效或者其他非数值的数据，下面通过图示化来进一步展示 **SalePrice** <br/>

{{< figure src="/ox-hugo/2022-05-05_19-56-37_screenshot.png" >}} <br/>

这里需要一个 `distplot` 函数来绘制图像 <br/>

1.  得到数组的 **distribution** <br/>
2.  画出这个分布 <br/>

然而我还不会这个东西，放一放 <br/>


#### 分析特征数据 {#分析特征数据}

入选特征 <br/>

| 变量名       | 数据类型   | 注释   |
|-----------|--------|------|
| LotArea      | Continuous | 地皮面积 |
| GrLiveArea   | Continuous | 生活面积 |
| TotalBsmtSF  | Continuous | 地下室总面积 |
| MiscVal      | Continuous | 其他资产 |
| GarageCars   | Count      | 容纳车辆 |
| GarageArea   | Continuous | 车库面积 |
| YearBuilt    | Multiclass | 建造年份 |
| CentralAir   | Multiclass | 中央空调 |
| OverallQual  | Multiclass | 总体评价 |
| Neighborhood | Multiclass | 地段   |


#### 验证主要特征是否满足要求 {#验证主要特征是否满足要求}

<!--list-separator-->

-  类别型特征

    <!--list-separator-->
    
    -  CentralAir 中央空调
    
        ```julia
        using StatsPlots
        let column = :CentralAir
            columnY = dataTrain[!, :SalePrice]
            columnX = dataTrain[!, column]
            boxplot(columnX, columnY) |> display
        end
        ```
        
           ![](/ox-hugo/2022-05-03_22-25-14_screenshot.png) <br/>
        可以很明显的看到有中央空调的房价明显更高。 <br/>
    
    <!--list-separator-->
    
    -  OverallQual 总体评价
    
        ```julia
        let column = :OverallQual
            columnY = dataTrain[!, :SalePrice]
            columnX = dataTrain[!, column]
            boxplot(columnX, columnY) |> display
        end
        ```
        
        {{< figure src="/ox-hugo/2022-05-03_22-27-36_screenshot.png" >}} <br/>
    
    <!--list-separator-->
    
    -  YearBuilt 建造年份
    
        ```julia
        let column = :YearBuilt
            columnY = dataTrain[!, :SalePrice]
            columnX = dataTrain[!, column]
            boxplot(columnX, columnY, size=(2600, 1200)) |> display
        end
        
        ```
        
        {{< figure src="/ox-hugo/2022-05-03_22-32-02_screenshot.png" >}} <br/>
        
        ```julia
        let column = :YearBuilt
            columnY = dataTrain[!, :SalePrice]
            columnX = dataTrain[!, column]
            boxplot(columnX, columnY, size=(2600, 1200)) |> display
            scatter(columnX, columnY, ylim=(0, 800000), size=(1500, 1000)) |> display
        end
        
        ```
        
        {{< figure src="/ox-hugo/2022-05-03_22-34-34_screenshot.png" >}} <br/>
        
        最开始我是用了箱线图绘制了房价与建造年份的关系，但是并不十分明显，所以又用点图来显示，可以很明显的看到有线性增长的趋势。 <br/>
    
    <!--list-separator-->
    
    -  Neighborhood 地段
    
        ```julia
        let column = :Neighborhood
            columnY = dataTrain[!, :SalePrice]
            columnX = dataTrain[!, column]
            boxplot(columnX, columnY, size = (1300, 600)) |> display
        end
        
        ```
        
        {{< figure src="/ox-hugo/2022-05-03_22-36-46_screenshot.png" >}} <br/>
        
        这个该怎么分析呢。。。。。。待定 <br/>

<!--list-separator-->

-  数值型特征

    <!--list-separator-->
    
    -  LotArea 地表面积
    
        ```julia
        let column = :LotArea
            columnY = dataTrain[!, :SalePrice]
            columnX = dataTrain[!, column]
            scatter(columnX, columnY) |> display
        end
        ```
        
        {{< figure src="/ox-hugo/2022-05-03_22-39-31_screenshot.png" >}} <br/>
        
        好像该特征并没有什么差别，所以不予考虑 <br/>
    
    <!--list-separator-->
    
    -  GrLivArea 生活面积
    
        ```julia
        let column = :GrLivArea
            columnY = dataTrain[!, :SalePrice]
            columnX = dataTrain[!, column]
            scatter(columnX, columnY) |> display
        end
        ```
        
        {{< figure src="/ox-hugo/2022-05-03_22-41-17_screenshot.png" >}} <br/>
    
    <!--list-separator-->
    
    -  TotalBsmtSF 地下室总面积
    
        ```julia
        let column = :TotalBsmtSF
            columnY = dataTrain[!, :SalePrice]
            columnX = dataTrain[!, column]
            scatter(columnX, columnY) |> display
        end
        ```
        
        {{< figure src="/ox-hugo/2022-05-03_22-43-16_screenshot.png" >}} <br/>
    
    <!--list-separator-->
    
    -  MiscVal
    
        ```julia
        let column = :MiscVal
            columnY = dataTrain[!, :SalePrice]
            columnX = dataTrain[!, column]
            scatter(columnX, columnY) |> display
        end
        ```
        
        {{< figure src="/ox-hugo/2022-05-03_22-44-30_screenshot.png" >}} <br/>
    
    <!--list-separator-->
    
    -  GarageArea/GarageCars 车库
    
        ```julia
        let columns = [:GarageArea, :GarageCars]
            columnY = dataTrain[!, :SalePrice]
            columnXs = map(column -> dataTrain[!, column], columns)
        
            for columnX in columnXs
        	scatter(columnX, columnY) |> display
            end
        end
        ```
        
        {{< figure src="/ox-hugo/2022-05-03_22-53-30_screenshot.png" >}} <br/>
        
        ![](/ox-hugo/2022-05-03_22-54-01_screenshot.png) <br/>
        由上面点图可以看出房价与车库面积和容纳车辆数呈现线性关系，所以入选主要特征 <br/>


#### 主要特征 {#主要特征}

总结起来，最后 <br/>

| 变量名       | 数据类型   | 注释   |
|-----------|--------|------|
| GrLiveArea   | Continuous | 生活面积 |
| TotalBsmtSF  | Continuous | 地下室总面积 |
| GarageCars   | Count      | 容纳车辆 |
| GarageArea   | Continuous | 车库面积 |
| YearBuilt    | Multiclass | 建造年份 |
| CentralAir   | Multiclass | 中央空调 |
| OverallQual  | Multiclass | 总体评价 |
| Neighborhood | Multiclass | 地段   |


### 更加科学的分析数据 {#更加科学的分析数据}

上面的分析可以说非常主观，所以说多多少少还是会不放心，会担心自己选择的特征会不会多了或者少了， <br/>
又或者选了一些没有太大作用的特征，所以接下来需要进行更加科学的分析 <br/>
为了做到更加科学，我们需要作如下工作： <br/>

-   得到各个特征之间的关系矩阵 -- correlation matrix <br/>
-   SalePrice 的关系矩阵 <br/>
-   绘制出最相关的特征之间的关系图 <br/>


#### 关系矩阵 {#关系矩阵}

教程中有局限性， **关系矩阵只涉及到数值型数据** ，这里我们也这样做，因为他的特征数有80多个，我懒得弄 <br/>

```julia
let _schema = schema(dataTrain)
    _names = _schema.names
    _scitypes = _schema.scitypes
    indexs = collect(map(x -> x == Count || x == Continuous, _scitypes))
    columns = _names[indexs] |> collect
    _data = select(dataTrain, columns)
    _corr = cor(Matrix(_data))
    labels = string.(columns)
    heatmap(labels, labels, _corr, xrotation = -90, size = figureSize, xticks = :all, yticks = :all) |> display
end
```

![](/ox-hugo/2022-05-04_21-11-07_screenshot.png) <br/>
像素块越亮表示两者之间相关性越强，我们可以很清楚地看到与“SalePrice”相关性很强的有 <br/>

-   `OverallQual` 总评价 <br/>
-   `YearBuilt` 建造年份 <br/>
-   `ToatlBsmtSF` 地下室面积 <br/>
-   `1stFlrSF` 一楼面积 <br/>
-   `GrLiveArea` 生活区面积 <br/>
-   `FullBath` 浴室？what。。。到底什么意思，知道的麻烦说一下 <br/>
-   `TotRmsAbvGrd` 总房间数（不包括浴室） <br/>
-   `GarageCars` 车库可容纳车辆数 <br/>
-   `GarageArea` 车库面积 <br/>


#### [存疑]房价关系矩阵 {#存疑-房价关系矩阵}

这里显示相关性最大的10个特征 <br/>

```python
k  = 10 # 关系矩阵中将显示10个特征
cols = corrmat.nlargest(k, 'SalePrice')['SalePrice'].index
cm = np.corrcoef(data_train[cols].values.T)
sns.set(font_scale=1.25)
hm = sns.heatmap(cm, cbar=True, annot=True, \
		 square=True, fmt='.2f', annot_kws={'size': 10}, yticklabels=cols.values, xticklabels=cols.values)
plt.show()
```

我不知道这个代码是怎么运行的，他是怎么画出这个热力图的 <br/>

{{< figure src="/ox-hugo/2022-05-05_18-23-15_screenshot.png" >}} <br/>

**重点是 `corrmat.nlargestk` 是怎么得出 10x10 的矩阵** <br/>

我只做到这里 <br/>

```julia
let _schema = schema(dataTrain)
    _names = _schema.names
    _scitypes = _schema.scitypes
    indexs = collect(map(x -> x == Count || x == Continuous, _scitypes))
    columns = _names[indexs] |> collect
    labels = string.(columns)
    _data = select(dataTrain, columns)
    _corr = cor(Matrix(_data))

    _dataframe = DataFrame(_corr, columns)
    nlarget = _dataframe[partialsortperm(_dataframe[!, :SalePrice], 1:10, rev=true), :]

    heatmap(Matrix(nlarget), xrotation = -90, size = figureSize, xticks = :all, yticks = :all, aspect_ratio = :equal)

    nrow, ncol = size(_corr)
    fontsize = 15

    fn(tuple) = (tuple[1], tuple[2], text(round(_corr[tuple[1], tuple[2]], digits = 2), fontsize, :white, :center))
    ann = map(fn, Iterators.product(1:nrow, 1:ncol) |> collect |> vec)

    annotate!(ann, linecolor = :white) |> display
end
```

{{< figure src="/ox-hugo/2022-05-05_18-58-23_screenshot.png" >}} <br/>

疑点如下 <br/>

1.  如何获取 `Dataframe` 最大的 10x10 切片 <br/>
2.  `Dataframe` 的字段名也要根据数据排序进行修改吧？ <br/>


#### [存疑]绘制关系点图 {#存疑-绘制关系点图}

目前找到一个 `PairPlots` 包，我还要研究一下 <br/>


### 开始模拟数据 {#开始模拟数据}


#### 处理数据 {#处理数据}

1.  首先我们选取特征 <br/>
    ```julia
    columns = [:OverallQual, :GrLivArea, :GarageCars, :TotalBsmtSF, :FullBath, :TotRmsAbvGrd, :YearBuilt]
    ```

2.  定义训练集的处理模型 <br/>
    ```julia
    trainTransformModel = Pipeline(
        FeatureSelector(features = columns),
        dataframe -> coerce(dataframe, Count => Continuous))
    ```

3.  定义测试集的处理模型 <br/>
    ```julia
    processFeature!(dataframe::DataFrame) = begin
        dataframe[!, :GarageCars] = replace(dataframe[!, :GarageCars], "NA" => missing)
        dataframe[!, :GarageCars] = map(x -> ismissing(x) ? x : parse(Float64, x), dataframe[!, :GarageCars])
        dataframe[!, :TotalBsmtSF] = replace(dataframe[!, :TotalBsmtSF], "NA" => missing)
        dataframe[!, :TotalBsmtSF] = map(x -> ismissing(x) ? x : parse(Float64, x), dataframe[!, :TotalBsmtSF])
    
        coerce!(dataframe, Count => Continuous)
        return dataframe
    end
    
    testTransformModel = Pipeline(
        FeatureSelector(features = columns),
        processFeature!,
        FillImputer(features = columns),
        # Standardizer(features = columns)
    )
    
    ```

4.  处理原始数据，产出数据集 <br/>
    ```julia
    trainTransformMach = machine(trainTransformModel, dataTrain)
    testTransformMach = machine(testTransformModel, dataTest)
    fit!(trainTransformMach)
    fit!(testTransformMach)
    
    transformedDataTrain = transform(trainTransformMach, dataTrain)
    transformedDataTest = transform(testTransformMach, dataTest)
    ```

5.  拿出训练用数据 <br/>
    ```julia
    X = transformedDataTrain
    y = coerce(dataTrain[!, :SalePrice], Continuous)
    train, test = partition(eachindex(y), 0.8, rng=rng)
    ```


#### 模型训练 {#模型训练}

这里我们使用 **Ridge** 模型来检验 <br/>

```julia
rng = StableRNG(1234)
cv = CV(nfolds = 6, rng = rng)
tuning = Grid(resolution=10, rng = rng)

# MODULE try Ridge
ridge = RidgeRegressor()
rangeLambda = range(ridge, :lambda, lower = 0.1, upper = 10.0, scale=:log)


tunedModel = TunedModel(model = ridge,
			range = [rangeLambda],
			measure = rms,
			resampling = cv,
			tuning = tuning)
tunedMach = machine(tunedModel, X, y)
fit!(tunedMach, rows = train)

evaluate!(tunedMach, resampling = cv, measure = [rms, l1], rows = test)
```

{{< figure src="/ox-hugo/2022-05-05_19-24-51_screenshot.png" >}} <br/>


#### 补充: lightGBM 模型训练 {#补充-lightgbm-模型训练}

```julia
LGBMRegressor = @load LGBMRegressor
lgb = LGBMRegressor()
lgbm = machine(lgb, X, y)
boostRange = range(lgb, :num_iterations, lower = 2, upper = 500)
rangeLeaf = range(lgb, :min_data_in_leaf, lower = 1, upper = 50)
rangeIteration = range(lgb, :num_iterations, lower = 50, upper = 100)
rangeMinData = range(lgb, :min_data_in_leaf, lower = 2, upper = 10)
rangeLearningRate = range(lgb, :learning_rate, lower = 0.1, upper = 1)

tunedModel = TunedModel(model = lgb,
			tuning = Grid(resolution = 5, rng = rng),
			resampling = cv,
			ranges = [rangeIteration, rangeMinData, rangeLearningRate],
			measure = rms)

tunedMachine = machine(tunedModel, X, y)
fit!(tunedMachine, rows = train)
evaluate!(tunedMach, resampling = cv, measure = [rms, l1], rows = test)
```

{{< figure src="/ox-hugo/2022-05-05_19-29-43_screenshot.png" >}} <br/>


### 检验测试集数据 {#检验测试集数据}

这里我们用 **lightGBM** 产出的数据来提交，不得不说，这个模型老牛逼了 <br/>

```julia
predictions = predict(tunedMachine, transformedDataTest)
output = DataFrame(Id=dataTest.Id)
output[!, :SalePrice] = predictions
CSV.write("data/submission.csv", output)
```

![](/ox-hugo/2022-05-05_19-31-25_screenshot.png) <br/>
哟系