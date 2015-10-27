# Android笔记之Service和Activity之间进行通信

标签（空格分隔）： Android

---

[TOC]

通过阅读http://blog.csdn.net/xiaanming/article/details/9750689这个博客，将Service与Activity之间通信方式总结如下：

##1.通过Binder对象绑定。

新建一个回调接口：
```
public interface OnProgressListener {
	void onProgress(int progress);
}
```

Service类：
```
public class MsgService extends Service {
	/**
	 * 进度条的最大值
	 */
	public static final int MAX_PROGRESS = 100;
	/**
	 * 进度条的进度值
	 */
	private int progress = 0;
	
	/**
	 * 更新进度的回调接口
	 */
	private OnProgressListener onProgressListener;
	
	
	/**
	 * 注册回调接口的方法，供外部调用
	 * @param onProgressListener
	 */
	public void setOnProgressListener(OnProgressListener onProgressListener) {
		this.onProgressListener = onProgressListener;
	}

	/**
	 * 增加get()方法，供Activity调用
	 * @return 下载进度
	 */
	public int getProgress() {
		return progress;
	}

	/**
	 * 模拟下载任务，每秒钟更新一次
	 */
	public void startDownLoad(){
		new Thread(new Runnable() {
			
			@Override
			public void run() {
				while(progress < MAX_PROGRESS){
					progress += 5;
					
					//进度发生变化通知调用方
					if(onProgressListener != null){
						onProgressListener.onProgress(progress);
					}
					
					try {
						Thread.sleep(1000);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					
				}
			}
		}).start();
	}


	/**
	 * 返回一个Binder对象
	 */
	@Override
	public IBinder onBind(Intent intent) {
		return new MsgBinder();
	}
	
	public class MsgBinder extends Binder{
		/**
		 * 获取当前Service的实例
		 * @return
		 */
		public MsgService getService(){
			return MsgService.this;
		}
	}

}
```

Activity类：
```
public class MainActivity extends Activity {  
    private MsgService msgService;  
    private ProgressBar mProgressBar;  
      
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
          
          
        //绑定Service  
        Intent intent = new Intent("com.example.communication.MSG_ACTION");  
        bindService(intent, conn, Context.BIND_AUTO_CREATE);  
          
          
        mProgressBar = (ProgressBar) findViewById(R.id.progressBar1);  
        Button mButton = (Button) findViewById(R.id.button1);  
        mButton.setOnClickListener(new OnClickListener() {  
              
            @Override  
            public void onClick(View v) {  
                //开始下载  
                msgService.startDownLoad();  
            }  
        });  
          
    }  
      
  
    ServiceConnection conn = new ServiceConnection() {  
        @Override  
        public void onServiceDisconnected(ComponentName name) {  
              
        }  
          
        @Override  
        public void onServiceConnected(ComponentName name, IBinder service) {  
            //返回一个MsgService对象  
            msgService = ((MsgService.MsgBinder)service).getService();  
              
            //注册回调接口来接收下载进度的变化  
            msgService.setOnProgressListener(new OnProgressListener() {  
                  
                @Override  
                public void onProgress(int progress) {  
                    mProgressBar.setProgress(progress);  
                      
                }  
            });  
              
        }  
    };  
  
    @Override  
    protected void onDestroy() {  
        unbindService(conn);  
        super.onDestroy();  
    }  
  
  
}  
```

##2.通过BroadCast（广播）的形式：

Activity类如下：
```
public class MainActivity extends Activity {
	private ProgressBar mProgressBar;
	private Intent mIntent;
	private MsgReceiver msgReceiver;
	

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		
		//动态注册广播接收器
		msgReceiver = new MsgReceiver();
		IntentFilter intentFilter = new IntentFilter();
		intentFilter.addAction("com.example.communication.RECEIVER");
		registerReceiver(msgReceiver, intentFilter);
		
		
		mProgressBar = (ProgressBar) findViewById(R.id.progressBar1);
		Button mButton = (Button) findViewById(R.id.button1);
		mButton.setOnClickListener(new OnClickListener() {
			
			@Override
			public void onClick(View v) {
				//启动服务
				mIntent = new Intent("com.example.communication.MSG_ACTION");
				startService(mIntent);
			}
		});
		
	}

	
	@Override
	protected void onDestroy() {
		//停止服务
		stopService(mIntent);
		//注销广播
		unregisterReceiver(msgReceiver);
		super.onDestroy();
	}


	/**
	 * 广播接收器
	 * @author len
	 *
	 */
	public class MsgReceiver extends BroadcastReceiver{

		@Override
		public void onReceive(Context context, Intent intent) {
			//拿到进度，更新UI
			int progress = intent.getIntExtra("progress", 0);
			mProgressBar.setProgress(progress);
		}
		
	}

}
```
Service类如下：
```
public class MsgService extends Service {
	/**
	 * 进度条的最大值
	 */
	public static final int MAX_PROGRESS = 100;
	/**
	 * 进度条的进度值
	 */
	private int progress = 0;
	
	private Intent intent = new Intent("com.example.communication.RECEIVER");
	

	/**
	 * 模拟下载任务，每秒钟更新一次
	 */
	public void startDownLoad(){
		new Thread(new Runnable() {
			
			@Override
			public void run() {
				while(progress < MAX_PROGRESS){
					progress += 5;
					
					//发送Action为com.example.communication.RECEIVER的广播
					intent.putExtra("progress", progress);
					sendBroadcast(intent);
					
					try {
						Thread.sleep(1000);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					
				}
			}
		}).start();
	}

	

	@Override
	public int onStartCommand(Intent intent, int flags, int startId) {
		startDownLoad();
		return super.onStartCommand(intent, flags, startId);
	}



	@Override
	public IBinder onBind(Intent intent) {
		return null;
	}


}
```
##3.通过共享文件交互
这里提到的共享文件指的是Activity和Service使用同一个文件来达到传递数据的目的。我们使用SharedPreferences来实现共享，当然也可以使用其它IO方法实现，通过这种方式实现交互时需要注意，对于文件的读写的时候，同一时间只能一方读一方写，不能两方同时写。

实现原理：Server端将当前下载进度写入共享文件中，Client端通过读取共享文件中的下载进度，并更新到主界面上。

实现步骤：
在Client端通过startService()启动Service
```
if(startSerBtn==v){
	Log.i(TAG, "Start Button Clicked.");
	if(intent!=null){
	startService(intent);
	timer.schedule(new MyTimerTask(), 0, TIME * 1000);
	}
}
```

Server端收到启动intent之后执行onCreate()方法，并开启timer，模拟下载，以及初始化SharedPreferences对象preferences
```
@Override
public void onCreate() {
	super.onCreate();
	Log.i(TAG, "DownLoadService.onCreate()...");
	preferences = getSharedPreferences("CurrentLoading_SharedPs", 0);
	timer = new Timer();
	timer.schedule(new MyTimerTask(), 0, TIME*1000);
}
```

开始计数并将下载进度写入shared_prefs文件夹下的xml文件中，内容以键值对的方式保存。
```
class MyTimerTask extends TimerTask{
	@Override
	public void run() {
		setCurrentLoading();
		if(100==i){
			i=0;
		}
		i++;
	}		
}	
private void setCurrentLoading() {
	preferences.edit().putInt("CurrentLoading", i).commit();
}
```

Client端通过读取/data/data/com.seven.servicetestdemo/shared_prefs文件夹下的xml文件，并取得里面的键值对，从而获取到当前的下载进度，并更新到主界面上。
```
Handler mHandler = new Handler(){
	@Override
	public void handleMessage(Message msg) {
		super.handleMessage(msg);
		int couLoad = preferences.getInt("CurrentLoading", 0);
		mProgressBar.setProgress(couLoad);
		currentTv.setText(couLoad+"%");
	}
 };
```





##4.总结：

 - Activity调用bindService (Intent service, ServiceConnection conn, int
   flags)方法，得到Service对象的一个引用，这样Activity可以直接调用到Service中的方法，如果要主动通知Activity，我们可以利用回调方法
 - Service向Activity发送消息，可以使用广播，当然Activity要注册相应的接收器。比如Service要向多个Activity发送同样的消息的话，用这种方法就更好
 -  对於使用共享文件这种方式实现Activity与Service的交互，可以说很方便，就像使用管道，一个往裡写，一个往外读。但这种方式也有缺陷，写入数据较为复杂以及数据量较大时，就有可能导致写入与读数据出不一致的错误。同时因为经过了一个中转站，这种操作将更耗时

