# Android笔记之SeekBar

标签（空格分隔）： SeekBar

---

听歌、看电影，快进或者退回某一时间或者调节音量大小，我们都会用到SeekBar拖动条。

下面是一个SeekBar的Demo：

---
activity_main.xml布局文件如下:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <SeekBar 
        android:id="@+id/seekbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:max="100"
        android:progress="50"
        android:secondaryProgress="75"/>
    
    <TextView 
        android:id="@+id/info1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
    
    <TextView 
        android:id="@+id/info2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

---

MainActivity.java文件如下:

```java
package com.example.seekbartest;

import com.example.helloworld.R;

import android.app.Activity;
import android.os.Bundle;
import android.widget.SeekBar;
import android.widget.SeekBar.OnSeekBarChangeListener;
import android.widget.TextView;


public class MainActivity extends Activity implements OnSeekBarChangeListener{
	//要实现OnSeekBarChangeListener接口，并重写里面的三个方法
	private SeekBar seekBar = null;
	private TextView info1 = null;
	private TextView info2 = null;
	
	
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        seekBar = (SeekBar)findViewById(R.id.seekbar);
        info1 = (TextView)findViewById(R.id.info1);
        info2 = (TextView)findViewById(R.id.info2);
        
        seekBar.setOnSeekBarChangeListener(this);
    }
    //进度条发生变化时调用
	@Override
	public void onProgressChanged(SeekBar seekBar, int progress,
			boolean fromUser) {
		// TODO Auto-generated method stub
		info1.setText("音量当前值：" + progress);
	}
    
    //用户开始拖动手势时调用
	@Override
	public void onStartTrackingTouch(SeekBar arg0) {
		// TODO Auto-generated method stub
		info2.setText("音量正在调解"); 
	}
    //用户停止拖动手势时调用
	@Override
	public void onStopTrackingTouch(SeekBar arg0) {
		// TODO Auto-generated method stub
		info2.setText("音量停止调解");
	}
}
```
---
####效果如下：
---
![效果图](https://github.com/Mage-M/StudyAndroid/raw/master/控件篇/图片/SeekBar.png)




