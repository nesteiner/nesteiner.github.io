+++
title = "Symbolics.jl 简单使用"
date = 2022-07-02T14:31:00+08:00
lastmod = 2022-07-02T14:45:20+08:00
tags = ["Symbolics"]
categories = ["Julia"]
draft = false
toc = true
+++

## Symbolics.jl 简单使用 {#symbolics-dot-jl-简单使用}

由于文档太难看懂，我这里简单地介绍下这个包的用法 <br/>
![](/ox-hugo/Symbolics.png) <br/>


### 变量符号 {#变量符号}

生成变量符号，只需要 <br/>

```julia
@variables x y
```

这样，我们就有两个符号为 `x` , `y` 的变量 <br/>


### 变量表达式 {#变量表达式}


#### 用表达式生成符号变量 {#用表达式生成符号变量}

如同写一个算数表达式 `1+1` 一样，Julia 为符号变量类型重载了运算符，这里创建一个变量 `p`, <br/>
用其来代表 <br/>

```julia
p = -16x^2 + 100
```

注意， `p` 也是一个符号变量 <br/>


#### 用函数生成符号变量 {#用函数生成符号变量}

或者我们还可以这样声明 `p` <br/>

```julia
f(x) = -16x^2 + 100
p = f(x)
```


#### 替换表达式 {#替换表达式}

写好了表达式还不够，有时候我们将其中一个函数看作另一个函数的参数，比如 `f(g(x))` <br/>

```julia
substitude(p, x => x^2)
```

得到结果为 `-16x^4 + 100` <br/>
如果是多元函数，替换的部分用字典来替代 <br/>


#### 简化/展开表达式 {#简化-展开表达式}

简化表达式，使用 `simplify` 函数，但是这个包里没有提供单独的 **展开** 函数，还好 `simplify` <br/>
有一个 `expand` 参数，为 `true` 时展开表达式，来看一个例子 <br/>

```julia
  @variables a b c x
  quad = a*x^2 + b*x + c
  quad + quad^2 - quad^3

  simpify(quad, expand=true)
# julia> simplify(ans,expand=true)
# c + b*x + c^2 + (a + b^2)*(x^2) + (a^2)*(x^4) + 2a*c*(x^2) + 2b*c*x + 2a*b*(x^3) - (c^3) - (a^3)*(x^6) - (b^3)*(x^3) - 3a*(c^2)*(x^2) - 3a*(b^2)*(x^4) - 3b*x*(c^2) - 3c*(b^2)*(x^2) - 3c*(a^2)*(x^4) - 3b*(a^2)*(x^5) - 6a*b*c*(x^3)
```


#### 求导表达式 {#求导表达式}

与求导相关的函数是 `Differential` ，中文意思 **微分** <br/>

```julia
@variables t
D = Differential(t)
```

声明一个变量符号 `t`, 再声明一个变量 `D` 代表有关 `t` 的微分 <br/>
接着，声明有关 `t` 的表达式，调用 `D(z)` 得到导数表达式 <br/>

```julia
z = t + t^2
D(z)
```

然而求导部分是惰性的，怕消耗太多时间，需要调用 `expand_derivatives` 来获取表达式 <br/>

```julia
expand_derivatives(D(z)) # 1 + 2t
```


### 生成 Julia 函数 {#生成-julia-函数}

表达式可以看作函数的简化版本，比如 <br/>

```julia
p = -16x^2 + 100
```

我们直接调用 `p(1)` 是会报错的，我们需要根据他来生成一个函数调用 <br/>

```julia
f_expr = build_function(p, x)
f_func = eval(f_expr)
f_func(1) # 84
```

1.  `build_function` 需要一个表达式，和其参数变量表 <br/>
2.  如果是 `f(x, y)` ，应该输入 `build_function(f, x, y)` <br/>
3.  如果是 `f([x, y])` , 应该输入 `build_function(f, [x, y])` <br/>
4.  `build_function` 返回的是 Julia 函数的 `Expr` 类型, `eval` 他即可获得函数 <br/>


## 疑问 {#疑问}


### 怎么没有积分啊 {#怎么没有积分啊}

这个包里确实没有写积分的函数，不过我已经提了个 **Issue** 给作者 <br/>
<https://github.com/JuliaSymbolics/Symbolics.jl/issues/532> <br/>

![](/ox-hugo/2022-02-19_16-52-00_screenshot.png) <br/>
我都已经给他跪了，不信他不理我 <br/>