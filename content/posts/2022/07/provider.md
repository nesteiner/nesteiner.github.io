+++
title = "Flutter 中 provider 的使用"
date = 2022-07-02T01:43:00+08:00
lastmod = 2022-07-02T14:43:18+08:00
tags = ["Flutter"]
categories = ["Flutter"]
draft = false
toc = true
+++

## Provider 的引入 {#provider-的引入}

Provider 解决的是组件之间传递数据的问题 <br/>
在 Vue 中，父组件通过 props 传递数据给子组件，子组件通过 $emit 通知父组件，然后 Vue 重新渲染 <br/>
DOM <br/>
在 Vue 中，也可以使用 provider 和 inject 传递数据，不过不能很好的处理响应性 <br/>

与 Vue 相同的是，Flutter 的 Provider 的引入解决了数据传递中嵌套组件过多的问题，若在当前节点没有找到 Provider 组件， <br/>
则会通过上下文寻找 Provider ，直至找到 <br/>

不过想使用 Provider，父组件得暴露出一个对象用以交互， ****而这个对象得继承自 ChangeNotifier**** <br/>
以 ChangeNotifierProvider为例 <br/>

```dart
ChangeNotifierProvider(
  builder: (context) => ExposedObject,
  child: ...
)
```

子组件想使用其中的值，得通过 Consumer 组件或者 context.read&lt;ExposedType&gt; <br/>

```dart
Consumer(
  create: (context, ExposedType expose, child) {
    // call expose
  }
)
```

```dart
var expose = context.read<ExposedType>();
```

****注意**** <br/>

1.  除了 context.read 外，还有 context.watch <br/>
2.  context.watch 会监听 Provider 的变化，调用后会更改页面的渲染， <br/>
3.  context.read 不会监听 Provider 的变化，调用后不会更改页面的渲染，所以不要用在 Widget 的 build 方法中 <br/>


## Provider 与 StatefulWidget {#provider-与-statefulwidget}

Provider 与 StatefulWidget 类似，都是把组件和状态分离开来， <br/>
其中 Statefulwidget 使用的是 State 类型来 build 组件，通过 setState 来通知变化 <br/>
而 Provider 渲染的是其 child 组件，暴露出的组件通过调用 notifyListener 来通知变化，这个时候 Flutter 会丢弃 <br/>
修改过的组件，直接换新的 <br/>


## Counter with Provider {#counter-with-provider}

首先是拥有 Provider 的父组件 App <br/>

```dart
class App extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Counter with Provider',
      home: Scaffold(
	appBar: AppBar(title: Text('Counter with Provider'),),
	body: ChangeNotifierProvider(
	  create: (_) => CounterState(),
	  child: Counter(),
	),
      )
    );
  }
}

```

接着是暴露出的对象 CounterState <br/>

```dart
class CounterState extends ChangeNotifier {
  int count = 0;

  void increment() {
    count += 1;
    notifyListeners();
  }
}
```

再是使用 Provider 的子组件 Counter <br/>

```dart
class Counter extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
	Text('You have pushed button many times: '),
	Text(context.watch<CounterState>().count.toString(), style: TextStyle(fontSize: 16)),
	FlatButton(
	  child: Text('click me'),
	  onPressed: () {
	    var state = context.read<CounterState>();
	    state.increment();
	  },
	),
      ]
    );
  }
}
```

不过我还是更推荐用 Consumer ，简洁明了 <br/>

```dart
@override
Widget build(BuildContext context) {
  return Consumer(
    builder: (context, CounterState state, child) {
      return Column(
	children: [
	  Text('You have pushed button many times: '),
	  Text(state.count.toString(), style: TextStyle(fontSize: 16)),
	  FlatButton(
	    child: Text('Click me'),
	    onPressed: () {
	      state.increment();
	    }
	  )
	]
      )
    }
  )
}
```


## 总结 {#总结}

总的来说，使用 Provider 需要注意几点 <br/>

1.  提供暴露的接口 <br/>
2.  状态的更改在接口内部 <br/>
3.  状态的更改需要通知 Provider <br/>