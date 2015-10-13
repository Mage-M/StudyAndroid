# Android笔记之OnCheckedChangeListener监听器

标签（空格分隔）： Android

---

单选按钮RadioGroup、复选框CheckBox都有OnCheckedChangeListener事件。

下面是一个简单的demo使用这个监听器：

---
###布局文件如下：
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" >

    <RadioGroup
        android:id="@+id/gender"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentLeft="true"
        android:layout_alignParentTop="true" >
        <RadioButton
            android:id="@+id/male"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:checked="true"
            android:text="男" />
        <RadioButton
            android:id="@+id/female"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="女" />
    </RadioGroup>

    <CheckBox
        android:id="@+id/football"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentLeft="true"
        android:layout_below="@+id/gender"
        android:text="足球" />
    <CheckBox
        android:id="@+id/basketball"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentLeft="true"
        android:layout_below="@+id/football"
        android:text="蓝球" />

</RelativeLayout>
```
---

###MainActivity代码如下：
```java
public class MainActivity extends Activity {
	private RadioGroup GenderGroup=null;
	private RadioButton rbMale=null;
	private RadioButton rbFemale=null;
	private CheckBox cbFootBall=null;
	private CheckBox cbBasketBall=null;
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		
		GenderGroup=(RadioGroup)super.findViewById(R.id.gender);
		rbMale=(RadioButton)super.findViewById(R.id.male);
		rbFemale=(RadioButton)super.findViewById(R.id.female);
		cbFootBall=(CheckBox)super.findViewById(R.id.football);
		cbBasketBall=(CheckBox)super.findViewById(R.id.basketball);
		//在GenderGroup注册OnCheckedChangeListener事件
                  GenderGroup.setOnCheckedChangeListener(new GenderOnCheckedChangeListener());
                  //在cbFootBall注册OnCheckedChangeListener事件
		cbFootBall.setOnCheckedChangeListener(new BootBallOnCheckedChangeListener());
                  //在cbBasketBall注册OnCheckedChangeListener事件

		cbBasketBall.setOnCheckedChangeListener(new BasketBallOnCheckedChangeListener());
	}
	
	private class GenderOnCheckedChangeListener implements OnCheckedChangeListener{
		@Override
		public void onCheckedChanged(RadioGroup group,int checkedId){
			String sGender="";
			if(rbFemale.getId()==checkedId){
				sGender=rbFemale.getText().toString();
			}
			if(rbMale.getId()==checkedId){
				sGender=rbMale.getText().toString();
			}
			Toast.makeText(getApplicationContext(), "您选择的性别是："+sGender, Toast.LENGTH_LONG).show();
		}
		
	}
	
	private class BootBallOnCheckedChangeListener implements CompoundButton.OnCheckedChangeListener{
		@Override
		public void onCheckedChanged(CompoundButton button, boolean isChecked){
			String sFav="";
			if(isChecked){
				sFav=cbFootBall.getText().toString();
				sFav=sFav+"选中！";
			}
			else
				sFav=sFav+"未选中";
			Toast.makeText(getApplicationContext(), "您选择的爱好是："+sFav, Toast.LENGTH_LONG).show();
		}
	}
	
	private class BasketBallOnCheckedChangeListener implements CompoundButton.OnCheckedChangeListener{
		@Override
		public void onCheckedChanged(CompoundButton button,boolean isChecked){
			String sFav="";
			if(cbBasketBall.isChecked()){
				sFav=cbBasketBall.getText().toString();
				sFav=sFav+"选中！";
			}
			else
				sFav=sFav+"未选中";
			Toast.makeText(getApplicationContext(), "您选择的爱好是："+sFav, Toast.LENGTH_LONG).show();
		}
	}
	
}
```

---

###效果如下：

![效果图](https://github.com/Mage-M/StudyAndroid/raw/master/杂篇/图片/OnCheckedChangeListener1.png)

![效果图](https://github.com/Mage-M/StudyAndroid/raw/master/杂篇/图片/OnCheckedChangeListener2.png)
