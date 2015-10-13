# Android笔记之OnFocusChangeListener监听器

标签（空格分隔）： Android

---
OnFocusChangeListener监听器的一个Demo：
---

###布局文件如下：
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

    <EditText
        android:id="@+id/name"
        android:layout_width="190dp"
        android:layout_height="wrap_content"
        android:text="您的姓名" />

    <EditText
        android:id="@+id/password"
        android:layout_width="190dp"
        android:layout_height="wrap_content"
        android:text="您的密码" />

</LinearLayout>
```


---

###MainActivity代码如下：
```java
public class MainActivity extends Activity {
	//声明EditText
	private EditText  name=null;
	private EditText  password=null;
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		//获取EditText
		name=(EditText)super.findViewById(R.id.name);
		password=(EditText)super.findViewById(R.id.password);
		//注册OnClick、OnFocusChange监听器
		name.setOnClickListener(new NameOnClickListener());
		name.setOnFocusChangeListener(new NameOnFocusChanageListener());
		password.setOnClickListener(new PasswordOnClickListener());
		password.setOnFocusChangeListener(new PasswordOnFocusChanageListener());
	}
	//MobileOnClickListener单击监听器
	private class NameOnClickListener implements OnClickListener{
		@Override
		public void onClick(View view){
			name.setText("");
		}
	}
	//MobileOnFocusChanageListener焦点监听器
	private class NameOnFocusChanageListener implements OnFocusChangeListener{
		@Override
		public void onFocusChange(View view,boolean hasFocus){
			if(view.getId()==name.getId())
			    Toast.makeText(getApplicationContext(), "姓名获得焦点！",Toast.LENGTH_LONG).show();
		}
	}
	//MobileOnClickListener单击监听器
	private class PasswordOnClickListener implements OnClickListener{
		@Override
		public void onClick(View view){
			password.setText("");
		}		
	}
	//MobileOnFocusChanageListener焦点监听器	
	private class PasswordOnFocusChanageListener implements OnFocusChangeListener{
		@Override
		public void onFocusChange(View view,boolean hasFocus){
			if(view.getId()==password.getId())
				Toast.makeText(getApplicationContext(), "密码获得焦点！",Toast.LENGTH_LONG).show();
		}
	}
}
```

###效果如下：
![效果图](https://github.com/Mage-M/StudyAndroid/raw/master/杂篇/图片/OnFocusChangeListener.png)


