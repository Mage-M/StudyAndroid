# Android学习笔记之异步消息处理机制（在子线程中更新UI）

标签（空格分隔）： Android

---

**今天看了很多讲异步消息处理，在这做一下自己的总结，方便以后复习。如果有引用到谁的代码段或描述，不要见怪 - -**
---
##在子线程中进行UI操作一共有四种方式：
###1.发送Message，并重写Handle中的handleMessage（）方法
###2.Handle的post（）方法
###3.View的post（）方法
###4.Activity的runOnUiThread（）方法
---
其中最常用的可能就是第一和第四两种方法，下面我分别将这四种方法分析一下：

##1.发送Message，并重写Handle中的handleMessage（）方法
（1）创建Handler对象只能在主线程中，在子线程中创建会导致崩溃，提示为 Can't create handler inside thread that has not called Looper.prepare()。说明Handler的创建是依赖Looper.prepare()这个方法的。
（2）如果我们在子线程中调用Looper.prepare()这个方法后，发现再次创建就不会再崩溃。为什么会这样？
在查阅资料后发现Handler（）的构造方法中需要获取一个Looper对象，如果为空，就会抛出一个异常，正是前面的那个异常。
```java
    public Handler() {
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }
        mLooper = Looper.myLooper();        //此方法获取一个Looper对象
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = null;
    }
```
（3）根据郭神博客主线继续，何时Looper.myLooper()为空呢？
```java
public static final Looper myLooper() {
    return (Looper)sThreadLocal.get();
}
```
发现是在sThreadLocal.get()方法取出Looper，如果没有就返回空，返回空以后Looper.myLooper()方法失败，所以导致Handler的构造方法失败，抛出异常。所以又回到了前面的问题，怎么样让它不为空呢？就是调用Looper.prepare()方法啦，这样你在子线程中创建Handler就不会抛异常啦！
（4）我们来看看Looper.prepare()是怎么样设置Looper的
```java
public static final void prepare() {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper());
}
```
可以看到，首先判断sThreadLocal中是否已经存在Looper了，如果还没有则创建一个新的Looper设置进去。这样也就完全解释了为什么我们要先调用Looper.prepare()方法，才能创建Handler对象。同时也可以看出每个线程中最多只会有一个Looper对象

（5）这个时候我们会有疑问，主线程中也没有调用Looper.prepare()方法，为什么没有崩溃呢？郭神这个省略了一下，我花时间找了一下，发现在我们主线程程序启动的时候，会执行ActivityThread中的main()方法，这个类是Android源码中的类，是程序启动是调用的。意思就是我们的主线程启动以前，系统会调用这个方法来启动主线程，这个方法优先级还在onStart（）方法之上。
```java
public static void main(String[] args) {
    SamplingProfilerIntegration.start();
    CloseGuard.setEnabled(false);
    Environment.initForCurrentUser();
    EventLogger.setReporter(new EventLoggingReporter());
    Process.setArgV0("<pre-initialized>");
    Looper.prepareMainLooper();         //在这个方法中调用了prepare（）方法
    ActivityThread thread = new ActivityThread();
    thread.attach(false);
    if (sMainThreadHandler == null) {
        sMainThreadHandler = thread.getHandler();
    }
    AsyncTask.init();
    if (false) {
        Looper.myLooper().setMessageLogging(new LogPrinter(Log.DEBUG, "ActivityThread"));
    }
    Looper.loop();
    throw new RuntimeException("Main thread loop unexpectedly exited");
}
```

接着是 Looper.prepareMainLooper()代码：
```java
public static final void prepareMainLooper() {
    prepare();                  //  在这里就调用了prepare（）方法
    setMainLooper(myLooper());
    if (Process.supportsProcesses()) {
        myLooper().mQueue.mQuitAllowed = false;
    }
}
```

现在我终于搞懂了为什么主线程不需要再去调用Looper.prepare()方法了。到这Handler的创建流程完了，下来是发送消息的流程。

（6）发送消息用handler.sendMessage(message);可是Handler将这些Message发送到了哪里去呢？ok，继续跟着郭神走！通过查看Handler的源码我们发现Handler中提供了很多个发送消息的方法，其中除了sendMessageAtFrontOfQueue()方法之外，其它的发送消息方法最终都会辗转调用到sendMessageAtTime()方法中，这个方法的源码如下所示：
```java
public boolean sendMessageAtTime(Message msg, long uptimeMillis)
{
    boolean sent = false;
    MessageQueue queue = mQueue;                        //消息队列
    if (queue != null) {
        msg.target = this;                                                                                        //上面这行说明msg.target为Handler，因为这个方法是在Handler中被用来发送消息的
        sent = queue.enqueueMessage(msg, uptimeMillis); //将发送的消息入队
    }
    else {
        RuntimeException e = new RuntimeException(
            this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
    }
    return sent;
}
```
这个方法第二个参数表示发送消息的时间。
MessageQueue就是一个消息队列，它是在Looper的构造函数中创建的，因此一个Looper也就只能对应一个
MessageQueue。这个队列还提供入队和出队的方法，第7行的就是入队的方法了。
入队的源码我就不贴了，只在这说一下里面的大概内容。
MessageQueue并没有使用一个集合把所有的消息都保存起来，它只使用了一个mMessages对象表示当前待处理的消息。所谓的入队其实就是将所有的消息按时间来进行排序，这个时间当然就是我们刚才介绍的uptimeMillis参数，处理完一个就将mMessages表示为时间最新的下一条消息。具体的操作方法就根据时间的顺序调用msg.next，从而为每一个消息指定它的下一个消息是什么。当然如果你是通过sendMessageAtFrontOfQueue()方法来发送消息的，意思就是插队，它也会调用enqueueMessage()来让消息入队，只不过时间为0，这时会把mMessages赋值为新入队的这条消息，然后将这条消息的next指定为刚才的mMessages，这样也就完成了添加消息到队列头部的操作。
出队的方法在Looper.loop()方法中调用的：
```java
public static final void loop() {
    Looper me = myLooper();
    MessageQueue queue = me.mQueue;
    while (true) {
        Message msg = queue.next(); // 这个next就是出队方法了
        if (msg != null) {
            if (msg.target == null) {
                return;
            }
            if (me.mLogging!= null) me.mLogging.println(
                    ">>>>> Dispatching to " + msg.target + " "
                    + msg.callback + ": " + msg.what
                    );
            msg.target.dispatchMessage(msg);
            if (me.mLogging!= null) me.mLogging.println(
                    "<<<<< Finished to    " + msg.target + " "
                    + msg.callback);
            msg.recycle();
        }
    }
}
```
可以看到，这个方法从第4行开始，进入了一个死循环，然后不断地调用的MessageQueue的next()方法，这个next()方法就是消息队列的出队方法。它的简单逻辑就是如果当前MessageQueue中存在mMessages(即待处理消息)，就将这个消息出队，然后让下一条消息成为mMessages，否则就进入一个阻塞状态，一直等到有新的消息入队。继续看loop()方法的第14行，每当有一个消息出队，就将它传递到msg.target的dispatchMessage()方法中，那这里msg.target又是什么呢？其实就是Handler啦，你观察一下上面sendMessageAtTime()方法的第6行就可以看出来了。接下来当然就要看一看Handler中dispatchMessage()方法的源码了
```java
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {                                                                           //这个callback，如果调用Handler的post（）方法，将在其中设置为传入其中Runnable对象
        handleCallback(msg);    //如果是post方法调用
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {         
                return;
            }
        }
        handleMessage(msg);     //正常异步消息调用
    }
}
```
在第5行进行判断，如果mCallback不为空，则调用mCallback的handleMessage()方法，否则直接调用Handler的handleMessage()方法，并将消息对象作为参数传递过去。
所以，一个标准的异步消息处理线程的写法应该是这样：

```
    class LooperThread extends Thread {
          public Handler mHandler;
          public void run() {
              Looper.prepare();
              mHandler = new Handler() {
                  public void handleMessage(Message msg) {
                      // process incoming messages here
                  }
              };
              Looper.loop();
          }
      }
```
这就是消息的接收过程。

那么我们还是要来继续分析一下，为什么使用异步消息处理的方式就可以对UI进行操作了呢？这是由于Handler总是依附于创建时所在的线程，比如我们的Handler是在主线程中创建的，而在子线程中又无法直接对UI进行操作，于是我们就通过一系列的发送消息、入队、出队等环节，最后调用到了Handler的handleMessage()方法中，这时的handleMessage()方法已经是在主线程中运行的，因而我们当然可以在这里进行UI操作了。

##2.Handle的post（）方法

（1）我们先来看下Handler中的post()方法，代码如下所示：
```java
public final boolean post(Runnable r)
{
   return  sendMessageDelayed(getPostMessage(r), 0);
}
```
（2）post方法使用sendMessageDelayed（）发送了一条消息，并且还使用getPostMessage（）方法将Runnable对象转换成了一条消息，源码如下：
```java
private final Message getPostMessage(Runnable r) {
    Message m = Message.obtain();
    m.callback = r;
    return m;
}
```
（3）这个方法中将m的callback字段指定为传入的Runnable对象。在Handler的dispatchMessage()方法中原来有做一个检查，如果Message的callback等于null才会去调用handleMessage()方法，否则就调用handleCallback()方法。handleCallback()方法中的代码如下：
```java
private final void handleCallback(Message message) {
    message.callback.run();
}
```
我们发现它直接就调用了调用了一开始传入的Runnable对象的run()方法。因此在子线程中通过Handler的post()方法进行UI操作就可以这么写：
```java
public class MainActivity extends Activity {

	private Handler handler;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		handler = new Handler();
		new Thread(new Runnable() {
			@Override
			public void run() {
				handler.post(new Runnable() {
					@Override
					public void run() {
						// 在这里进行UI操作
					}
				});
			}
		}).start();
	}
}
```

其实原理和第一种是一样的。

##3.View的post（）方法
```java
public boolean post(Runnable action) {
    Handler handler;
    if (mAttachInfo != null) {
        handler = mAttachInfo.mHandler;
    } else {
        ViewRoot.getRunQueue().post(action);
        return true;
    }
    return handler.post(action);
}
```
其实就是调用了Handler的post（）方法而已，和第二种原理完全相同

##4.Activity的runOnUiThread（）方法
```java
public final void runOnUiThread(Runnable action) {
    if (Thread.currentThread() != mUiThread) {
        mHandler.post(action);
    } else {
        action.run();
    }
}
```
其实也是一样，如果当前线程不是主线程，则就去调用Handler的post（）方法，否则就直接调用Runnable对象的run（）方法就好了。也没有新的原理。

---
ok，这就是四种在子线程中更新UI的方法，其实等于说只有两种，最后总结下只能归结成一种，就是异步消息处理。今天花这么多时间在这，就是为了一次弄透彻，以后不会忘！谢谢郭神！ - -