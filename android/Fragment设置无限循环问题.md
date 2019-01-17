## Fragment中RecyclerView设置Integer.MAX_VALUE出现stackoverflow

#### 问题描述

Viewpager中使用Fragment，嵌套一个RecyclerView实现一个类似于中奖信息展示的效果,结果发现出现此问题

#### 问题导致原因

fragment布局中使用百分比进行布局

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center_vertical"
    android:orientation="horizontal">

    <ImageView
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:scaleType="fitXY"
        android:src="@drawable/img" />

    <android.support.v7.widget.RecyclerView
        android:id="@+id/mAutoRv"
        android:layout_width="0dp"
       android:layout_weight="1"
        android:layout_height="300dp" />
    
</LinearLayout>
```



#### 解决方案

在RecyclerView外面再套一层FrameLayout或不使用layout_weight

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center_vertical"
    android:orientation="horizontal">

    <ImageView
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:scaleType="fitXY"
        android:src="@drawable/img" />

   <FrameLayout
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1">

        <android.support.v7.widget.RecyclerView
            android:id="@+id/mAutoRv"
            android:layout_width="match_parent"
            android:layout_height="300dp" />
   </FrameLayout>
</LinearLayout>
```

