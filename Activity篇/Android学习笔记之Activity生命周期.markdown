# Android学习笔记之Activity生命周期

标签（空格分隔）： Android

---

##下面这张图讲解了activity的生命周期：(这个金字塔模型要比之前Dev Guide里面的生命周期图更加容易理解，更加形象)

在一个activity的生命周期中，系统会像金字塔模型一样去调用一系列的生命周期回调函数。Activity生命周期的每一个阶段就像金字塔中的台阶。当系统创建了一个新的activity实例，每一个回调函数会向上一阶移动activity状态。处在金字塔顶端意味着当前activity处在前台并处于用户可与其进行交互的状态。

当用户退出这个activity时，为了回收该activity，系统会调用其它方法来向下一阶移动activity状态。在某些情况下，activity会隐藏在金字塔下等待(例如当用户切换到其他app),此时activity可以重新回到顶端(如果用户回到这个activity)并恢复用户离开时的状态。

![效果图](https://github.com/Mage-M/StudyAndroid/raw/master/Activity篇/图片/Activity_lifecycle.png)

根据activity的复杂度，也许不需要实现所有的生命周期方法。但了解每一个方法的回调时机并在其中填充相应功能，使得确保app能够像用户期望的那样执行是很有必要的。如何实现一个符合用户期待的app，我们需要注意下面几点：

####1.使用app的时候，不会因为有来电通话或者切换到其他app而导致程序crash。
####2.用户没有激活某个组件时不会消耗宝贵的系统资源。
####3.离开app并且一段时间后返回，不会丢失用户的使用进度。
####4.设备发生屏幕旋转时不会crash或者丢失用户的使用进度。


下面的课程会介绍上图所示的几个生命状态。然而，其中只有三个状态是静态的，这三个状态下activity可以存在一段比较长的时间。(其它几个状态会很快就切换掉，停留的时间比较短暂)

####Resumed：该状态下，activity处在前台，用户可以与它进行交互。(通常也被理解为"running" 状态)
####Paused：该状态下，activity的部分被另外一个activity所遮盖：另外的activity来到前台，但是半透明的，不会覆盖整个屏幕。被暂停的activity不再接受用户的输入且不再执行任何代码。
####Stopped：该状态下, activity完全被隐藏，对用户不可见。可以认为是在后台。当stopped, activity实例与它的所有状态信息（如成员变量等）都会被保留，但activity不能执行任何代码。
####其它状态 (Created与Started)都是短暂的，系统快速的执行那些回调函数并通过执行下一阶段的回调函数移动到下一个状态。也就是说，在系统调用onCreate(), 之后会迅速调用onStart(), 之后再迅速执行onResume()。以上就是基本的activity生命周期。
---
##几种状态详解：
###1.Active状态：这时候Activity处于栈顶，且是可见的，有焦点的，能够接收用户输入前景Activity。OPhone Runtime将试图不惜一切代价保持它活着，甚至杀死其他Activity以确保它有它所需的资源。当另一个Activity变成Active时，当前的将变成Paused状态。
---
###2.Paused状态：在某些情况下，你的Activity是可见的，但没有焦点，在这时候，Actvity处于Paused状态。例如，如果有一个透明或非全屏幕上的Activity在你的Actvity上面，你的 Activity将。当处于Paused状态时，该Actvity仍被认为是Active的，但是它不接受用户输入事件。在极端情况下，OPhone Runtime将杀死Paused Activity，以进一步回收资源。当一个Actvity完全被遮住时，它将进入Stopped状态。
---
###3.Stopped 状态：当Activity是不可见的时，Activity处于Stopped状态。Activity将继续保留在内存中保持当前的所有状态和成员信息，假设系统别的地方需要内存的话，这时它是被回收对象的主要候选。当Activity处于Stopped状态时，一定要保存当前数据和当前的UI状态，否则一旦Activity退出或关闭时，当前的数据和UI状态就丢失了。
---
###4.另外，还有一点要注意，Activity在处于onPause、onStop、onDestroy状态下，系统都可以销毁该Activity所在进程，所以我们在处理一些要保存的数据时，必须在onPause方法中进行，因为onStop和onDestroy方法不一定会被调用。
---
###注:
####1.只要该Activity处于看不见的状态想要再次显示，则都需要调用3个方法：
    -->未启动时：(1)onCreate();
                 (2)onStart();
                 (3)onResume();
    -->从stop到继续运行：
                 (1)onRestart();
                 (2)onStart();
                 (3)onResume();
####2.不管什么状态下在Activity显示出来之前调用的都是onResume()；
