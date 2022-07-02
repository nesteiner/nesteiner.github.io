+++
title = "ViewPager 简单使用"
date = 2022-07-02T14:37:00+08:00
lastmod = 2022-07-02T14:47:05+08:00
tags = ["ViewPager"]
categories = ["Android"]
draft = false
toc = true
+++

## ViewPager {#viewpager}

`ViewPager` 简单来说就是一个能滑动切换视图的控件，由一个 `Activity` 包裹，总体结构是 <br/>

-   `Activity` <br/>
    -   **adapter** <br/>
        -   `fragment` <br/>
        -   `fragment` <br/>
        -   `fragment` <br/>

其中 `Activity` 通过 **adapter** 来操作 `fragment` 显示 <br/>


### Activity 布局 {#activity-布局}

`activity_pageview.xml` <br/>

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
		xmlns:tools="http://schemas.android.com/tools"
		android:layout_width="match_parent"
		android:layout_height="match_parent" >

  <androidx.viewpager.widget.ViewPager
      android:id="@+id/viewpager"
      android:layout_width="fill_parent"
      android:layout_height="fill_parent" />

</RelativeLayout>

```

`PageViewActivity.java` <br/>

```java
public class PageViewActivity extends FragmentActivity {
    MyPageAdapter pageAdapter;

    @Override
    public void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	setContentView(R.layout.activity_page_view);

	List<Fragment> fragments = getFragments();

	pageAdapter = new MyPageAdapter(getSupportFragmentManager(), fragments);

	ViewPager pager = (ViewPager) findViewById(R.id.viewpager);
	pager.setAdapter(pageAdapter);

    }

    private List<Fragment> getFragments() {
	List<Fragment> fList = new ArrayList<Fragment>();

	fList.add(MyFragment.newInstance("Fragment 1"));
	fList.add(MyFragment.newInstance("Fragment 2"));
	fList.add(MyFragment.newInstance("Fragment 3"));

	return fList;
    }
}
```


### Adapter 介绍 {#adapter-介绍}

`Adapter` 虽然有三种，但这里只需要介绍 `FragmentPagerAdapter` 与 `FragmentStatePagerAdapter` 即可，他们两个需要重写的方法是一致的 <br/>


#### FragmentPagerAdapter {#fragmentpageradapter}

-   `FragmentPagerAdapter` 是 `PagerAdapter` 中的其中一种实现 <br/>
-   它将每一个页面表示为一个 `Fragment` ，并且每一个 `Fragment` 都将会保存到 `fragment manager` 当中 <br/>
-   这种pager十分适用于有一些静态 `fragment` ，例如一组 `tabs` 的时候使用 <br/>
-   每个页面对应的 `Fragment` 当用户可以访问的时候会一直存在内存中，但是，当这个页面不可见的时候，view hierarchy将会被销毁 <br/>
-   这样子会导致应用程序占有太多资源。当页面数量比较大的时候，建议使用 `FragmentStatePagerAdapter` <br/>


#### FragmentStatePagerAdapter {#fragmentstatepageradapter}

-   当使用FragmentStatePagerAdapter 时，实现将只保留当前页面，当页面离开视线后，就会被消除，释放其资源 <br/>
-   而在页面需要显示时，生成新的页面 <br/>
-   这么实现的好处就是当拥有大量的页面时，不必在内存中占用大量的内存 <br/>


#### 实例 {#实例}

`MyPageAdapter.java` <br/>

```java
public class MyPageAdapter extends FragmentStatePagerAdapter {
    private List<Fragment> fragments;

    public MyPageAdapter(FragmentManager fm, List<Fragment> fragments) {
	super(fm);
	this.fragments = fragments;
    }
    @Override
    public Fragment getItem(int position) {
	return this.fragments.get(position);
    }

    @Override
    public int getCount() {
	return this.fragments.size();
    }
}
```


### Fragment 布局 {#fragment-布局}

`fragment_layout.xml` <br/>

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
	  android:id="@+id/tv_bg"
	  android:layout_width="match_parent"
	  android:layout_height="match_parent"
	  android:gravity="center_horizontal"
	  android:paddingTop="100dp">

</TextView>
```

`MyFragment.java` <br/>

```java
public class MyFragment extends Fragment {
    public static final String EXTRA_MESSAGE = "EXTRA_MESSAGE";

    public static final MyFragment newInstance(String message)
    {
	MyFragment f = new MyFragment();
	Bundle bdl = new Bundle(1);
	bdl.putString(EXTRA_MESSAGE, message);
	f.setArguments(bdl);
	return f;
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
			     Bundle savedInstanceState) {
	String message = getArguments().getString(EXTRA_MESSAGE);
	View v = inflater.inflate(R.layout.myfragment_layout, container, false);
	TextView messageTextView = (TextView)v.findViewById(R.id.textView);
	messageTextView.setText(message);

	return v;
    }

}

```