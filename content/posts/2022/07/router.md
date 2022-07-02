+++
title = "Flutter 中的Router"
date = 2022-07-02T01:43:00+08:00
lastmod = 2022-07-02T14:43:30+08:00
tags = ["Flutter路由"]
categories = ["Flutter"]
draft = false
toc = true
+++

## 用你的应用来演示 路由用法 {#用你的应用来演示-路由用法}

在这个演示里定义两个界面，HomePage 与 NextPage ，并从 HomePage 传递参数到 NextPage 中 <br/>
接下来演示匿名路由和命名路由的情况 <br/>

这是 **HomePage** <br/>

{{< figure src="/ox-hugo/2021-06-13_14-36-32_screenshot.png" >}} <br/>

点击文字按钮后，跳转到 NextPage <br/>

{{< figure src="/ox-hugo/2021-06-13_14-37-33_screenshot.png" >}} <br/>


### 1. 匿名路由跳转 {#1-dot-匿名路由跳转}

首先是 HomePage <br/>

```dart
class HomePage extends StatelessWidget {

  Widget buildBody(BuildContext context) {
    var fnpress = (String message) {
      return () {
	Navigator.push(context, MaterialPageRoute(builder: (builder) => NextPage(message: message)));
      };
    };

    return Center(
      child: Column(
	mainAxisAlignment: MainAxisAlignment.center,
	crossAxisAlignment: CrossAxisAlignment.center,
	children: [
	  TextButton(onPressed: fnpress('Hello World'), child: Text('goto the next page with status 1')),
	  TextButton(onPressed: fnpress('Fuck You'), child: Text('goto the next page with status 2'))
	],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return Scaffold(
      appBar: AppBar(title: Text('Home Page'),),
      body: buildBody(context),
    );
  }
}

```

在跳转时通过构造函数来传递参数，其中 NextPage 为 <br/>

```dart
class NextPage extends StatelessWidget {
  String message;
  NextPage({required this.message}): super();

  Widget buildBody(BuildContext context) {
    return Center(
      child: Text('the message from HomePage is' + message, style: TextStyle(color: Colors.red, fontSize: 40),),
    );
  }
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return Scaffold(
      appBar: AppBar(title: Text('Next Page',)),
      body: buildBody(context),
    );
  }
}
```


### 2. 命名路由跳转 {#2-dot-命名路由跳转}

命名路由需要在入口程序 App 中注册路由表， <br/>

```dart
class App extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return MaterialApp(
      title: 'Flutter Router Demo',
      home: HomePage(),
      routes: {
	"Home": (ctx) => HomePage(),
	"Next": (ctx) => NextPage(),
      },
    );
  }
}
```

在 HomePage 中跳转与传递参数方式已经改变 <br/>

```dart
// in HomePage class, 
  Widget buildBody(BuildContext context) {
    var fnpress = (String message) {
      return () {
	Navigator.pushNamed(context, "Next", arguments: message);
      };
    };
  }

```

参数的传递不通过构造方法， NextPage 接收参数的方式变为了 <br/>

```dart
Widget buildBody(BuildContext context) {
  var message = ModalRoute.of(context)!.settings.arguments as String;

  return Center(
    child: Text('the message from HomePage is ' + message, style: TextStyle(color: Colors.red, fontSize: 40),),
  );
}

```


## 与 HTML 类比 {#与-html-类比}

类比 html 中的路由跳转，路由是从一个页面地址跳转到另一个页面地址，而不变的对象是浏览器的标签页 <br/>
在 这个程序 中，页面地址可以看作 Scaffold ，而标签页看作 MaterialApp <br/>
接下来是如何跳转的问题，在 html 中使用 &lt;a&gt; 标签跳转，flutter 则是 <span class="underline">Navigator.push(context, MaterialPageRoute)</span> <br/>
还有就是跳转时如何传递参数，html 可以通过添加查询参数， <br/>
flutter 则有两种情况，在非命名的路由中，只需修改 MaterialPageRoute 的 builder 方法，传递数据至组件的构造方法中即可 <br/>

```dart
Navigator.push(context, MaterialPageRoute(builder: (ctx) {
  return NextPage(
    data: "Home Page Data",
  );
}));
```

而在命名路由中，可以通过专门的参数 argument 来传递 <br/>

```dart
Navigator.pushNamed(context, "NextPage", arguments: "传值");
```

到目标路由后，通过 <br/>

```dart
ModalRoute.of(context)!.settings.arguments as 'type you want';
```

来接收参数，同时要注意 **null safety**