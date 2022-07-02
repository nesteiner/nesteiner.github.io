+++
title = "Android 数据绑定简单使用"
date = 2022-07-02T14:39:00+08:00
lastmod = 2022-07-02T14:48:21+08:00
tags = ["数据绑定"]
categories = ["Android"]
draft = false
toc = true
+++

## Android 数据绑定简单使用 {#android-数据绑定简单使用}

这次我们要替代获取控件的方法，以前是使用 `findViewById` 一个一个拿到控件 <br/>
现在我们需要直接在 `layout` 中注入数据对象，我们还要直接通过 `id` 访问到控件 <br/>


### 数据绑定 {#数据绑定}


#### 启用数据绑定 {#启用数据绑定}

在 `build.gradle` 中添加如下代码 <br/>

```groovy
android {
    dataBinding {
	enable = true
    }
}
```

即可开启数据绑定 <br/>


#### 修改布局文件 {#修改布局文件}

先看 **Vue** 中的数据绑定是怎样操作的 <br/>

```vue
<template>
  <div class="container">
    <span> {{data}} </span>
  </div>
</template>

<script lang="ts" setup>
  const data = 'hello'
</script>
```

在 **XML** 中 <br/>

-   `layout` 类似 `template` <br/>
-   使用 `data` 标签声明数据 <br/>

假设有数据类型 `User` <br/>

```java
public class User {
    public String name;
    public String password;

    public User(String name, String password) {
	this.name = name;
	this.password = password;
    }
}
```

先在 `activity_main` 中声明数据 <br/>

```xml
<data>
  <varibale name="userInfo" type="com.example.databinding.User"/>
</data>
```

再在控件中注入数据 <br/>

```xml
<LinearLayout
     android:layout_width="match_parent"
     android:layout_height="match_parent"
     android:layout_margin="20dp"
     android:orientation="vertical">

    <TextView android:id="@+id/tv_userName"
	      android:text="@{userInfo.name}" />

    <TextView android:text="@{userInfo.password}" />

</LinearLayout>
```

完整代码 <br/>

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
	<import type="com.leavesc.databinding_demo.model.User" />
	<variable
	    name="userInfo"
	    type="User" />
    </data>

    <LinearLayout
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:layout_margin="20dp"
	android:orientation="vertical"
	tools:context="com.leavesc.databinding_demo.Main2Activity">

	<TextView
	    android:id="@+id/tv_userName"
	    ···
	    android:text="@{userInfo.name}" />

	<TextView
	    ···
	    android:text="@{userInfo.password}" />

    </LinearLayout>

</layout>

```


#### 修改 MainActivity 类 {#修改-mainactivity-类}

在 `Activity` 中 <br/>

-   通过 `DataBindingUtil` 设置布局文件 <br/>
-   数据绑定对象 `ActivityMainBinding` 的实例名根据布局文件名来生成 <br/>
-   数据绑定对象通过 `setXXX` 方法注入数据 <br/>

<!--listend-->

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    ActivityMainBinding activityMainBinding = DataBindingUtil.setContentView(this, R.layout.activity_main);
    User user = new User("leavesC", "123456");
    activityMainBinding.setUserInfo(user);
}

```


### 响应式数据绑定 {#响应式数据绑定}

参考 **Vue** 响应式， <br/>

```vue
<template>
<div class="container">
  <span> {{data}} </span>
</div>
</template>

<script lang="ts" setup>
  const data = ref('hello')
</script>
```

只需要用 `ref` 包裹数据即可 <br/>
而在 Android 中也需要用 `ObservableField` 模板类包装数据 <br/>
这里定义一个 `ObservableGoods` 类型 <br/>

```java
public class ObservableGoods {
    public ObservableField<String> name;

    public ObservableField<Float> price
    public ObservableField<String> details;

    public ObservableGoods(String name, float price, String details) {
	this.name = new ObservableField<>(name);
	this.price = new ObservableField<>(price);
	this.details = new ObservableField<>(details);
    }
}

```

同上注入数据即可 <br/>


### 补充 {#补充}


#### Fragment {#fragment}

假设 `Fragment` 有布局文件 `fragment_collection` ，在 `onCreateView` 中 <br/>

```java
@Nullable
@Override
public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
    FragmentCollectionBinding binding = DataBindingUtil.inflate(inflater, R.layout.fragment_collection, container, false);
    return binding.getRoot();
}
```


#### 替代 findViewById {#替代-findviewbyid}

假设在布局文件中有 `id` 为 `tvUserName` 的控件，获取控件可以 <br/>

```java
activityMainBinding.tvUserName.setText("Hello");
```


#### Observable 容器类 {#observable-容器类}

`DataBinding` 也提供了包装类用于替代原生的 List 和 Map，分别是 ObservableList 和 ObservableMap <br/>
当其包含的数据发生变化时，绑定的视图也会随之进行刷新 <br/>
不过在布局文件声明中指定模板类型的时候需要转义符 <br/>

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
	<import type="android.databinding.ObservableList"/>
	<import type="android.databinding.ObservableMap"/>
	<variable
	    name="list"
	    type="ObservableList&lt;String&gt;"/>
	<variable
	    name="map"
	    type="ObservableMap&lt;String,String&gt;"/>
	<variable
	    name="index"
	    type="int"/>
	<variable
	    name="key"
	    type="String"/>
    </data>

<LinearLayout
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:orientation="vertical"
	tools:context="com.leavesc.databinding_demo.Main12Activity">

	<TextView
	    android:padding="20dp"
	    android:text="@{list[index],default=xx}"/>

	<TextView
	    android:layout_marginTop="20dp"
	    android:padding="20dp"
	    android:text="@{map[key],default=yy}"/>

    </LinearLayout>
</layout>
```