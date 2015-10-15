# Android学习笔记之Activity生命周期

标签（空格分隔）： Android

---

##下面这张图讲解了activity的生命周期：(这个金字塔模型要比之前Dev Guide里面的生命周期图更加容易理解，更加形象)

根据activity的复杂度，也许不需要实现所有的生命周期方法。但了解每一个方法的回调时机并在其中填充相应功能，使得确保app能够像用户期望的那样执行是很有必要的。如何实现一个符合用户期待的app，我们需要注意下面几点：

1.使用app的时候，不会因为有来电通话或者切换到其他app而导致程序crash。
2.用户没有激活某个组件时不会消耗宝贵的系统资源。
3.离开app并且一段时间后返回，不会丢失用户的使用进度。
4.设备发生屏幕旋转时不会crash或者丢失用户的使用进度。
---
##几种状态详解：
###1.Active状态：这时候Activity处于栈顶，且是可见的，有焦点的，能够接收用户输入前景Activity。OPhone Runtime将试图不惜一切代价保持它活着，甚至杀死其他Activity以确保它有它所需的资源。当另一个Activity变成Active时，当前的将变成Paused状态。
---
###2.Paused状态：在某些情况下，你的Activity是可见的，但没有焦点，在这时候，Actvity处于Paused状态。例如，如果有一个透明或非全屏幕上的Activity在你的Actvity上面，你的 Activity将。当处于Paused状态时，该Actvity仍被认为是Active的，但是它不接受用户输入事件。在极端情况下，OPhone Runtime将杀死Paused Activity，以进一步回收资源。当一个Actvity完全被遮住时，它将进入Stopped状态。
---
###3.Stopped 状态：当Activity是不可见的时，Activity处于Stopped状态。Activity将继续保留在内存中保持当前的所有状态和成员信息，假设系统别的地方需要内存的话，这时它是被回收对象的主要候选。当Activity处于Stopped状态时，一定要保存当前数据和当前的UI状态，否则一旦Activity退出或关闭时，当前的数据和UI状态就丢失了。
---
*###4.另外，还有一点要注意，Activity在处于onPause、onStop、onDestroy状态下，系统都可以销毁该Activity所在进程，所以我们在处理一些要保存的数据时，必须在onPause方法中进行，因为onStop和onDestroy方法不一定会被调用。*
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
