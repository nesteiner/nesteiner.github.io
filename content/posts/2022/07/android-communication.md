+++
title = "Android 通信简单应用"
date = 2022-07-02T14:39:00+08:00
lastmod = 2022-07-02T14:48:05+08:00
tags = ["Android通信"]
categories = ["Android"]
draft = false
toc = true
+++

## Activity 之间通信 {#activity-之间通信}

从一个 `Activity` 跳转到另一个 `Activity` 时，可以携带一些数据进去，这个就是 `Activity` 之间的通信 <br/>
跳转到 `Activity` 之后可以响应通信，不过需要发起端调用 `startActivityForResult` ，接收端调用 `setResult` <br/>
这种通信方法我不能类比到组件之间的通信，组件之间通信都是单向数据流，这个是 **双向数据流** <br/>


### 原来的通信方法 {#原来的通信方法}


#### 通信发起端 {#通信发起端}

跳转需要借助 `Intent` 对象，指定来源的上下文和目标的类，调用 `startActivity` 即可 <br/>

```java
Intent intent = new Intent(this, TargetClass.class);
startActivity(intent)
```

如果需要携带数据，创建一个 `Bundle` 对象绑定到 `Intent` 即可 <br/>

```java
Intent intent = new Intent(this, TargetClass.class);
Bundle bundle = new Bundle();
bundle.putString("message", "Hello World");
intent.putExtras(bundle);
startActivity(intent);
```

其实也可以不用 `Bundle` ，直接给 `Intent` 来添加 <br/>


#### 通信接收端 {#通信接收端}

跳转到接收端后，使用 `getIntent` 来获取对端的 `Intent` 对象，并查看绑定的数据 <br/>

```java
Intent intent = getIntent();
Bundle bundle = intent.getExtras();
```


#### 双向数据流 {#双向数据流}

通信发起端调用 `startActivityForResult` 并携带数据是一个方向，通信接收端调用 `setResult` 并携带数据与对端是另一个方向 <br/>
在发起端 <br/>

```java
Intent intent = new Intent();
Bundle bundle = new Bundle();
bundle.putString("message", "Hello");
intent.putExtras(bundle);

startActivityForResult(intent, 2)
```

注意 `startActivityForResult` 第二个参数要大于0，才会接收对端的数据，并跳转到 `onActivityResult` 回调 <br/>

```java
protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    // 对端将数据写入 data ，可以从里面获取数据
}

```

在接收端回复 <br/>

```java
Intent intent = new Intent();
Bundle bundle = new Bundle();
bundle.putString("result", "Fuck You");

intent.putExtras(bundle);
setResult(RESULT_OK, intent);
finish() // 结束声明周期，跳转到原来的 Activity
```


### Result API 方法 {#result-api-方法}

`Result API` 是官方支持的方法，可以替代原来的通信方法，通信之间可以更加安全吧，应该 <br/>


#### 两边协定 {#两边协定}

在 `Result API` 下，两端的通信更像是函数，输入一个数据，函数会输出一个结果 <br/>
两边需要定义一个协议，规定输入的数据类型和输出的数据类型 <br/>

```java
public class ResultContract extends ActivityResultContract<String, String> 
```

为其定义 `createIntent` 方法，表示如何输入数据 <br/>

```java
public Intent createIntent(@NonNull Context context, String input) {
    Intent intent = new Intent(context, SecondActivity.class);
    intent.putExtra("name", input);
    return intent;
}

```

为其定义 `parseIntent` 方法，表示如何解析接收到的输出数据 <br/>

```java
public String parseResult(int resultCode, @Nullable Intent intent) {
    String data = intent.getStringExtra("result");
    if(resultCode == Activity.RESULT_OK && data != null) {
	return data;
    } else {
	return null;
    }
}
```


#### 输入端 {#输入端}

使用 `Result API` ，页面跳转的方式也要改变了，需要一个 `Launcher` 来进行操作 <br/>
这个 `Launcher` 的构造需要上面的两边协定和一个回调，这个回调用来处理 `paserResult` 后的数据 <br/>
这里设置用 `Toast` 回显 <br/>

```java
ActivityResultLauncher activityLauncher = registerForActivityResult(
	new ResultContract(), result -> {
	    Toast.makeText(getApplicationContext(), result, Toast.LENGTH_SHORT).show();
	});
```


#### 输出端 {#输出端}

输出端的做法没有多大改变，一样的 <br/>

```java
button.setOnClickListener(view -> {
	Intent intent = new Intent();
	Bundle bundle = new Bundle();
	bundle.putString("result", "Hello Mother Fucker");
	intent.putExtras(bundle);
	setResult(RESULT_OK, intent);
	finish();
    });
```


## Fragment 之间通信 {#fragment-之间通信}

`Framgent` 之间的通信也想函数那样，输入一个值，输出一个值 <br/>
不同的是，输入端需要在 `FragmentManager` 上设置一个 `Listener` 来接收输出值，并设置键 `requestKey` 提供位置 <br/>
输出端需要通过 `FragmentManager` 来设置输出值到 `requestKey` 上 <br/>
![](/ox-hugo/2022-03-08_20-46-33_screenshot.png) <br/>


### 输入端 {#输入端}

挑重要的来讲，输入端需要设置 `requestKey` 和处理输出值的回调 <br/>

```java
@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    getParentFragmentManager().setFragmentResultListener(
	    "requestKey",
	    this,
	    (requestKey, bundle) -> {
		String result = bundle.getString("bundleKey");
		resultTextView.setText("Fuck You" + result);
	    });
}

```


### 输出端 {#输出端}

```java
public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
    final View view = inflater.inflate(R.layout.fragment2, container, false);
    Button button = view.findViewById(R.id.button);
    button.setOnClickListener(_view -> {
	    Bundle result = new Bundle();
	    result.putString("bundleKey", "result");
	    getParentFragmentManager().setFragmentResult("requestKey", result);
	});

    return view;
}
```


### 补充 {#补充}

以上都是同一级的 `Fragment` 之间通信，没涉及到父子之间通信，其实也是类似的 <br/>
![](/ox-hugo/2022-03-08_20-50-31_screenshot.png) <br/>
只需将 `getParentFragmentManager` 改为 `getChildFragmentManager` 即可 <br/>
注意，他们都是用同一个 `FragmentManager` 来通信的