# Android笔记之进程的生命周期

标签（空格分隔）： Android

---

作为一个多任务的系统，Android 当然系统能够尽可能长的保留一个应用进程。但是由于新的或者更重要的进程需要更多的内存，系统不得不逐渐终结老的进程来获取内存。为了声明哪些进程需要保留，哪些需要kill，系统根据这些进程里面的组件以及这些组件的状态为每个进程生成了一个“重要性层级” 。处于最低重要性层级的进程将会第一时间被清楚，接着时重要性高一点，然后依此类推，根据系统需要来终结进程。

在这个重要性层级里面有5个等级。下面的列表按照重要性排序展示了不同类型的进程（第一种进程是最重要的，因此将会在最后被kill）:

 - Foreground 进程 一个正在和用户进行交互的进程。

 如果一个进程处于下面的状态之一，那么我们可以把这个进程称为 foreground 进程:
进程包含了一个与用户交互的 Activity  (这个 Activity的 onResume() 方法被调用)。
进程包含了一个绑定了与用户交互的activity的 Service 。
进程包含了一个运行在”in the foreground”状态的 Service —这个 service 调用了 startForeground()方法。
进程包含了一个正在运行的它的生命周期回调函数 (onCreate(), onStart(), oronDestroy())的 Service 。
进程包含了一个正在运行 onReceive() 方法的 BroadcastReceiver 。
一般说来，任何时候，系统中只存在少数的 foreground 进程。 只有在系统内存特别紧张以至于都无法继续运行下去的时候，系统才会通过kill这些进程来缓解内存压力。在这样的时候系统必须kill一些 （Generally, at that point, the device has reached a memory paging state，这句如何翻译较好呢）foreground 进程来保证 用户的交互有响应。

 - Visible 进程 一个进程没有任何 foreground 组件, 但是它还能影响屏幕上的显示。

 如果一个进程处于下面的状态之一，那么我们可以把这个进程称为 visible 进程:
进程包含了一个没有在foreground 状态的 Activity ，但是它仍然被用户可见 (它的 onPause() 方法已经被调用)。这种情况是有可能出现的，比如，一个 foreground activity 启动了一个 dialog，这样就会让之前的 activity 在dialog的后面部分可见。
进程包含了一个绑定在一个visible（或者foreground）activity的 Service 。
一个 visible 进程在系统中是相当重要的，只有在为了让所有的foreground 进程正常运行时才会考虑去kill visible 进程。

 - Service 进程 一个包含着已经以 startService() 方法启动的 Service
   的进程，同时还没有进入上面两种更高级别的种类。
尽管 service 进程没有与任何用户所看到的直接关联，但是它们经常被用来做用户在意的事情（比如在后台播放音乐或者下载网络数据），所以系统也只会在为了保证所有的foreground and visible 进程正常运行时kill掉 service 进程。

 - Background 进程 一个包含了已不可见的activity的 进程 (这个 activity 的 onStop() 已经被调用)。

这样的进程不会直接影响用户的体验，系统也可以为了foreground 、visible 或者 service 进程随时kill掉它们。一般说来，系统中有许多的 background 进程在运行，所以将它们保持在一个LRU (least recently used)列表中可以确保用户最近看到的activity 所属的进程将会在最后被kill。如果一个 activity 正确的实现了它的生命周期回调函数，保存了自己的当前状态，那么kill这个activity所在的进程是不会对用户在视觉上的体验有影响的，因为当用户回退到这个 activity时，它的所有的可视状态将会被恢复。查看 Activities 可以获取更多如果保存和恢复状态的文档。

 - Empty 进程 一个不包含任何活动的应用组件的进程。

 这种进程存在的唯一理由就是缓存。为了提高一个组件的启动的时间需要让组件在这种进程里运行。为了平衡进程缓存和相关内核缓存的系统资源，系统需要kill这些进程。
Android是根据进程中组件的重要性尽可能高的来评级的。比如，如果一个进程包含来一个 service 和一个可见 activity，那么这个进程将会被评为 visible 进程，而不是 service 进程。

另外，一个进程的评级可能会因为其他依附在它上面的进程而被提升—一个服务其他进程的进程永远不会比它正在服务的进程评级低的。比如，如果进程A中的一个 content provider 正在为进程B中的客户端服务，或者如果进程A中的一个 service 绑定到进程B中的一个组件，进程A的评级会被系统认为至少比进程B要高。

因为进程里面运行着一个 service 的评级要比一个包含background activities的进程要高，所以当一个 activity 启动长时操作时，最好启动一个 service 来做这个操作，而不是简单的创建一个worker线程—特别是当这个长时操作可能会拖垮这个activity。比如，一个需要上传图片到一个网站的activity 应当开启一个来执行这个上传操作。这样的话，即使用户离开来这个activity也能保证上传动作在后台继续。使用 service 可以保证操作至少处于”service process” 这个优先级，无论这个activity发生了什么。这也是为什么 broadcast receivers 应该使用 services 而不是简单的将耗时的操作放到线程里面。




###线程

当一个应用启动的时候，系统会为它创建一个线程，称为“主线程”。这个线程很重要因为它负责处理调度事件到相关的 user interface widgets，包括绘制事件。你的应用也是在这个线程里面与来自Android UI toolkit (包括来自 android.widget 和 android.view 包的组件)的组件进行交互。因此，这个主线程有时候也被称为 UI 线程。

系统没有为每个组件创建一个单独的线程。同一进程里面的所有组件都是在UI 线程里面被实例化的，系统对每个组件的调用都是用过这个线程进行调度的。所以，响应系统调用的方法（比如 onKeyDown() 方法是用来捕捉用户动作或者一个生命周期回调函数）都运行在进程的UI 线程里面。

比如，当用户点击屏幕上的按钮，你的应用的UI 线程会将这个点击事件传给 widget，接着这个widget设置它的按压状态，然后发送一个失效的请求到事件队列。这个UI 线程对请求进行出队操作，然后处理（通知这个widget重新绘制自己）。

当你的应用与用户交互对响应速度的要求比较高时，这个单线程模型可能会产生糟糕的效果（除非你很好的实现了你的应用）。特别是，当应用中所有的事情都发生在UI 线程里面，那些访问网络数据和数据库查询等长时操作都会阻塞整个UI线程。当整个线程被阻塞时，所有事件都不能被传递，包括绘制事件。这在用户看来，这个应用假死了。甚至更糟糕的是，如果UI 线程被阻塞几秒（当前是5秒）以上，系统将会弹出臭名昭著的 “application not responding” (ANR) 对话框。这时用户可能选择退出你的应用甚至卸载。

另外，Android的UI 线程不是线程安全的。所以你不能在一个worker 线程操作你的UI—你必须在UI线程上对你的UI进行操作。这有两条简单的关于Android单线程模型的规则：

 - 不要阻塞 UI 线程
 - 不要在非UI线程里访问 Android UI toolkit
### Worker 线程

由于上面对单一线程模型的描述，保证应用界面的及时响应同时UI线程不被阻塞变得很重要。如果你不能让应用里面的操作短时被执行玩，那么你应该确保把这些操作放到独立的线程里(“background” or “worker” 线程)。

比如，下面这段代码在一个额外的线程里面下载图片并在一个 ImageView显示：



    public void onClick(View v){
        new Thread(new Runnable(){
            public void run(){
                Bitmap b = loadImageFromNetwork("http://example.com/image.png");
                mImageView.setImageBitmap(b);
            }
        }).start();}

起先这段代码看起来不错，因为它创建一个新的线程来处理网络操作。然而，它违反来单一线程模型的第二条规则: 不在非UI线程里访问 Android UI toolkit—这个例子在一个worker线程修改了 ImageView 。这会导致不可预期的结果，而且还难以调试。
public void onClick(View v){
    new Thread(new Runnable(){
        public void run(){
            Bitmap b = loadImageFromNetwork("http://example.com/image.png");
            mImageView.setImageBitmap(b);
        }
    }).start();}

为了修复这个问题，Android提供了几个方法从非UI线程访问Android UI toolkit 。详见下面的这个列表：

Activity.runOnUiThread(Runnable)
View.post(Runnable)
View.postDelayed(Runnable, long)
那么，你可以使用 View.post(Runnable) 方法来修改之前的代码：



    public void onClick(View v) {
        new Thread(new Runnable() {
            public void run() {
                final Bitmap bitmap = loadImageFromNetwork("http://example.com/image.png");
                mImageView.post(new Runnable() {
                    public void run() {
                        mImageView.setImageBitmap(bitmap);
                    }
                });
            }
        }).start();
    }


现在这个方案的线程安全的：这个网络操作在独立线程中完成后，UI线程便会对ImageView 进行操作。

然而，随着操作复杂性的增长，代码会变得越来越复杂，越来越难维护。为了用worker 线程处理更加复杂的交互，你可以考虑在worker线程中使用Handler ，用它来处理UI线程中的消息。也许最好的方案就是继承 AsyncTask 类，这个类简化了需要同UI进行交互的worker线程任务的执行。

###使用 AsyncTask

AsyncTask 能让你在UI上进行异步操作。它在一个worker线程里进行一些阻塞操作然后把结果交给UI主线程，在这个过程中不需要你对线程或者handler进行处理。

使用它，你必须继承 AsyncTask 并实现 doInBackground() 回调方法，这个方法运行在一个后台线程池里面。如果你需要更新UI，那么你应该实现onPostExecute()，这个方法从 doInBackground() 取出结果，然后在 UI 线程里面运行，所以你可以安全的更新你的UI。你可以通过在UI线程调用 execute()方法来运行这个任务。

比如，你可以通过使用 AsyncTask来实现之前的例子：


    public void onClick(View v) {
        new DownloadImageTask().execute("http://example.com/image.png");
    }
     
    private class DownloadImageTask extends AsyncTask<String, Void, Bitmap> {
        /** The system calls this to perform work in a worker thread and
          * delivers it the parameters given to AsyncTask.execute() */
        protected Bitmap doInBackground(String... urls) {
            return loadImageFromNetwork(urls[0]);
        }
     
        /** The system calls this to perform work in the UI thread and delivers
          * the result from doInBackground() */
        protected void onPostExecute(Bitmap result) {
            mImageView.setImageBitmap(result);
        }
    }

现在UI是安全的了，代码也更加简单了，因为AsyncTask把worker线程里做的事和UI线程里要做的事分开了。

你应该阅读一下 AsyncTask 的参考文档以便更好的使用它。下面就是一个对 AsyncTask 如何作用的快速的总览：

 - 你可以具体设置参数的类型，进度值，任务的终值，使用的范型
 - doInBackground() 方法自动在 worker 线程执行
 - onPreExecute(), onPostExecute(), 和 onProgressUpdate() 方法都是在UI线程被调用
 - doInBackground() 的返回值会被送往 onPostExecute()方法
 - 你可以随时在 doInBackground()方法里面调用 publishProgress() 方法来执行UI
   线程里面的onProgressUpdate() 方法
 - 你可以从任何线程取消这个任务

注意: 你在使用worker线程的时候可能会碰到的另一个问题就是因为runtime configuration change (比如用户改变了屏幕的方向)导致你的activity不可预期的重启，这可能会kill掉你的worker线程。为了解决这个问题你可以参考 Shelves 这个项目。

线程安全的方法

在某些情况下，你实现的方法可能会被多个线程所调用，因此你必须把它写出线程安全的。