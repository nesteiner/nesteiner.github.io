+++
title = "计算真值表"
date = 2022-07-02T14:30:00+08:00
lastmod = 2022-07-02T14:45:06+08:00
tags = ["真值表"]
categories = ["离散数学"]
draft = false
toc = true
+++

## 程序设计说明 {#程序设计说明}

本来想用 **lisp** 程序来写，他的写法是 <br/>

```scheme
(function arg1 arg2 ...)
```

S表达式不需要考虑优先级顺序，写起来比较简单 <br/>
但是在这里有太多运算符，写成 **lisp** 很不方便 <br/>

```text
p∧￢(q→p
(p ↔ q) → r
p → (p ∨ ￢q ∨ r)
```

还好有 **Julia** ，他可以自定义运算符 <br/>
首先将运算符作为一个函数定义(以下例子这样写是因为 **+** 在程序中是特殊符号，需要加一个 **:** 前缀来标识， **Julia** 语言的一个 **feature**) <br/>

```julia
function Base.:+(arg1::SomeType, arg2::SomeType)
  return arg1 * arg2
end
```

解释器就可以把以下语句 <br/>

```julia
arg1 + arg2
```

解析为 <br/>

```julia
Base:+(arg1, arg2)
```

如果是调用多次 <br/>

```julia
arg1 + arg2 + arg3
```

可以看作先对前两个进行求值，得出结果，与第三个参数求值 <br/>


## 实现细节 {#实现细节}


### 演示说明 {#演示说明}

这里给出一个演示的例子 <br/>

```julia
print(TruthTable("(p ↔ q) → r", [:p, :q, :r], (p, q, r) -> (p ↔ q) → r))
```

需要自定义真值表的 <br/>

1.  标题 <br/>
2.  参数表 <br/>
3.  生成真值的逻辑函数 <br/>


### 运算符定义 {#运算符定义}

```julia
¬(p::Bool) = !p # negative
∧(p::Bool, q::Bool) = p && q #conjunction, use \wedge
∨(p::Bool, q::Bool) = p || q #disjunction, use \vee
→(p::Bool, q::Bool) = begin #implication, use \rightarrow
  !(p == true && q == false)
end

↔(p::Bool, q::Bool) = p == q #equivalence, use \leftrightarrow
```

这些函数我都是通过参考真值表写出来的，一开始看符号推演我都看懵了，他娘的什么玩样，要我死记硬背，还不如自己写一个 <br/>


### 真值表结构 {#真值表结构}

```julia
struct TruthTable
  caption::String
  params::Vector{Symbol}
  target::Function
end
```

我为真值表定义了三个 **field**  <br/>

1.  __caption\_\_ 标题 <br/>
2.  __target\_\_ 计算出真值的函数定义 <br/>
3.  __params\_\_ 为 <span class="underline"><span class="underline">target</span></span> 配套的参数列表，每个都是符号类型 <br/>


### 计算真值并打印 {#计算真值并打印}

**Julia** 语言中可以重载 **print** 函数来打印 **TruthTable** 结构体，通过访问 **TruthTable** 结构体 <br/>
就可以查看对应的真值表 <br/>
这里需要用到第三方库 **DataFrames**  <br/>

```julia
function Base.print(truthTable::TruthTable)
  table = DataFrames.DataFrame()
  len = length(truthTable.params)
  rows = 2 ^ len
  logicTable = reshape(collect(Iterators.product(repeat([[true, false]], len)...)), rows)

  for (symbol, index) in Iterators.zip(truthTable.params, Iterators.countfrom(1, 1))
    table[!, symbol] = map(nums -> nums[index], logicTable)
  end

  results = map(nums -> truthTable.target(nums...), logicTable)
  table[!, truthTable.caption] = results

  println(table)
end
```


## 效果展示 {#效果展示}

{{< figure src="/ox-hugo/shortcut1.png" >}} <br/>