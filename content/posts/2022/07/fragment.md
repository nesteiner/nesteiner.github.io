+++
title = "Android Fragment 简单使用"
date = 2022-07-02T14:36:00+08:00
lastmod = 2022-07-02T14:46:34+08:00
tags = ["Fragment"]
categories = ["Android"]
draft = false
toc = true
+++

## Fragment {#fragment}

这篇笔记大部分参考自 [这篇教程](https://www.jianshu.com/p/2bf21cefb763) <br/>


### 什么是 Fragment {#什么是-fragment}

`Fragment` 是 `Activity` 界面中的一部分，可理解为模块化的 `Activity` <br/>

1.  `Fragment` 不能独立存在，必须嵌入到 `Activity` 中 <br/>
2.  `Fragment` 具有自己的生命周期，接收它自己的事件，并可以在 `Activity` 运行时被添加或删除 <br/>
3.  `Fragment` 的生命周期直接受所在的Activity的影响。如：当 `Activity` 暂停时，它拥有的所有 `Fragment` 们都暂停 <br/>


### Fragment 有什么用 {#fragment-有什么用}

支持动态，灵活的界面设计 <br/>


### 如何使用 Fragment {#如何使用-fragment}


#### 相关回调 {#相关回调}

1.  `onAttach` 方法 <br/>
    Fragment和Activity建立关联的时候调用（获得activity的传递的值） <br/>
2.  `onCreateView` 方法 <br/>
    为Fragment创建视图（加载布局）时调用（给当前的fragment绘制UI布局，可以使用线程更新UI） <br/>
3.  `onActivityCreated` 方法 <br/>
    当Activity中的onCreate方法执行完后调用（表示activity执行oncreate方法完成了的时候会调用此方法） <br/>
4.  `onDestroyView` 方法 <br/>
    Fragment中的布局被移除时调用（表示fragment销毁相关联的UI布局） <br/>
5.  `onDetach` 方法 <br/>
    Fragment和Activity解除关联的时候调用（脱离activity） <br/>


#### 静态添加 Fragment {#静态添加-fragment}

首先需要创建一个 `fragment` 类，并为其创建布局文件 `example_fragment.xml` ，再将其配置到 `activity_main.xml` 中 <br/>

1.  `example_fragment.xml` 布局文件 <br/>
    ```xml
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical" >
    
        <TextView
    	android:text="@string/example_fragment"
    	android:layout_width="wrap_content"
    	android:layout_height="wrap_content"/>
    
    </LinearLayout>
    ```

2.  `FragmentLayoutTest.java` 类文件 <br/>
    ```java
    public class FragmentLayoutTest extends Fragment {
        @Override
        public View onCreateView(LayoutInflater inflater, ViewGroup container,
    			     Bundle savedInstanceState) {
    
    	return inflater.inflate(R.layout.example_fragment, container, false);
    	// 将example_fragment.xml作为该Fragment的布局文件
    	// 即相当于FragmentLayoutTest直接调用example_fragment.xml来显示到Fragment中
        }
    }
    ```
3.  在 `activity_main` 布局文件中, <br/>
    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    	      android:layout_width="match_parent"
    	      android:layout_height="match_parent"
    	      android:baselineAligned="false">
      <fragment android:name="com.example.criminalintent.FragmentLayoutTest"
    	    android:id="@+id/list"
    	    android:layout_weight="1"
    	    android:layout_width="match_parent"
    	    android:layout_height="match_parent"/>
    </LinearLayout>
    ```


#### 动态添加 Fragment {#动态添加-fragment}

或者，我们在运行时为主界面添加 `fragment` ，这时我们不用在 `activity_main.xml` 布局中声明 `fragment` <br/>
而是使用三个步骤添加 <br/>

1.  获取 `FragmentManager` <br/>
2.  获取 `FragmentTransaction` <br/>
3.  创建 `Fragment` <br/>
4.  添加 `Fragment` <br/>

不过我们先调整下 `activity_main.xml` 布局 <br/>

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

    <FrameLayout
	android:id="@+id/fragment_container"
	android:layout_width="match_parent"
	android:layout_height="match_parent"/>

</LinearLayout>
```

在主界面的类文件中 <br/>

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.fragment_transaction_test);
    // 步骤1：获取FragmentManager
    FragmentManager fragmentManager = getFragmentManager();
    // 步骤2：获取FragmentTransaction        
    FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();

    FragmentLayoutTest fragment = new FragmentLayoutTest();
    fragmentTransaction.add(R.id.fragment_container, fragment);
    fragmentTransaction.commit();
}
```