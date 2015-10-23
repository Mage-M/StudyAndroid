# Android笔记之View

标签（空格分隔）： Android

---

[TOC]

##一.LayoutInflater
加载布局的任务通常都是在Activity中调用setContentView()方法来完成的。其实setContentView()方法的内部也是使用LayoutInflater来加载布局的。

LayoutInflater实例获取：
第一种：
```
LayoutInflater layoutInflater = LayoutInflater.from(context);  
```

第二种：
```
LayoutInflater layoutInflater = (LayoutInflater) context
		.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
```

获取实例后我们可以调用其inflate方法动态加载布局文件。
```
layoutInflater.inflate(resourceId, root);  
```
第一个参数表示我们要加载的资源的布局id，第二个参数表示给该布局外面再加一层父布局，不用就直接传入null
在现实使用中大部分情况都只用传入null，不用再加上一层父布局。

以下例子来自郭神。
我们新建一个项目，里面有一个activity_main.xml的布局文件:
```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/main_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

</LinearLayout>
```
只有一个LinearLayout布局，其他的什么都没有。

现在我们新建一个button_layout.xml布局文件如下：
```
<Button xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Button" >

</Button>
```
只有一个button按钮，现在我们需要用LayoutInflater动态将这个button加到刚才的主布局文件中。
MainActivity.java代码如下：
```
public class MainActivity extends Activity {

	private LinearLayout mainLayout;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		mainLayout = (LinearLayout) findViewById(R.id.main_layout);
		LayoutInflater layoutInflater = LayoutInflater.from(this);
		View buttonLayout = layoutInflater.inflate(R.layout.button_layout, null);
		mainLayout.addView(buttonLayout);
	}

}
```
这样我们就可以动态的将这个button加载到主布局中了。

***LayoutInflater技术广泛应用于需要动态添加View的时候，比如在ScrollView和ListView中，经常都可以看到LayoutInflater的身影。***

PS：inflate()方法还有个接收三个参数的方法重载，结构如下：
```
inflate(int resource, ViewGroup root, boolean attachToRoot)  
```
第三个参数attachToRoot意义为下：
 1. 如果root为null，attachToRoot将失去作用，设置任何值都没有意义。
 2. 如果root不为null，attachToRoot设为true，则会给加载的布局文件的指定一个父布局，即root。
 3. 如果root不为null，attachToRoot设为false，则会将布局文件最外层的所有layout属性进行设置，当该view被添加到父view当中时，这些layout属性会自动生效。
 4. 在不设置attachToRoot参数的情况下，如果root不为null，attachToRoot参数默认为true。

当我们想要修改刚才Button布局文件的大小时，如下：
```
<Button xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="300dp"
    android:layout_height="80dp"
    android:text="Button" >

</Button>
```
发现重新运行后button并没有大小的变化。这里引出另一个点：
其实这里不管你将Button的layout_width和layout_height的值修改成多少，都不会有任何效果的，因为这两个值现在已经完全失去了作用。平时我们经常使用layout_width和layout_height来设置View的大小，并且一直都能正常工作，就好像这两个属性确实是用于设置View的大小的。而实际上则不然，它们其实是用于设置View在布局中的大小的，也就是说，首先View必须存在于一个布局中，之后如果将layout_width设置成match_parent表示让View的宽度填充满布局，如果设置成wrap_content表示让View的宽度刚好可以包含其内容，如果设置成具体的数值则View的宽度会变成相应的数值。这也是为什么这两个属性叫作layout_width和layout_height，而不是width和height。

想让Button大小变化最简单就是在这个Button外面再套入一层布局：
```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <Button
        android:layout_width="300dp"
        android:layout_height="80dp"
        android:text="Button" >
    </Button>

</RelativeLayout>
```
这个时候因为外面有了一层布局，所以Button的layout_width和layout_height属性又可以生效，但是最外层的RelativeLayout的这两个属性会失效。重新运行程序，按钮的大小确实变化了。

PS：可是平时在Activity中指定布局文件的时候，最外层的那个布局是可以指定大小的呀，layout_width和layout_height都是有作用的。确实，这主要是因为，在setContentView()方法中，Android会自动在布局文件的最外层再嵌套一个FrameLayout，所以layout_width和layout_height属性才会有效果。用如下例子验证：
```
public class MainActivity extends Activity {

	private LinearLayout mainLayout;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		mainLayout = (LinearLayout) findViewById(R.id.main_layout);
		ViewParent viewParent = mainLayout.getParent();
		Log.d("TAG", "the parent of mainLayout is " + viewParent);
	}

}
```
运行程序我们可以发现log中打印出了它的父布局确实是一个FrameLayout，而这个FrameLayout就是由系统自动帮我们添加上的。

说到这里，虽然setContentView()方法大家都会用，但实际上Android界面显示的原理要比我们所看到的东西复杂得多。任何一个Activity中显示的界面其实主要都由两部分组成，标题栏和内容布局。标题栏就是在很多界面顶部显示的那部分内容，比如刚刚我们的那个例子当中就有标题栏，可以在代码中控制让它是否显示。而内容布局就是一个FrameLayout，这个布局的id叫作content，我们调用setContentView()方法时所传入的布局其实就是放到这个FrameLayout中的，这也是为什么这个方法名叫作setContentView()，而不是叫setView()

##二.自定义View：
###1.自绘控件： 
自绘控件的意思就是这个View上所有的内容都是我们自己写，我们写什么，上面就显示什么，绘制的代码是写在onDraw（）方法中的。
下面例子同上来自郭神博客：我们准备一个自定义的计数器View，这个View可以响应用户的点击事件，并自动记录点击了多少次。
新建一个类CounterView继承自View，代码如下：
```
public class CounterView extends View implements OnClickListener {

	private Paint mPaint;
	
	private Rect mBounds;

	private int mCount;
	
	public CounterView(Context context, AttributeSet attrs) {
		super(context, attrs);
		mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
		mBounds = new Rect();
		setOnClickListener(this);
	}

	@Override
	protected void onDraw(Canvas canvas) {
		super.onDraw(canvas);
		mPaint.setColor(Color.BLUE);
		canvas.drawRect(0, 0, getWidth(), getHeight(), mPaint);
		mPaint.setColor(Color.YELLOW);
		mPaint.setTextSize(30);
		String text = String.valueOf(mCount);
		mPaint.getTextBounds(text, 0, text.length(), mBounds);
		float textWidth = mBounds.width();
		float textHeight = mBounds.height();
		canvas.drawText(text, getWidth() / 2 - textWidth / 2, getHeight() / 2
				+ textHeight / 2, mPaint);
	}

	@Override
	public void onClick(View v) {
		mCount++;
		invalidate();
	}

}
```

可以看到，首先我们在CounterView的构造函数中初始化了一些数据，并给这个View的本身注册了点击事件，这样当CounterView被点击的时候，onClick()方法就会得到调用。而onClick()方法中的逻辑就更加简单了，只是对mCount这个计数器加1，然后调用invalidate()方法。通过 Android视图状态及重绘流程分析，调用invalidate()方法会导致视图进行重绘，因此onDraw()方法在稍后就将会得到调用.

这样，一个自定义的View就已经完成了，并且目前这个CounterView是具备自动计数功能的。那么剩下的问题就是如何让这个View在界面上显示出来了，其实这也非常简单，我们只需要像使用普通的控件一样来使用CounterView就可以了。比如在布局文件中加入如下代码：
```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <com.example.customview.CounterView
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_centerInParent="true" />

</RelativeLayout>
```

可以看到，这里我们将CounterView放入了一个RelativeLayout中，然后可以像使用普通控件来给CounterView指定各种属性，比如通过layout_width和layout_height来指定CounterView的宽高，通过android:layout_centerInParent来指定它在布局里居中显示。只不过需要注意，自定义的View在使用的时候一定要写出完整的包名，不然系统将无法找到这个View。
好了，就是这么简单，接下来我们可以运行一下程序，并不停地点击CounterView，效果如下图所示:

![效果图](https://github.com/Mage-M/StudyAndroid/raw/master/控件篇/图片/counterview.gif)


###2.组合控件：
组合控件的意思就是，我们并不需要自己去绘制视图上显示的内容，而只是用系统原生的控件就好了，但我们可以将几个系统原生的控件组合到一起，这样创建出的控件就被称为组合控件。

举个例子来说，标题栏就是个很常见的组合控件，很多界面的头部都会放置一个标题栏，标题栏上会有个返回按钮和标题，点击按钮后就可以返回到上一个界面。那么下面我们就来尝试去实现这样一个标题栏控件。

新建一个title.xml布局文件，如下：
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="50dp"
    android:background="#ffcb05" >

    <Button
        android:id="@+id/button_left"
        android:layout_width="60dp"
        android:layout_height="40dp"
        android:layout_centerVertical="true"
        android:layout_marginLeft="5dp"
        android:background="@drawable/back_button"
        android:text="Back"
        android:textColor="#fff" />

    <TextView
        android:id="@+id/title_text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:text="This is Title"
        android:textColor="#fff"
        android:textSize="20sp" />

</RelativeLayout>
```

接下来创建一个TitleView继承自FrameLayout，代码如下所示：
```
public class TitleView extends FrameLayout {

	private Button leftButton;

	private TextView titleText;

	public TitleView(Context context, AttributeSet attrs) {
		super(context, attrs);
		LayoutInflater.from(context).inflate(R.layout.title, this);
		titleText = (TextView) findViewById(R.id.title_text);
		leftButton = (Button) findViewById(R.id.button_left);
		leftButton.setOnClickListener(new OnClickListener() {
			@Override
			public void onClick(View v) {
				((Activity) getContext()).finish();
			}
		});
	}

	public void setTitleText(String text) {
		titleText.setText(text);
	}

	public void setLeftButtonText(String text) {
		leftButton.setText(text);
	}

	public void setLeftButtonListener(OnClickListener l) {
		leftButton.setOnClickListener(l);
	}

}
```

TitleView中的代码非常简单，在TitleView的构建方法中，我们调用了LayoutInflater的inflate()方法来加载刚刚定义的title.xml布局,接下来调用findViewById()方法获取到了返回按钮的实例，然后在它的onClick事件中调用finish()方法来关闭当前的Activity，也就相当于实现返回功能了。

另外，为了让TitleView有更强地扩展性，我们还提供了setTitleText()、setLeftButtonText()、setLeftButtonListener()等方法，分别用于设置标题栏上的文字、返回按钮上的文字、以及返回按钮的点击事件。

到了这里，一个自定义的标题栏就完成了，那么下面又到了如何引用这个自定义View的部分，其实方法基本都是相同的，在布局文件中添加如下代码：
```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <com.example.customview.TitleView
        android:id="@+id/title_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" >
    </com.example.customview.TitleView>

</RelativeLayout>
```
效果如下所示：

![效果图](https://github.com/Mage-M/StudyAndroid/raw/master/控件篇/图片/titleview.png)

现在点击一下Back按钮，就可以关闭当前的Activity了。如果你想要修改标题栏上显示的内容，或者返回按钮的默认事件，只需要在Activity中通过findViewById()方法得到TitleView的实例，然后调用setTitleText()、setLeftButtonText()、setLeftButtonListener()等方法进行设置就OK了。

###3.继承控件：
继承控件的意思就是，我们并不需要自己重头去实现一个控件，只需要去继承一个现有的控件，然后在这个控件上增加一些新的功能，就可以形成一个自定义的控件了。这种自定义控件的特点就是不仅能够按照我们的需求加入相应的功能，还可以保留原生控件的所有功能，比如AndroidPowerImageView实现，可以播放动画的强大ImageView 这篇文章中介绍的PowerImageView就是一个典型的继承控件。

在这我就不再给出例子了。



**关于View的应用上面这些先消化，至于原理性的东西，将来有时间一定回来搞懂。**
