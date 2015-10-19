# Android学习笔记之Activity的四种LaunchMode

标签（空格分隔）： Android

---

###Android的Activity有四种启动模式，在我看了一些资料，博客后，在这里总结如下：

##Activity的四种启动模式分别如下：

 - standard
 - singleTop
 - singleTask
 - singleInstance
 
设置的位置在AndroidManifest.xml文件中activity元素的android:launchMode属性：

<activity android:launchMode ="singleTask"></activity>

##standard
standard模式每次都会创建新的实例，可以在界面中打印出对象的toString值然后根据其hashcode识别确实是新的activity实例。standard的加载模式就是这样的，intent将发送给新的实例。

现在点Android设备的回退键，可以看到是按照刚才创建Activity实例的倒序依次出现，类似退栈的操作，而刚才操作跳转按钮的过程是压栈的操作。如下图：

![效果图](https://github.com/Mage-M/StudyAndroid/raw/master/Activity篇/图片/standard.png)


##singleTop
singleTop和standard模式，都会将intent发送新的实例（后两种模式不发送到新的实例，如果已经有了的话）。不过，singleTop要求如果创建intent的时候栈顶已经有要创建的Activity的实例，则将intent发送给该实例，而不发送给新的实例。

运行的时候会发现，按多少遍按钮，都是相同的ActiA实例，因为该实例在栈顶，因此不会创建新的实例。如果回退，将退出应用。（从activityA跳到activityA）

![效果图](https://github.com/Mage-M/StudyAndroid/raw/master/Activity篇/图片/singleTop1.png)

singleTop模式，可用来解决栈顶多个重复相同的Activity的问题。

如果是A Activity跳转到B Activity，再跳转到A Activity，行为就和standard一样了，会在B Activity跳转到A Activity的时候创建A Activity的新实例，因为当时的栈顶不是A Activity实例。
ActB类使用默认（standard）加载，ActA使用singleTop加载。结果类似下图：

![效果图](https://github.com/Mage-M/StudyAndroid/raw/master/Activity篇/图片/singleTop2.png)


##singleTask
singleTask模式和后面的singleInstance模式都是只创建一个实例的。

当intent到来，需要创建singleTask模式Activity的时候，系统会检查栈里面是否已经有该Activity的实例。如果有直接将intent发送给它。

把ActA的launchMode改为singleTask，ActB的改为standard。那么会发现在ActA界面中按一次按钮跳到ActB：

![效果图](https://github.com/Mage-M/StudyAndroid/raw/master/Activity篇/图片/singleTask1.png)


然后在ActB1界面中按按钮跳到ActA，因为ActA是singleTask，会使用原来的ActA1实例。这时候栈内的情况:

![效果图](https://github.com/Mage-M/StudyAndroid/raw/master/Activity篇/图片/singleTask2.png)

如果多次按按钮跳转，会发现始终只有ActA1这一个ActA类的实例。

##singleInstance
首先要说一下Task（任务）的概念。

如果是Swing或者Windows程序，可能有多个窗口可以切换，但是你无法在自己程序中复用人家的窗口。注意是直接复用人家的二进制代码，不是你拿到人家api后的源代码级调用。

Android可以做到，让别人的程序直接复用你的Activity（类似桌面程序的窗口）。

Android为提供这种机制，就引入了Task的概念。Task可以认为是一个栈，可放入多个Activity。比如启动一个应用，那么 Android就创建了一个Task，然后启动这个应用的入口Activity，就是intent-filter中配置为main和launch的那个 （见一个APK文件部署产生多个应用安装的效果 ）。这个Activity是根（Root）Activity，可能会在它的界面调用其他Activity，这些Activity如果按照上面那三个模式，也会在这个栈（Task）中，只是实例化的策略不同而已。

验证的办法是调用和打印Activity的taskId：

TextView textView2 = new TextView(this); 
textView2.setText("task id: "+this.getTaskId());

会发现，无论切换Activity，taskId是相同的。

当然也可以在这个单一的Task栈中，放入别人的Activity，比如google地图，这样用户看过地图按回退键的时候，会退栈回到调用地图的Activity。对用户来说，并不觉得在操作多个应用。这就是Task的作用。

但是，有这样的需求，多个Task共享一个Activity（singleTask是在一个task中共享一个Activity）。

现成的例子是google地图。比如我有一个应用是导游方面的，其中调用的google地图Activity。那么现在我比如按home键，然后到应用列表中打开google地图，你会发现显示的就是刚才的地图，实际上是同一个Activity。

如果使用上面三种模式，是无法实现这个需求的。google地图应用中有多个上下文Activity，比如路线查询等的，导游应用也有一些上下文Activity。在各自应用中回退要回退到各自的上下文Activity中。

singleInstance模式解决了这个问题（绕了这么半天才说到正题）。让这个模式下的Activity单独在一个task栈中。这个栈只有一个Activity。导游应用和google地图应用发送的intent都由这个Activity接收和展示。

这里又有两个问题：

 - 如果是这种情况，多个task栈也可以看作一个应用。比如导游应用启动地图Activity，实际上是在导游应用task栈之上
   singleInstance模式创建的（如果还没有的话，如果有就是直接显示它）一个新栈，当这个栈里面的唯一Activity，地图Activity
   回退的时候，只是把这个栈移开了，这样就看到导游应用刚才的Activity了；

 - 多个应用（Task）共享一个Activity要求这些应用都没有退出，比如刚才强调要用home键从导游应用切换到地图应用。因为，如果退出导游应用，而这时也地图应用并未运行的话，那个单独的地图Activity（task）也会退出了。

如果还是拿刚才的ActA和ActB的示例，可以把ActB的模式改为singleInstance，ActA为standard，如果按一次按钮切换到ActB，看到现象用示意图类似这样：

![效果图](https://github.com/Mage-M/StudyAndroid/raw/master/Activity篇/图片/singleInstance1.png)

如果是第一次按钮切换到ActB，在ActB在按按钮切换到ActA，然后再回退，示意图是：

![效果图](https://github.com/Mage-M/StudyAndroid/raw/master/Activity篇/图片/singleInstance.png)

另外，可以看到两个Activity的taskId是不同的。
