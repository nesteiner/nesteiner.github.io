+++
title = "Android ToolBar 简单使用"
date = 2022-07-02T14:38:00+08:00
lastmod = 2022-07-02T14:47:50+08:00
tags = ["Toolbar"]
categories = ["Android"]
draft = false
toc = true
+++

## Android ToolBar 简单使用 {#android-toolbar-简单使用}


### 介绍 {#介绍}

![](https://upload-images.jianshu.io/upload_images/912181-0f2cde151fdc03db.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/391/format/webp) <br/>
按照效果图，从左到右分别是 <br/>

-   导航栏图标 <br/>
-   App Logo <br/>
-   标题 <br/>
-   子标题 <br/>
-   自定义控件 (Clock) <br/>
-   ActionMenu <br/>

接下来我们制作这个页面 <br/>


### activity 布局文件 {#activity-布局文件}

在 `layout/activity_toolbar.xml` 中 <br/>

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	      android:layout_width="match_parent"
	      android:layout_height="match_parent"
	      android:orientation="vertical">

  <android.support.v7.widget.Toolbar
      android:id="@+id/toolbar"
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:background="@color/color_0176da">

    <!--自定义控件-->
    <TextView
	android:layout_width="wrap_content"
	android:layout_height="wrap_content"
	android:text="Clock" />
  </android.support.v7.widget.Toolbar>
</LinearLayout>
```


### menu 菜单文件 {#menu-菜单文件}

在 `menu/toolbar_menu.xml` 中，这里记得新建 `menu` 资源文件夹 <br/>

```xml

<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto">

  <item
      android:id="@id/action_search"
      android:icon="@mipmap/ic_search"
      android:title="@string/menu_search"
      app:showAsAction="ifRoom" />

  <item
      android:id="@id/action_notification"
      android:icon="@mipmap/ic_notifications"
      android:title="@string/menu_notifications"
      app:showAsAction="ifRoom" />

  <item
      android:id="@+id/action_item1"
      android:title="@string/item_01"
      app:showAsAction="never" />

  <item
      android:id="@+id/action_item2"
      android:title="@string/item_02"
      app:showAsAction="never" />
</menu>
```

这里我们通过设置 `showAsAction` 来确定 `menu item` 的位置 <br/>

-   `ifRoom` 如果工具栏由剩余空间，那么显示在工具栏中 <br/>
-   `never` 显示在菜单弹出框中 <br/>


### 设置 activity 的 ToolbarActivity {#设置-activity-的-toolbaractivity}

在 `ToolbarActivity.java` 中 <br/>

```java

/**
 * Toolbar的基本使用
 */
public class ToolBarActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	setContentView(R.layout.activity_toolbar);

	Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);

	toolbar.setNavigationIcon(R.mipmap.ic_drawer_home);//设置导航栏图标
	toolbar.setLogo(R.mipmap.ic_launcher);//设置app logo
	toolbar.setTitle("Title");//设置主标题
	toolbar.setSubtitle("Subtitle");//设置子标题

	toolbar.inflateMenu(R.menu.toolbar_menu);//设置右上角的填充菜单
	toolbar.setOnMenuItemClickListener(new Toolbar.OnMenuItemClickListener() {
		@Override
		public boolean onMenuItemClick(MenuItem item) {
		    int menuItemId = item.getItemId();
		    if (menuItemId == R.id.action_search) {
			Toast.makeText(ToolBarActivity.this , R.string.menu_search , Toast.LENGTH_SHORT).show();

		    } else if (menuItemId == R.id.action_notification) {
			Toast.makeText(ToolBarActivity.this , R.string.menu_notifications , Toast.LENGTH_SHORT).show();

		    } else if (menuItemId == R.id.action_item1) {
			Toast.makeText(ToolBarActivity.this , R.string.item_01 , Toast.LENGTH_SHORT).show();

		    } else if (menuItemId == R.id.action_item2) {
			Toast.makeText(ToolBarActivity.this , R.string.item_02 , Toast.LENGTH_SHORT).show();

		    }
		    return true;
		}
	    });

    }

}

```

在这里，我们 <br/>

1.  设置 `ToolBar` 的属性 <br/>
    -   `setNavigationicon` <br/>
    -   `setLogo` <br/>
    -   `setTitle` <br/>
    -   `setSubtitle` <br/>
2.  渲染菜单选项 <br/>
    -   `inflateMenu` <br/>
3.  设置点击事件 <br/>
    -   `setOnMenuitemClickListener` <br/>


### 补充 {#补充}


#### 在 xml 布局中设置 toolbar 的属性 {#在-xml-布局中设置-toolbar-的属性}

这里需要自定义一个命名空间 `toolbar` ，而不是使用 `android` <br/>

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	      xmlns:toolbar="http://schemas.android.com/apk/res-auto"
	      android:layout_width="match_parent"
	      android:layout_height="match_parent"
	      android:orientation="vertical">

  <android.support.v7.widget.Toolbar
      android:id="@+id/toolbar"
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:background="@color/color_0176da"
      toolbar:navigationIcon="@mipmap/ic_drawer_home"
      toolbar:logo="@mipmap/ic_launcher"
      toolbar:subtitle="456"
      toolbar:title="123">

    <!--自定义控件-->
    <TextView
	android:layout_width="wrap_content"
	android:layout_height="wrap_content"
	android:text="Clock" />
  </android.support.v7.widget.Toolbar>
</LinearLayout>
```