+++
title = "Android 样式与主题"
date = 2022-07-02T14:37:00+08:00
lastmod = 2022-07-02T14:47:20+08:00
tags = ["样式与主题"]
categories = ["Android"]
draft = false
toc = true
+++

## Android 样式与主题 {#android-样式与主题}


### 设置样式 {#设置样式}

假设我们给一个按钮设置样式，在 `res/values/style.xml` 中 <br/>

```xml
<style name="BeatBoxButton">
  <item name="android:background">@color/dark_blue</item>
</style>
```

在需要设置的按钮布局中 <br/>

```xml
<Button
    style="@style/BeatBoxButton"/>
```

这样就完成了样式的设置 <br/>


### 继承样式 {#继承样式}

样式支持继承，我们来继承上面的样式，创建一个新的样式 <br/>

```xml
<style name="BeatBoxButton.Strong">
  <item name="android:textStyle">bold</item>
</style>
```

也可以使用 <br/>

```xml
<style name="StrongBeatBoxButton" parent="@style/BeatBoxButton">
  <item name="android:textStyle">bold</item>
</style>
```


### 使用主题 {#使用主题}

主题也是一种样式，他必须继承自 `Theme` 下的主题，这里定义一个 `style` <br/>

```xml
<style name="BoxButtonTheme" parent="Theme.MaterialComponents.DayNight.DarkActionBar">
  <item name="android:background">@color/black</item>
  <item name="android:textColor">@color/white</item>
  <item name="android:textStyle">bold</item>
</style>
```

在 `AndroidManifest.xml` 中，在 `activity` 标签添加 <br/>

```xml
<activity android:name=".MainActivity" android:theme="@style/BoxButtonTheme">
```

这样以后，在 `.MainActivity` 下的所有控件都会使用 `BoxButtontheme` 主题中定义的样式 <br/>
同理，在 `application` 标签添加主题，旗下的 `activity` 都会使用该主题 <br/>