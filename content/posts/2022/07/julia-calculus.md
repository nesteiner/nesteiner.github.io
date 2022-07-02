+++
title = "Julia 微积分简要介绍"
date = 2022-07-02T14:32:00+08:00
lastmod = 2022-07-02T14:45:51+08:00
tags = ["微积分"]
categories = ["Julia"]
draft = false
toc = true
+++

## 介绍 {#介绍}

**JuliaMath** 下有一个微积分包，闲得没事看看文档，给大家简要整理了一下 <br/>

-   `derivative` 求导 <br/>
-   `second_derivative` 求二阶导 <br/>

然而关于积分，文档里说 <br/>

> The Calculus package no longer provides routines for univariate numerical integration. Use QuadGK.jl instead. <br/>

他说单变量的积分在这个包中不再支持，可以去使用 `QuadGK.jl` 包中 <br/>
我又尝试了下 `Calculus` 包中的 `integrate` 函数 <br/>

```julia
f((x, y)) = sin(x) + cos(y)
```

{{< figure src="/ox-hugo/shortcut.png" >}} <br/>

我发现，这个错误中的 `quadgk` 和 `QuadGk` 中的 `quadgk` 不是同一个函数，他指的是 `Calculus.quadgk` <br/>
这个函数不存在 <br/>

另外我去谷歌了一下，发现 `Quadgk` 不支持多重积分，所以我推荐使用同在 `JuliaMath` 仓库下的 `Cubature.jl` 包 <br/>
既可以做单变量积分，又可以做多变量积分 <br/>


## 使用 {#使用}


### 导数 using Calculus {#导数-using-calculus}

**derivative** <br/>

```julia
derivative(sin, 0) == cos(0)
derivative(sin, 1) == cos(1)
derivative(sin, float(pi)) = cos(float(pi))
```


### 多元导数 using Calculus {#多元导数-using-calculus}

**derivative** <br/>

```julia
julia> derivative(f, [1, 1])
2-element Vector{Float64}:
  0.5403023058631036
 -0.8414709847974693
```


### 积分 using Cubature {#积分-using-cubature}

**hquadrature** <br/>

```julia
julia> (val, err) = hquadrature(x -> x^3, 0, 1)
(0.25, 2.7755575615628914e-15)

```

```julia
(val, err) = hquadrature(f::Function, xmin, xmax;
			 reltol=1e-8, abstol=0, maxevals=0)
```

其中 <br/>

-   `f` 是被积函数 <br/>
-   `xmin` 和 `xmax` 是积分区间 <br/>
-   `reltol` 表示 _required relative error tolerance_ <br/>
-   `abstol` 表示 _required absolute error tolerance_ <br/>
-   `maxevals` 表示 _指定函数评估的(粗略)最大数量_ <br/>


### 多元积分 using Cubature {#多元积分-using-cubature}

**hcubature** <br/>

```julia
(val,err) = hcubature(f::Function, xmin, xmax;
		      reltol=1e-8, abstol=0, maxevals=0)
```

只有函数名不同，参数类型不同，其他解释同上 <br/>

```julia
f((x, y)) = x^3 * y^2
hcubature(f, [0,0],[1,1])
```

```julia
julia> hcubature(f, [0,0],[1,1])
(0.08333333333333331, 2.7755575615628914e-17)
```


## 补充说明 {#补充说明}

这份介绍主要是简单介绍，详细内容可以去查看这两个包的文档 <br/>

-   Calculus <br/>
    <https://github.com/JuliaMath/Calculus.jl> <br/>
-   Cubature <br/>
    <https://github.com/JuliaMath/Cubature.jl>