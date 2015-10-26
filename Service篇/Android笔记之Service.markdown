# Android笔记之Service

标签（空格分隔）： Android

---

##1.基本用法：
一个普通的java类继承自service方法后，重写其中的三个方法即可成为一个服务。

 - onCreate（）；
 - onStartCommand；
 - onDestroy()；
 - onBind；
 
其中如果不需要活动和服务联系过分紧密，就不必重写其中的onBind方法。

***切记在创建一个新的服务以后要在AndroidManifest.xml文件中进行注册。***

 - 当一个服务启动时，依次调用onCreate（）方法和onStartCommand（）方法
 - 而再次启动这个服务时，只会调用onStartCommand（）方法
 - 这是由于onCreate()方法只会在Service第一次被创建的时候调用，如果当前Service已经被创建过了，不管怎样调用startService()方法，onCreate()方法都不会再执行

##2.Service和Activity通信
例子来自郭霖博客

如果想要在Activity中指定Service中执行的方法，即紧密联系起开，我们需要将Activity与Service绑定起来。

服务的基础类代码如下：
```
public class MyService extends Service {

	public static final String TAG = "MyService";

	private MyBinder mBinder = new MyBinder();

	@Override
	public void onCreate() {
		super.onCreate();
		Log.d(TAG, "onCreate() executed");
	}

	@Override
	public int onStartCommand(Intent intent, int flags, int startId) {
		Log.d(TAG, "onStartCommand() executed");
		return super.onStartCommand(intent, flags, startId);
	}

	@Override
	public void onDestroy() {
		super.onDestroy();
		Log.d(TAG, "onDestroy() executed");
	}

	@Override
	public IBinder onBind(Intent intent) {
		return mBinder;
	}

	class MyBinder extends Binder {

		public void startDownload() {
			Log.d("TAG", "startDownload() executed");
			// 执行具体的下载任务
		}

	}

}
```
这里我们新增了一个MyBinder类继承自Binder类，然后在MyBinder中添加了一个startDownload()方法用于在后台执行下载任务，当然这里并不是真正地去下载某个东西，只是做个测试，所以startDownload()方法只是打印了一行日志。
activity_main.xml中的代码，在布局文件中添加用于绑定Service和取消绑定Service的按钮:
```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

    <Button
        android:id="@+id/start_service"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Start Service" />

    <Button
        android:id="@+id/stop_service"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Stop Service" />

    <Button
        android:id="@+id/bind_service"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Bind Service" />
    
    <Button 
        android:id="@+id/unbind_service"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Unbind Service"
        />
    
</LinearLayout>
```
接下来是MainActivity中的代码，让MainActivity和MyService之间建立关联，代码如下所示：

```
 public class MainActivity extends Activity implements OnClickListener {

	private Button startService;

	private Button stopService;

	private Button bindService;

	private Button unbindService;

	private MyService.MyBinder myBinder;

	private ServiceConnection connection = new ServiceConnection() {

		@Override
		public void onServiceDisconnected(ComponentName name) {
		}

		@Override
		public void onServiceConnected(ComponentName name, IBinder service) {
			myBinder = (MyService.MyBinder) service;
			myBinder.startDownload();
		}
	};

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		startService = (Button) findViewById(R.id.start_service);
		stopService = (Button) findViewById(R.id.stop_service);
		bindService = (Button) findViewById(R.id.bind_service);
		unbindService = (Button) findViewById(R.id.unbind_service);
		startService.setOnClickListener(this);
		stopService.setOnClickListener(this);
		bindService.setOnClickListener(this);
		unbindService.setOnClickListener(this);
	}

	@Override
	public void onClick(View v) {
		switch (v.getId()) {
		case R.id.start_service:
			Intent startIntent = new Intent(this, MyService.class);
			startService(startIntent);
			break;
		case R.id.stop_service:
			Intent stopIntent = new Intent(this, MyService.class);
			stopService(stopIntent);
			break;
		case R.id.bind_service:
			Intent bindIntent = new Intent(this, MyService.class);
			bindService(bindIntent, connection, BIND_AUTO_CREATE);
			break;
		case R.id.unbind_service:
			unbindService(connection);
			break;
		default:
			break;
		}
	}

}
```

这里首先创建了一个ServiceConnection的匿名类，在里面重写了onServiceConnected()方法和onServiceDisconnected()方法，这两个方法分别会在Activity与Service建立关联和解除关联的时候调用。
在onServiceConnected()方法中，我们又通过向下转型得到了MyBinder的实例，有了这个实例，Activity和Service之间的关系就变得非常紧密了。现在我们可以在Activity中根据具体的场景来调用MyBinder中的任何public方法，即实现了Activity指挥Service干什么Service就去干什么的功能。
当然，现在Activity和Service其实还没关联起来了呢，这个功能是在BindService按钮的点击事件里完成的。可以看到，这里我们仍然是构建出了一个Intent对象，然后调用bindService()方法将Activity和Service进行绑定。bindService()方法接收三个参数，第一个参数就是刚刚构建出的Intent对象，第二个参数是前面创建出的ServiceConnection的实例，**第三个参数是一个标志位，这里传入BIND_AUTO_CREATE表示在Activity和Service建立关联后自动创建Service，这会使得MyService中的onCreate()方法得到执行，但onStartCommand()方法不会执行。**

*这里注意，点击绑定服务以后，只会调用onCreate（）方法和我们定义的绑定成功时执行的方法startDownload（），并不会执行onStartCommand（）方法。*

然后如何我们想解除Activity和Service之间的关联，调用一下unbindService()方法就可以了，这也是Unbind Service按钮的点击事件里实现的逻辑。

##3.销毁Service：

 - 在Service的基本用法这一部分，我们介绍了销毁Service最简单的一种情况，点击Start
   Service按钮启动Service，再点击Stop Service按钮停止Service，这样MyService就被销毁了。
 - 那么如果我们是点击的BindService按钮呢？由于在绑定Service的时候指定的标志位是BIND_AUTO_CREATE，说明点击BindService按钮的时候Service也会被创建，这时应该怎么销毁Service呢？其实也很简单，点击一下UnbindService按钮，将Activity和Service的关联解除就可以了
 - **那么如果我们既点击了StartService按钮，又点击了BindService按钮,这个时候，不管单独点击StopService按钮还是UnbindService按钮，Service都不会被销毁，必要将两个按钮都点击一下，Service才会被销毁。也就是说，点击StopService按钮只会让Service停止，点击UnbindService按钮只会让Service和Activity解除关联，一个Service必须要在既没有和任何Activity关联又处理停止状态的时候才会被销毁。**
 - 我们应该始终记得在Service的onDestroy()方法里去清理掉那些不再使用的资源，防止在Service被销毁后还会有一些不再使用的对象仍占用着内存。
 

*注：如果先startService()，则会先调用onCreate（）和onStartCommand（），再点击bindService（）时，只会执行绑定成功时执行的方法，不会再一次执行onCreate（）*。

##4.Service和Thread的关系
Service其实运行在主线程中，意思就是如果你在Service中执行耗时的代码，程序一定会AHR的。


这里面其实有一个后台和子线程的区别：
 - Android的后台就是指，它的运行是完全不依赖UI的。即使Activity被销毁，或者程序被关闭，只要进程还在，Service就可以继续运行。
 - 可是子线程是一个新的线程，和主线程并行执行。
 
比如说一些应用程序，始终需要与服务器之间始终保持着心跳连接，就可以使用Service来实现。你可能又会问，前面不是刚刚验证过Service是运行在主线程里的么？在这里一直执行着心跳连接，难道就不会阻塞主线程的运行吗？当然会，但是我们可以在Service中再创建一个子线程，然后在这里去处理耗时逻辑就没问题了。

额，既然在Service里也要创建一个子线程，那为什么不直接在Activity里创建呢？这是因为Activity很难对Thread进行控制，当Activity被销毁之后，就没有任何其它的办法可以再重新获取到之前创建的子线程的实例。而且在一个Activity中创建的子线程，另一个Activity无法对其进行操作。但是Service就不同了，所有的Activity都可以与Service进行关联，然后可以很方便地操作其中的方法，即使Activity被销毁了，之后只要重新与Service建立关联，就又能够获取到原有的Service中Binder的实例。因此，使用Service来处理后台任务，Activity就可以放心地finish，完全不需要担心无法对后台任务进行控制的情况。

理解：Activity中如果创建了一个新的线程，当此Activity结束以后，这个线程也就失控，就算下次这个Activity又被创建出来，它创建的新的线程也不在是刚才的那个线程。所以线程对于Activity来说，很难控制。可是如果在Service中创建线程，然后用Activity来绑定Service，就算Activity结束。下次Activity创建出来后，可以再次绑定这个Service，它还会是刚才的Service，在它里面创建的线程当然也是刚才的线程了。Service比线程共容易控制！

所以一个标准的Service就可以写成：
```
@Override
public int onStartCommand(Intent intent, int flags, int startId) {
	new Thread(new Runnable() {
		@Override
		public void run() {
			// 开始执行后台任务
		}
	}).start();
	return super.onStartCommand(intent, flags, startId);
}

class MyBinder extends Binder {

	public void startDownload() {
		new Thread(new Runnable() {
			@Override
			public void run() {
				// 执行具体的下载任务
			}
		}).start();
	}

}
```

##5.前台Service：
Service几乎都是在后台运行的，一直以来它都是默默地做着辛苦的工作。但是Service的系统优先级还是比较低的，当系统出现内存不足情况时，就有可能会回收掉正在后台运行的Service。如果你希望Service可以一直保持运行状态，而不会由于系统内存不足的原因导致被回收，就可以考虑使用前台Service。前台Service和普通Service最大的区别就在于，它会一直有一个正在运行的图标在系统的状态栏显示，下拉状态栏后可以看到更加详细的信息，非常类似于通知的效果。当然有时候你也可能不仅仅是为了防止Service被回收才使用前台Service，有些项目由于特殊的需求会要求必须使用前台Service，比如说墨迹天气，它的Service在后台更新天气数据的同时，还会在系统状态栏一直显示当前天气的信息。

如何才能创建一个前台Service，其实并不复杂，修改MyService中的代码，如下所示：
```
public class MyService extends Service {

	public static final String TAG = "MyService";

	private MyBinder mBinder = new MyBinder();

	@Override
	public void onCreate() {
		super.onCreate();
		Notification notification = new Notification(R.drawable.ic_launcher,
				"有通知到来", System.currentTimeMillis());
		Intent notificationIntent = new Intent(this, MainActivity.class);
		PendingIntent pendingIntent = PendingIntent.getActivity(this, 0,
				notificationIntent, 0);
		notification.setLatestEventInfo(this, "这是通知的标题", "这是通知的内容",
				pendingIntent);
		startForeground(1, notification);
		Log.d(TAG, "onCreate() executed");
	}

	.........

}
```
这里只是修改了MyService中onCreate()方法的代码。可以看到，我们首先创建了一个Notification对象，然后调用了它的setLatestEventInfo()方法来为通知初始化布局和数据，并在这里设置了点击通知后就打开MainActivity。然后调用startForeground()方法就可以让MyService变成一个前台Service，并会将通知的图片显示出来。
现在重新运行一下程序，并点击Start Service或Bind Service按钮，MyService就会以前台Service的模式启动了，并且在系统状态栏会弹出一个通栏图标，下拉状态栏后可以看到通知的详细内容.