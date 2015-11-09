# Android笔记之给Button加上圆角 and 设置按下效果

标签（空格分隔）： Android

---
##圆角
在drawable文件夹资源下，选择shape，写一个XML文件：
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

##按下效果
在drawable文件夹资源下，选择selector，写一个XML文件：
```
<?xml version="1.0" encoding="UTF-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true" android:drawable="@drawable/btn_pressed"/>
    <item android:state_pressed="false" android:drawable="@drawable/btn_bg"/>
</selector>
```
对应的在你的drawable文件夹下要有btn_pressed和btn_bg两张图片。

最后在布局文件中对button的background属性设置为此xml文件即可。