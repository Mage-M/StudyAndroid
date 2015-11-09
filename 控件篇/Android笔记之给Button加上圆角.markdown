# Android笔记之给Button加上圆角

标签（空格分隔）： Android

---

在布局文件夹资源下，写一个XML文件：
```
//round_corner_bg.xml

<?xml version="1.0" encoding="UTF-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle" >
    <!-- 填充的颜色 -->
    <solid android:color="#ff000000" />
    <!-- 设置矩形的四个角为弧形 -->
    <!-- android:radius 弧形的半径 -->
    <corners android:radius="7dip" />
</shape>
```

使用Button时在其布局文件中加上一个属性如下：
layout中给button加上background属性
android:background="@layout/round_corner_bg"


其他需要加上圆角的控件也可以采取类似的方法。


