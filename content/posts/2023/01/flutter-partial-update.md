+++
title = "Flutter 局部更新"
date = 2023-01-08T15:09:00+08:00
lastmod = 2023-02-07T12:32:26+08:00
tags = ["Flutter"]
categories = ["Flutter"]
draft = false
toc = true
+++

## Flutter 局部更新 {#flutter-局部更新}


### StatefulBuilder {#statefulbuilder}

```dart
Widget build(BuildContext context) {
  return Column(
    children: [
      buildCheapUpdateWidget(context),
      buildExpensiveUpdateWidget(context)
    ]
  )
}
```

在上述代码中，如果调用了 `setState` 更新状态，整个组件会重新 `build` 一遍，届时 `buildExpensiveUpdateWidget` 会 <br/>
重新调用一遍，这是我们引入局部更新组建 <br/>

```dart
StatefulBuilder(
  builder: (context, setState) => Widget
)
```

更新时用 `builder` 中的 `setState` 即可 <br/>

正是有了局部更新，我们可以解决在 `Dialog` 中 `setState` 不起作用的问题，详情看 [这篇文章](https://stackoverflow.com/questions/51962272/how-to-refresh-an-alertdialog-in-flutter) <br/>


### Selector {#selector}

`Selector` 是另一种状态管理方式 `Provider` 的局部更新方法 <br/>
在使用 `Consumer` 的时候，如果 `ChangeNotifier` 调用了 `notifyListeners` ，所有监听的 `Consumer` 都会更新， <br/>
但是没那个必要，所以我们引入局部更新来避免不必要的更新 <br/>

```dart
Selector<State, T>(
  selector: (context, state) => state.value,
  builder: (context, value, child) => Widget
)
```

注意 <br/>
目前的理解，需要搭配 `context.read` 使用 <br/>