+++
title = "创建一个时钟组件"
date = 2023-02-06T20:05:00+08:00
lastmod = 2023-02-07T12:27:16+08:00
tags = ["笔记"]
categories = ["Flutter"]
draft = false
toc = true
+++

## 指针绘制 {#指针绘制}

给定时，分，秒，绘制出指针在表盘上的位置 <br/>


### 时针 {#时针}

给定 时，分 <br/>

```dart
class HourHand extends CustomPainter {
  int hours;
  int minutes;
  final Paint painter; // 1

  HourHand({
      required this.hours,
      required this.minutes
  }): painter = Paint()
  ..color = Colors.black
  ..strokeWidth = 20
  ..style = PaintingStyle.stroke;

  @override
  void paint(Canvas canvas, Size size) {
    final radius = size.width / 2; // 2
    canvas.save(); // 3

    canvas.translate(radius, radius); // 4

    canvas.rotate(hours >= 12
      ? 2 * pi * ((hours - 12) / 12 + minutes / 720)
      : 2 * pi * (hours / 12 + minutes / 720)); // 5

    Path path = Path();
    path.lineTo(0, -radius + radius / 4); // 6
    canvas.drawPath(path, painter);
    canvas.restore(); // 7
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) {
    return true; // 8
  }
}
```

在代码中 <br/>

1.  定义了画笔 `painter` ，定义他的颜色为黑色，画笔样式为描线，描线粗细为20 <br/>
2.  设置半径为 `width` 的一半 <br/>
3.  画布保存当前位置 <br/>
4.  画布移动 <br/>
5.  画布旋转，方便后续画线， **这一点最为重要** <br/>
6.  画线 <br/>
7.  画布恢复到原来位置 <br/>
8.  `shouldRepaint` 设置为 `true` <br/>


### 分针 {#分针}

给定分，秒，绘制分针在表盘的位置 <br/>

```dart
class MinuteHand extends CustomPainter {
  int minutes;
  int seconds;
  final Paint painter;

  MinuteHand({
    required this.minutes,
    required this.seconds
  }): painter = Paint()
    ..color = Colors.black
    ..strokeWidth = 10
    ..style = PaintingStyle.stroke;

  @override
  void paint(Canvas canvas, Size size) {
    final radius = size.width / 2;
    canvas.save();
    canvas.translate(radius, radius);
    Path path = Path();
    canvas.rotate(2 * pi * (minutes + seconds / 60) / 60);
    path.lineTo(0.0, -radius - 10);
    canvas.drawPath(path, painter);
    canvas.restore();
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) {
    return true;
  }
}

```


### 秒针 {#秒针}

给定秒，绘制出秒针在表盘的位置 <br/>

```dart
class SecondHand extends CustomPainter {
  int seconds;
  final Paint painter;

  SecondHand({
    required this.seconds
  }): painter = Paint()
    ..color = Colors.red
    ..strokeWidth = 2.0
    ..style = PaintingStyle.stroke;

  @override
  void paint(Canvas canvas, Size size) {
    final radius = size.width / 2;
    canvas.save();

    canvas.translate(radius, radius);
    Path path = Path();
    canvas.rotate(2 * pi * seconds / 60);
    path.moveTo(0.0, -radius);
    path.lineTo(0.0, radius / 4);

    canvas.drawPath(path, painter);
    canvas.restore();
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) {
    return true;
  }
}
```


### 将指针绘制在一起 {#将指针绘制在一起}

```dart
class ClockHands extends StatelessWidget {
  final DateTime dateTime; // 1
  ClockHands({
    required this.dateTime
  });

  @override
  Widget build(BuildContext context) {
    return AspectRatio(
      aspectRatio: 1.0,
      child: Container(
	width: double.infinity, // 2 
	padding: const EdgeInsets.all(20.0),
	child: Stack(
	  fit: StackFit.expand, // 3
	  children: [
	    CustomPaint(
	      painter: HourHand(
		hours: dateTime.hour,
		minutes: dateTime.minute
	      ),
	    ),

	    CustomPaint(
	      painter: MinuteHand(
		minutes: dateTime.minute,
		seconds: dateTime.second
	      ),
	    ),

	    CustomPaint(
	      painter: SecondHand(
		seconds: dateTime.second
	      ),
	    )
	  ],
	),
      ),
    );
  }
}
```

在代码中 <br/>

1.  给定 `dateTime` ，一次性绘制出时针，分针，秒针 <br/>
2.  给定 `width` 为 `infinity` ，表示宽度可以随着窗口的变化而变化，高度呢，有父组件 `AspectRatio` 限定宽高比 <br/>
3.  由于没有设置 `CustomPaint` 的大小，如果 `fit` 设置为 `loose` ，那么 `CustomPaint` 就没有大小了 <br/>


## 表盘绘制 {#表盘绘制}

表盘需要绘制刻度，每当刻度是5的倍数时，加粗这个刻度 <br/>

```dart
class ClockDial extends CustomPainter {
  final hourTickMarkLength = 10.0;
  final minuteTickMarkLength = 5.0;
  final hourTickMarkWidth = 3.0;
  final minuteTickMarkWidth = 1.5;

  final Paint tickpainter;

  ClockDial(): tickpainter = Paint()
    ..color = Colors.blueGrey
    ..style = PaintingStyle.stroke;


  @override
  void paint(Canvas canvas, Size size) {
    const angle = 2 * pi / 60; // 1
    final radius = size.width / 2;
    double tickMarkLength = 0;

    canvas.save();
    canvas.translate(radius, radius);
    for (int i = 0; i < 60; i += 1) {
      tickMarkLength = i % 5 == 0 ? hourTickMarkLength : minuteTickMarkLength; // 2
      tickpainter.strokeWidth = i % 5 == 0 ? hourTickMarkWidth : minuteTickMarkWidth; // 3
      Path path = Path();
      path.moveTo(0.0, -radius);
      path.lineTo(0.0, -radius + tickMarkLength);
      canvas.drawPath(path, tickpainter);
      canvas.rotate(angle); // 4
    }

    canvas.restore();
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) {
    return false; // 5
  }
}
```

在代码中 <br/>

1.  `angle` 指的是每个刻度之间的间隔角度，由于有60个刻度，所以除以60 <br/>
2.  `tickMarkLength` 指的是刻度的长度，如果当前刻度是5的倍数，那么加长 <br/>
3.  `tickMarkWidth` 指的是刻度的宽度，如果当前刻度是5的倍数，那么加粗 <br/>
4.  `canvas.rotate` 默认是顺时针旋转，每次旋转一刻度，降低代码画出刻度的难度 <br/>
5.  由于表盘是静态的，不需要随着时间变化而变化， `shouldRepaint` 设置为 false <br/>


## Clock 组件 {#clock-组件}

首先，我们把表盘和指针组合在一起 <br/>

```dart
class ClockFace extends StatelessWidget {
  final DateTime dateTime;

  const ClockFace({
    required this.dateTime
  });

  @override
  Widget build(BuildContext context) {
    final innerStack = Stack(
      children: [
	Container(
	  width: double.infinity,
	  height: double.infinity,
	  padding: const EdgeInsets.all(10.0),
	  child: CustomPaint(
	    painter: ClockDial(),
	  ),
	),

	ClockHands(dateTime: dateTime)
      ],
    );

    final child = AspectRatio(
      aspectRatio: 1.0,
      child: Container(
	width: double.infinity,
	decoration: const BoxDecoration(
	  shape: BoxShape.circle,
	  color: Colors.white
	),

	child: innerStack,
      ),
    );

    return Padding(
      padding: const EdgeInsets.all(10.0),
      child: child,
    );
  }
}

```

现在，我们定义一个 `StatefulWidget` <br/>

```dart
class Clock extends StatefulWidget {
  @override
  ClockState createState() => ClockState();
}
```

其中 `ClockState` 为 <br/>

```dart
class ClockState extends State<Clock> {
  late Timer timer;
  late DateTime dateTime;

  @override
  void initState() {
    super.initState();
    dateTime = DateTime.now(); // 1
    timer = Timer.periodic(const Duration(seconds: 1), setTime); // 2
  }

  @override
  void dispose() {
    timer.cancel();
    super.dispose();
  }

  void setTime(Timer timer) {
    setState(() {
      dateTime = dateTime.add(const Duration(seconds: 1));
    });
  }

  @override
  Widget build(BuildContext context) {
    return AspectRatio(
      aspectRatio: 1,
      child: buildClockCircle(context),
    );
  }

  Container buildClockCircle(BuildContext context) {
    return Container(
      width: double.infinity,
      decoration: const BoxDecoration(
	shape: BoxShape.circle,
	color: Colors.black
      ),
      child: ClockFace(dateTime: dateTime),
    ); // 3
  }
}

```

在代码中 <br/>

1.  初始化状态时，设置 `dateTime` 为现在的时间 <br/>
2.  初始化状态时，设置 `timer` 每秒执行一次 `setTime` <br/>
3.  通过设置 `decoration` 的 `shape` 和 `color` ，用黑色描出一个圆，作为表盘的边缘 <br/>


## 展示 {#展示}

{{< figure src="/ox-hugo/2023-02-07_12-26-45_screenshot.png" >}} <br/>