+++
title = "Flutter Counter"
date = 2022-07-02T01:42:00+08:00
lastmod = 2022-07-02T14:43:02+08:00
tags = ["Flutter"]
categories = ["Flutter"]
draft = false
toc = true
+++

## 如何写一个 Flutter 应用 {#如何写一个-flutter-应用}

我们从一个计数器讲起， 首先建立 **Counter** 类 <br/>

```dart
class Counter extends StatelessWidget {
  Counter(): super();
  @override
  Widget build(BuildContext context) {
	  // TODO: implement build
	  return MaterialApp(
	    title: 'Flutter Counter',
	    theme: ThemeData(primarySwatch: Colors.blue),
	    home: CounterHomePage(title: 'Flutter Counter Home Page'),
	  );
  }
}
```

在内部定义了构造方法，并且重构了 <span class="underline"><span class="underline">build</span></span> 方法 <br/>
其中  <span class="underline"><span class="underline">build</span></span> 方法返回了一个 <span class="underline"><span class="underline">MaterialApp</span></span> ，这就是 <span class="underline"><span class="underline">Flutter</span></span> 提供的根组件，接下来只需提供一个 <span class="underline"><span class="underline">Widget</span></span> ，将其赋值给 <span class="underline"><span class="underline">home</span></span> 作为主页程序即可 <br/>
再来看看赋值 <span class="underline"><span class="underline">home</span></span> 的组件 <span class="underline"><span class="underline">CounterHomePage</span></span> 是怎么写的 <br/>

```dart
class CounterHomePage extends StatefulWidget {
  CounterHomePage({this.title = 'Hello World'}): super();
  String title;

  @override
  CounterHomePageState createState() => CounterHomePageState();
}
```

这个类继承自 <span class="underline"><span class="underline">StatefulWidget</span></span> ，与 <span class="underline"><span class="underline">Counter</span></span> 类继承的 <span class="underline"><span class="underline">StatelessWidget</span></span> 成对比， <span class="underline">Counter</span> 是无状态的组件，  <span class="underline">CounterHomePage</span> 是有状态的组件 <br/>
其中重写了 <span class="underline">StatefulWidget</span> 的 <span class="underline">createState</span> 方法，返回值是用来管理其状态的 <span class="underline">CounterHomePageState</span> <br/>

```dart
class CounterHomePageState extends State<CounterHomePage> {
  int count = 0;

  void incrementCount() {
    setState(() {
	      count += 1;
    });
  }

  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return Scaffold(
      appBar: AppBar(title: Text(widget.title),),
      body: Center(
	      child: Column(
		mainAxisAlignment: MainAxisAlignment.center,
		children: <Widget>[
		  Text('You have pushed the button this many times:'),
		  Text(count.toString(), style: Theme
		    .of(context)
		    .textTheme
		    .headline4),
		  TextButton(
		    child: Text('Open new route'),
		    onPressed: () {
			    Navigator.push(context, MaterialPageRoute(builder: (context){
				  return NewRoute();
			    }));
		  },)
		]
	      ),
      ),
      floatingActionButton: FloatingActionButton(
	      onPressed: incrementCount,
	      tooltip: 'Increment',
	      child: Icon(Icons.add),
      ),
    );
  }
}
```

其中该类继承自泛型类 <span class="underline">State&lt;CounterHomePage&gt;</span> ，与 <span class="underline">CounterHomePage</span> 相关联 <br/>
需要改变状态的时候，调用 <span class="underline">setState</span> 即可，该函数接收一个闭包。 <br/>
而后， Flutter 框架检测到状态改变，销毁原来 <span class="underline">build</span> 的 <span class="underline">CouterHomePageState</span> ，而是重新调用 <span class="underline">build</span> 来代替 <br/>
在程序中，改变状态通过 <span class="underline">TextButton</span> 的 <span class="underline">OnPressed</span> 回调来触发 <br/>

还有一个程序的入口，主函数没有写 <br/>

```dart
void main() => runApp(Counter());
```

至此，我们的第一个计数器应用就搭建完毕了 <br/>

{{< figure src="/ox-hugo/2021-06-12_16-10-49_screenshot.png" >}} <br/>


## 什么是 Scaffold 组件 {#什么是-scaffold-组件}

在上面的 <span class="underline">CounterHomePageState</span> 中，还有个 <span class="underline">Scaffold</span> 组件没有讲，这是 <span class="underline">Flutter</span> 为我们提供的主页， <br/>
官网的描述：Material Design布局结构的基本实现， 类提供了用于显示drawer、snackbar和底部sheet的API。 <br/>
可以编辑他的 <span class="underline">appBar</span> , <span class="underline">body</span> 属性，其中 <span class="underline">body</span> 作为主页的主体，随便放个组件进去就行了 <br/>


## 如何布局多个组件 {#如何布局多个组件}

<span class="underline">Scaffold</span> 的 <span class="underline">body</span> 只能传递一个组件，想把一堆东西塞进去，可以使用 <span class="underline">Flutter</span> 提供的 **布局组件** 该类组件都有一个 <span class="underline">children</span> 属性，表示一个 <span class="underline">Widget</span> 数组 <br/>
在上面的例子中， <span class="underline">Column</span> 就是一个布局组件，按竖直方向布局组件，并赋值 <span class="underline">mainAxisalignment</span> 为 <span class="underline">center</span> 表示主轴方向垂直对齐 <br/>

---

等等， <span class="underline">Column</span> 上面还有个 <span class="underline">Center</span> 组件是怎么回事? 他为什么用 <span class="underline">child</span> ，而不用 <span class="underline">children</span> ? <br/>
<span class="underline">Center</span> 是一个容器组件，只有一个子元素，用 <span class="underline">child</span> 表示 <br/>

布局类Widget是按照一定的排列方式来对其子Widget进行排列； <br/>
而容器类Widget一般只是包装其子Widget，对其添加一些修饰（补白或背景色等）、变换(旋转或剪裁等)、或限制(大小等)。 <br/>
注意，Flutter官方并没有对Widget进行官方分类，我们对其分类主要是为了方便讨论和对Widget功能区分的记忆 <br/>

除了 <span class="underline">Center</span> 外，其他容器类还有 <br/>

1.  填充（Padding） <br/>
2.  尺寸限制类容器（ConstrainedBox等） <br/>
3.  装饰容器（DecoratedBox） <br/>
4.  变换（Transform） <br/>
5.  Container容器 <br/>
6.  Scaffold、TabBar、底部导航 <br/>
7.  剪裁（Clip） <br/>


## 如何修饰样式 {#如何修饰样式}

像 <span class="underline">css</span> 样式一样， <span class="underline">flutter</span> 中样式也可以继承，如果要更改，用 <span class="underline">Theme</span> 组件包装一下就可以了 <br/>
其中 <span class="underline">Theme</span> 组件 <br/>

```dart
Theme({Key? key, required ThemeData data, required Widget child})
```

最重要的是 [ThemeData](https://api.flutter.dev/flutter/material/ThemeData-class.html) 这个结构， child 表示需要修饰的组件 <br/>


## 对比 html 语法 {#对比-html-语法}

对比 html 的标签语法，比如 <br/>

```html
<div>

</div>

<style>
  div {
  padding: 12px;
  }
</style>
```

可以用 <span class="underline">Padding</span> 组件写为 <br/>

```dart
Padding(
  padding: EdgeInsets.all(12)
);
```

感觉是把一些属性单独提取出来，做成一个组件，如果需要修饰大量样式，应该用 <span class="underline">Theme</span> 组件 <br/>