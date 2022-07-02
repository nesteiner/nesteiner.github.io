+++
title = "Scrapy 爬虫的简单使用"
date = 2022-07-02T14:31:00+08:00
lastmod = 2022-07-02T14:45:36+08:00
tags = ["Scrapy", "爬虫", "Scrapy"]
categories = ["Python"]
draft = false
toc = true
+++

## 简单的爬虫流程 {#简单的爬虫流程}

一般的，程序向网站请求网页 <br/>

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">

程序 =========&gt; 网站 <br/>

</div>

而后，网站返回网页与程序 <br/>

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">

程序 &lt;========= 网站 <br/>

</div>

接着程序负责解析得到的 **HTML** 数据即可 <br/>


## Scrapy 如何扩展这一流程 {#scrapy-如何扩展这一流程}

程序请求网页时， `scrapy` 添加了中间件在双方之间 <br/>

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">

程序 =====&gt; 中间件 ======&gt; 网站 <br/>

</div>

网站返回数据时，也需要通过中间件 <br/>

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">

程序 &lt;===== 中间件 &lt;====== 网站 <br/>

</div>

程序解析完数据后，不会马上处理数据，而是把一个网页的数据打包成一个 `Item` 数据，传递给 `pipeline` 处理 <br/>

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">

pipeline &lt;==== item &lt;==== 程序 <br/>

</div>


## Scrapy 简单示例 {#scrapy-简单示例}

这里以爬取美图录的图片为例(注意，我不打算用默认的 `start_urls` 数据) <br/>
首先我们到一个模特的主页上，比如 <br/>

![](images/Scrapy_简单示例/2022-04-10_20-13-21_screenshot.png) <br/>
接下来我们要从这个 **指定模特页面** 上的每一个专辑上爬取图片 <br/>


### 准备项目 {#准备项目}

1.  安装 `scrapy` <br/>
    ```bash
    sudo pip3 install scrapy
    ```

2.  创建新项目 <br/>
    ```bash
    scrapy startproject meitulu
    ```


### 基础设定 {#基础设定}

在 `meitulu/spiders/` 下我们新建一个爬虫文件 `evelyn.py` <br/>

```python
class Spider(scrapy.Spider):
    name = 'evelyn'

```

其中的 `name` 属性是为了调用的时候能找到爬虫位置，比如调用这个爬虫的时候，我们指定 `name` <br/>

```bash
scrapy crawl evelyn
```


### 添加命令行参数 {#添加命令行参数}

```python
def __init__(self, albumUrl=None, *args, **kwargs):
    if albumUrl == None:
	raise Exception('usage: -a albumUrl=...')

    super(Spider, self).__init__(*args, **kwargs)
    self.start_url = albumUrl

def start_requests(self):
    yield scrapy.Request(url=self.start_url, callback=self.parse)

```

这里在类的 `__init__` 方法上我们做了修改，第二个参数是命令行参数名称，默认为 `None` ，第三，第四个参数不需要管 <br/>
我们这样调用 <br/>

```bash
scrapy crawl evelyn -a albumUrl='http://meitulu.cn/t/Evelynaili/'
```

注意这里我们没有定义 `start_urls` 而是 `start_url` ，这里我覆盖了这种默认行为，重新定义了 `start_requests`  <br/>
并将获取到的响应交给 `self.parse` 回调来解析 <br/>


### 解析函数 parseAlbum {#解析函数-parsealbum}

这里的函数解析一张专辑中的每一张图片，然后解析出 `item` ，传递给 `pipeline` ，如果有下一页，继续请求 <br/>

```python
def parseAlbum(self, response, firstUrl, albumName, count):
    url = response.css('div.content center a img.content_img::attr(src)').extract_first()
    name = str(count) + '.jpg'

    yield Image(url=url, name=name, albumName=albumName, fromUrl=response.url)

    nextPage = response.urljoin(response.css('div.content center a::attr(href)').extract_first())
    if nextPage != firstUrl:
	yield scrapy.Request(
	    url=nextPage,
	    callback=self.parseAlbum,
	    cb_kwargs=dict(firstUrl=nextPage, albumName=albumName, count=count+1))
```

在函数中， <br/>

-   `firstUrl` <br/>
    表示专辑的第一页网址，也是专辑的入口地址，设置他是因为最后一页的下一页是 `firstUrl` ， <br/>
    需要用来判断当前页是否为最后一页 <br/>
-   `albumName` <br/>
    表示专辑名称，设置他是因为我看了下html代码，每一张图片的 `alt` 属性都是专辑名，为其命名也麻烦 <br/>
-   `count` <br/>
    所以我就设置了这个参数，需要请求下一页的时候，这个参数就加1 <br/>


### 解析函数 parse {#解析函数-parse}

这个函数获取模特主页中每一张专辑的地址，然后传递给 `self.parseAlbum` 回调，并通过 `cb_kwargs` 添加额外的函数参数 <br/>

```python
def parse(self, response):
    urls = response.css('div.main div.boxs ul.img li>a::attr(href)').extract()
    albumUrls = list(map(lambda url: response.urljoin(url), urls))
    names = response.css('div.main div.boxs ul.img li p.p_title> a::text').extract()

    for (albumUrl, name) in zip(albumUrls, names):
	yield scrapy.Request(
	    url=albumUrl,
	    callback=self.parseAlbum,
	    cb_kwargs=dict(firstUrl=albumUrl, albumName=name, count=1))

```


### Pipeline 处理结果 {#pipeline-处理结果}

由于直接请求图片会被网站检测，并返回 403 代码，这里我们伪造下请求头 <br/>

```python
default_headers = {
    'Accept': 'image/avif,image/webp,*/*',
    'Accept-Encoding': 'gzip, deflate',
    'Accept-Language': 'zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2',
    'Host': 'image.meitulu.cn',
    'Referer': 'http://meitulu.cn/',
    'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:99.0) Gecko/20100101 Firefox/99.0'
}
```

接下来我们写一个类来继承 `FilesPipeline` ，处理结果 <br/>

```python
from scrapy.pipelines.images import FilesPipeline

class MeituluPipeline(FilesPipeline):
    default_headers = {
	'Accept': 'image/avif,image/webp,*/*',
	'Accept-Encoding': 'gzip, deflate',
	'Accept-Language': 'zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2',
	'Host': 'image.meitulu.cn',
	'Referer': 'http://meitulu.cn/',
	'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:99.0) Gecko/20100101 Firefox/99.0'
    }

    def file_path(self, request, response=None, info=None, *, item=None):
	dirname = item['albumName']
	basename = item['name']

	return os.path.join(dirname, basename)

    def get_media_requests(self, item, info):
	yield scrapy.Request(item['url'], headers=self.default_headers)

    def item_completed(self, results, item, info):
	return item
```

其中有三个函数可以挑选着来重写 <br/>

-   `file_path` <br/>
    指定 `item` 存贮的文件名 <br/>
-   `get_media_requests` <br/>
    自定义如何请求图片的函数 <br/>
-   `item_completed` <br/>
    下载完图片后如何处理 <br/>

另外这个 `file_path` 的重写我忘了从哪个教程里粘贴过来的了，先用着吧 <br/>


### 全局设置 {#全局设置}


#### 开启 pipeline {#开启-pipeline}

```python
ITEM_PIPELINES = {
   'meitulu.pipelines.MeituluPipeline': 300,
}

```


#### 设置 文件存贮 默认位置 {#设置-文件存贮-默认位置}

```python
FILES_STORE = '/home/steiner/workspace/meitulu/capture/'
```