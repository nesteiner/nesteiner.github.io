+++
title = "Julia 简单爬虫编写"
date = 2022-07-23T17:58:00+08:00
lastmod = 2022-07-23T18:04:18+08:00
tags = ["爬虫"]
categories = ["Julia"]
draft = false
toc = true
+++

## 项目介绍 {#项目介绍}

我想试试 do everything in Julia，所以这里我简单写了一个爬虫，来爬取涩图 <br/>
<https://github.com/nesteiner/WebCrawl.jl> <br/>


### 安装包 {#安装包}

以下的包先安装好 <br/>

-   HTTP 用于获取 HTTP <br/>
-   Gumbo 用于解析 HTTP <br/>
-   Cascadia 使用 CSS Selector 解析 HTTP <br/>
-   URIs 用于uri 的解析 <br/>

我们要爬取的 url 在这里 [未成年人请在父母陪同下点击](https://www.meitu131.com/meinv/5287/index.html) <br/>


### 工具函数 {#工具函数}

Julia 原有的 `joinpath` 不能满足我们的需求，比如我们要合并这两个 url <br/>

-   <https://example.com/hello> <br/>
-   /hello/1 <br/>

合并后的结果应该是 <https://example.com/hello/1> ，可 `joinpath` 合并后的结果是 /hello/1 <br/>
这里我们使用 URIs 中的 `resolvereference` 来简单写一个 `urljoin` <br/>

```julia
function urljoin(base::AbstractString, ref::AbstractString)
  return string(
    resolvereference(base, ref)
  )
end
```


## 整体流程 {#整体流程}

首先我们定义一个全局的 `CONFIG` 对象，来设置一些参数，比如 <br/>

-   代理地址 <br/>
-   请求头 <br/>
-   Cookie <br/>
-   User-Agent <br/>

我们再定义解析函数 `parse` ，由于处理方式与 `scrapy` 不太一样，没有必要将接口设计成一样 <br/>

```julia
parse(startpage::String, dict::Dict{String, T}) where T <: Any
```

由于出现分页，我们需要开始递归解析页面，我们从 `startpage` 开始解析， `dict` 存储一些额外参数，如 <br/>

-   存储的文件夹位置 <br/>
-   图片的名称 <br/>

调用解析函数时，我们还会处理图片下载请求 <br/>

```julia
pipeline(image::String, path::String)
```

他将从 `image` 下载资源，存储到 `path` 中 <br/>


## 详细代码 {#详细代码}

参考 `test/runtests.jl` ，我们这样调用接口 <br/>

```julia
@testset "test fetch meitulu" begin
  dict = Dict("startfrom" => 1,
	      "startpath" => "/home/steiner/Downloads/evelyn/[MyGirl美媛馆] 性感嫩模Evelyn艾莉 - 女仆厨娘装制服诱惑系列写真 Vol.157")
  startpage = "https://www.meitu131.com/meinv/5287/index.html"
  try
    WebCrawl.parse(startpage, dict)
  catch error
    print(stderr, error)
  end

end
```

接下来在包模式下，写下 `test` 即可，成品就不展示了，怕过不了审 <br/>


## 疑问 {#疑问}

在下载中，我们写下了如下代码 <br/>

```julia
@sync for image in images
  directory = dict["startpath"]
  if !isdir(directory)
    mkdir(directory)
  end

  path = joinpath(directory, string(dict["startfrom"]) * ".jpg")
  dict["startfrom"] += 1

  url = image.attributes["src"]
  # @async pipeline(url, path)
  @async pipeline(url, path)
end
```

他的意思是将 `for` 中的 `@async` 操作全部同步处理，实际操作下来发现跟去除 `@sync`, `@async` <br/>
没有多少区别，于是我又设计了这样的代码 <br/>

```julia
tasks = Task[]

for image in images
  task = () -> begin
    directory = dict["startpath"]
    if !isdir(directory)
      mkdir(directory)
    end

    path = joinpath(directory, string(dict["startfrom"]) * ".jpg")
    dict["startfrom"] += 1

    url = image.attributes["src"]
    # @async pipeline(url, path)
    pipeline(url, path)
  end

  push!(tasks, Task(task))
end

for task in tasks
  schedule(task)
end

for task in tasks
  wait(task)
end

```

其实效率还是一样的，我不知道如何通过异步加速下载过程，如果你了解的话，请务必告诉我