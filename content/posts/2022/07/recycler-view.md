+++
title = "RecyclerView 简单使用"
date = 2022-07-02T14:36:00+08:00
lastmod = 2022-07-02T14:46:50+08:00
tags = ["RecyclerView"]
categories = ["Android"]
draft = false
toc = true
+++

## RecyclerView {#recyclerview}

这个 `View` 的使用当初我用的不是很明白，现在有点懂了，我们自顶向下设计 `View` <br/>


### 代码总览 {#代码总览}

在 `MainActivity` 中， <br/>

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    RecyclerView recyclerView = (RecyclerView) findViewById(R.id.recyclerview);
    List<Integer> data = Stream.iterate(1, n -> n + 1).limit(100).collect(Collectors.toList());
    CustomAdapter adapter = new CustomAdapter(data);
    RecyclerView.LayoutManager layoutManager = new GridLayoutManager(this, 4);
    recyclerView.setAdapter(adapter);
    recyclerView.setLayoutManager(layoutManager);
}

```

我们 <br/>

1.  定义了一个 `RecyclerView` <br/>
2.  定义了一个 `CustomAdapter` <br/>
3.  为 `CustomAdapter` 提供了数据 <br/>
4.  为 `RecyclerView` 设置 `adapter` <br/>
5.  为 `RecyclerView` 设置 `layoutmanager` <br/>

注意，这里我们没用到 `ViewHolder` 这个东西，这个东西由 `Adapter` 操纵，相当于 <br/>

-   `RecyclerView` <br/>
    -   `Adapter` <br/>
        -   `ViewHolder` <br/>
        -   `ViewHolder` <br/>
        -   `ViewHolder` <br/>
        -   `ViewHolder` <br/>


### 有关Adapter {#有关adapter}

定义 `Adapter` 需要继承 `RecyclerView.Adapter` ，他的作用是衔接 `RecyclerView` 与 `ViewHolder` ，即 `RecyclerView` 通过 <br/>
`Adapter` 来设置每个 `ViewHolder` 的视图，这里我们需要设置三个关键方法 <br/>

-   `onCreateViewHolder` <br/>
    每当 `RecyclerView` 需要创建新的 ViewHolder 时，它都会调用此方法n <br/>
    此方法会创建并初始化 `ViewHolder` 及其关联的 `View` ，但不会填充视图的内容，因为 `ViewHolder` 此时尚未绑定到具体数据。 <br/>
-   `onBindViewHolder` <br/>
    `RecyclerView` 调用此方法将 `ViewHolder` 与数据相关联 <br/>
-   `getItemCount` <br/>
    `RecyclerView` 调用此方法来获取数据集的大小 <br/>

这里我们为 `Adapter` 设置私有数据 `data` <br/>

```java
public class CustomAdapter extends RecyclerView.Adapter<CustomViewHolder> {
    private List<Integer> data;

    public CustomAdapter(List<Integer> data) {
	this.data = data;
    }

    @NonNull
    @Override
    public CustomViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
	final View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_layout, parent, false);
	return new CustomViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull CustomViewHolder holder, int position) {
	holder.setText(String.format("this is item %d", data.get(position)));
    }

    @Override
    public int getItemCount() {
	return data.size();
    }
}

```

由于上面的代码使用到了 `ViewHolder` ，也要看下面的代码才能理解 <br/>


### 有关ViewHolder {#有关viewholder}

`ViewHolder` 用来表示一个布局，因此需要为其设置布局和类代码，这里随便布局一下 <br/>

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	      android:layout_width="match_parent"
	      android:layout_height="wrap_content">
  <TextView android:id="@+id/textview"
	    android:layout_width="match_parent"
	    android:layout_height="wrap_content"/>
</LinearLayout>
```

```java
public class CustomViewHolder extends RecyclerView.ViewHolder {
    TextView textView;
    public CustomViewHolder(@NonNull View itemView) {
	super(itemView);
	textView = itemView.findViewById(R.id.textview);
    }

    public void setText(String text) {
	textView.setText(text);
    }
}
```

在 `ViewHolder` 中，我们 <br/>

-   为其定义了一个私有变量 `TextView` <br/>
-   定义了一个 `setText` 来设置 `TextView` 的显示 <br/>
-   外部可以使用 `setText` 来设置布局的显示，看 `Adapter` 的代码 <br/>

至此，相关代码已完成编写，可以运行了