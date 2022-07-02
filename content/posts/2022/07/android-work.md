+++
title = "Android 大作业报告"
date = 2022-07-02T14:29:00+08:00
lastmod = 2022-07-02T14:44:50+08:00
tags = ["Flutter"]
categories = ["Flutter"]
draft = false
toc = true
+++

## 项目介绍 {#项目介绍}

这次的大作业用的 **Flutter** 框架，虽然构建 UI 界面很容易，难的是各个组件之间的通信，以及对 async/await 的异步操作 <br/>
这个应用由于时间原因选择了 **Flutter** 来开发，只实现了登录界面，笔记界面，通知界面 <br/>
由于还没有对应用进行优化，解耦，数据库的操作只有应用开启时的读取数据，删除，添加，修改操作都没有实现 <br/>


## 功能结构 {#功能结构}


### 登录界面 {#登录界面}

这里只提供了登录界面，没有提供登录的相关操作，不过表单还是有的 <br/>

{{< figure src="/ox-hugo/2021-06-24_02-28-03_screenshot.png" >}} <br/>


### 笔记界面 {#笔记界面}

笔记界面的每一项由一个 NoteCard 组件构成，只提供了编辑功能，路由切换后会重置 <br/>

{{< figure src="/ox-hugo/2021-06-24_02-35-06_screenshot.png" >}} <br/>

需要编辑时点击右边的三点按钮，扩展的编辑面板显示 <br/>

{{< figure src="/ox-hugo/2021-06-24_02-36-12_screenshot.png" >}} <br/>

编辑完成后，点击 Save 保存之后 <br/>

{{< figure src="/ox-hugo/2021-06-24_02-36-52_screenshot.png" >}} <br/>


### 消息界面 {#消息界面}

消息界面完全仿照 原来的应用，没有什么稀奇的 <br/>

{{< figure src="/ox-hugo/2021-06-24_02-37-47_screenshot.png" >}} <br/>


## 设计类 {#设计类}

这里介绍下这个项目的结构，重点不是代码，从入口讲起 <br/>


### 入口类 {#入口类}

```dart
void main() => runApp(App());

class App extends StatelessWidget {
  final title = 'HomeApp';
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return MaterialApp(
      title: title,
      home: Home(),
    );
  }
}

```


### 主页的设计 {#主页的设计}

主页中由三个页面，需要引入一个 TabView 组件来进行路由切换 <br/>

其中主页为 <br/>

```javascript
class Home extends StatefulWidget {
  @override
  HomeState createState() => HomeState();
}
```

主页的构建由另一个类 HomeState 来实现，在其中使用 TabView <br/>

```dart
class HomeState extends State<Home> with SingleTickerProviderStateMixin{
  late final TabController controller = TabController(length: 3, vsync: this);

  @override
  void dispose() {
    // TODO: implement dispose
    controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return Scaffold(
      appBar: AppBar(
	bottom: TabBar(
	  controller: controller,
	  tabs: [
	    Tab(icon: Icon(Icons.login),),
	    Tab(icon: Icon(Icons.note),),
	    Tab(icon: Icon(Icons.message),),
	  ],
	),

	title: Text('Keep Alive'),
      ),

      body: TabBarView(
	controller: controller,
	children: [
	  LoginRoute(),
	  NoteRoute(),
	  NotificationRoute(),
	],
      ),
    );
  }

}

```


### Login界面的设计 {#login界面的设计}

Login 的界面构建有些复杂，需要将其分解为几个部分 <br/>

1.  buildTopBannerWidget <br/>
2.  buildAccountLoginTip <br/>
3.  buildEditWidget <br/>
4.  buildLoginButton <br/>
    
    之后再将这些组件整合到一个 Column 中即可 <br/>
    需要注意的是，对文本框进行编辑时，会出现底部空间不够绘制的情况，需要在 build 方法中使用 SingleChildScrollView 来处理 <br/>
    ```dart
    @override
    Widget build(BuildContext context) {
      return SingleChildScrollView(
        child: buildColumn(context),
      );
    }
    ```


### 笔记界面的设计 {#笔记界面的设计}

在应用打开的时候，需要从数据库读取，再生成数据对象 NoteCard <br/>

```dart
class NoteCard extends StatefulWidget {
  String title;
  String content;

  NoteCard({required this.title, required this.content});

  @override
  CardState createState() => CardState(title: title, content: content);
}
```

通知界面也是如此 <br/>
有关数据库的操作写在下面 <br/>


## 数据库 {#数据库}

由于 dart 是单线程的语言，打开数据库需要一定的时间，为了不影响整个程序的效率，dart 使用了异步的概念 <br/>
这里将数据库的增，删，改，查操作都写在数据模型类中 <br/>


### 数据模型的操作 {#数据模型的操作}

存储笔记的数据模型 note 中 <br/>

```dart
static Future<void> insert(Database db, Note note) async {
  await db.insert(
    tableName,
    note.toMap(),
    conflictAlgorithm: ConflictAlgorithm.replace
  );
}

static Future<void> delete(Database db, int id) async {
  await db.delete(tableName, where: 'id = ?', whereArgs: [id]);
}

static Future<void> update(Database db, Note note) async {
  await db.update(
    tableName, note.toMap(), where: 'id = ?', whereArgs: [note.id]);
}

static Future<List<Note>> all(Database db) async {
  final List<Map<String, dynamic>> maps = await db.query(tableName);
  return List.generate(
    maps.length,
    (index) =>
    Note(
      id: maps[index]['id'],
      title: maps[index]['title'],
      content: maps[index]['content']
  ));
}

```

类似的，存储消息的数据模型 notify 中 <br/>

```dart
static Future<void> insert(Database db, Notify notification) async {
  await db.insert(
    tableName,
    notification.toMap(),
    conflictAlgorithm: ConflictAlgorithm.replace
  );
}

static Future<void> delete(Database db, int id) async {
  await db.delete(tableName, where: 'id = ?', whereArgs: [id]);
}

static Future<void> update(Database db, Notify notification) async {
  await db.update(
    tableName, notification.toMap(), where: 'id = ?', whereArgs: [notification.id]);
}

static Future<List<Notify>> all(Database db) async {
  final List<Map<String, dynamic>> maps = await db.query(tableName);
  return List.generate(
    maps.length,
    (index) =>
    Notify(
      id: maps[index]['id'],
      className: maps[index]['className'],
      notifyContent: maps[index]['notifyContent']
  ));
}
```


### 组件的构建 {#组件的构建}

这里为什么要单独讲组件的构建，因为这里牵扯到数据的异步加载，还有 build 方法不能定义为异步的，具体参照[这篇文章](https://flutterigniter.com/build-widget-with-async-method-call/) <br/>
另外这里应用的构建代码定义在路由界面中 <br/>
在 NoteRoute 中，先定义一个异步回调 <br/>

```dart
Future<List<NoteCard>> callAsyncFetch() async {
  final database = openDatabase(
    join(await getDatabasesPath(), 'todolist.db')
  );

  var cards = await Note.all(await database);
  return cards.map((e) => NoteCard(title: e.title, content: e.content)).toList();
}
```

再将 build 的返回值改为 FutureBuilder ，指定 future 与 builder <br/>

```dart
@override
Widget build(BuildContext context) {
  // TODO: implement build
  return FutureBuilder(
    future: callAsyncFetch(),
    builder: (context, AsyncSnapshot<List<NoteCard>> snapshot) {
      if(snapshot.hasData) {
	return ListView(
	  children: snapshot.data!,
	);
      } else {
	return CircularProgressIndicator();
      }
    }
  );
}

```