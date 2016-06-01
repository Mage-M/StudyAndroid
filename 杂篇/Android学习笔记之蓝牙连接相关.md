# Android学习笔记之蓝牙连接相关

标签（空格分隔）： Android

---

##最近完成了关于android蓝牙这一块的开发，在这里做一些概括上的总结。

###1.android蓝牙开发需要依赖到的jar包。
    有一些在开发中所用到的java类，我在android5.0的API中并没有找到相应的类。所以，应该是API中的隐藏java类，在这里，我将我没有在API中找到的Java类全部贴在下面。
    
```
import android.bluetooth.BluetoothHeadsetClient;
import android.bluetooth.BluetoothProfile;
import android.bluetooth.BluetoothA2dpSink;
import android.bluetooth.BluetoothHeadsetClientCall;
import android.bluetooth.BluetoothProfile.ServiceListener;
import android.bluetooth.BluetoothAvrcpController;
import android.bluetooth.client.map.BluetoothMapBmessage;
import android.bluetooth.client.map.BluetoothMapEventReport;
import android.bluetooth.client.map.BluetoothMapMessage;
import android.bluetooth.client.map.BluetoothMasClient;
import android.bluetooth.client.pbap.BluetoothPbapCard;
import android.bluetooth.client.pbap.BluetoothPbapClient;
```

###2.与蓝牙连接相关的一些广播
####(1)**BluetoothDevice.ACTION_ACL_DISCONNECTED**
>    首先这里需要明确在蓝牙连接过程中的几个阶段。
    ——首先两个蓝牙设备之间会建立一条最基础的链路，这条链路就叫做ACL，以后所有的连接都是在这个ACL连接的基础之上的。
    ——然后，我们会在ACL连接的基础上，进行其他profile的连接。常用的几个profile如hfp，a2dp，pbap等都是在ACL的基础上进行连接的。
    ——最后，各种profile连接完成，蓝牙的多样化功能都是建立在这些各式各样的profile之上的。

**所以，这条广播的意思就是最底层的那个链接断开。当我们收到这条广播的时候，说明这两个蓝牙设备之间最低层的那个连接已经断开，我们可以根据这个广播来判定两个设备蓝牙断开，并在自己的程序中设置相应的标志。**
    
####（2）**BluetoothDevice.ACTION_ACL_CONNECTED**
>    与第一个广播相对应，当收到这条广播时，证明需要连接的两个蓝牙设备之间已经建立了一条最基本的链路。
    但是在这里需要声明一点，这条广播并不一定适合做程序中判断蓝牙是否连接的标志信号。因为当这个广播发出时，只是底层铺好了链路，实际上完成功能的profile还正在连接中，这时立即使用的话会出现一些异常。    
>    最明显的例子就是，当两个从未配过对的蓝牙设备进行配对时，当弹出配对对话框，但是并没有点击配对时，我们已经可以收到这条广播，但是如果你的程序使用这个广播作为已经成功连接的标志信号的话，这时就已经会设置你的程序为已经连接状态了，但是实际上我们还并没有配对。
    
 **所以对于这条广播，不建议使用其做连接成功的标志信号。**
 
####(3)**Intent.ACTION_SHUTDOWN**与**Intent.ACTION_BOOT_COMPLETED**
>    这两条广播相信大家不会陌生，分别是android的关机广播和开机广播，在平时使用中，其实使用的不多。
    
>    其中比较典型的应用是在收到开机广播后启动蓝牙程序的自动重连功能，或者在收到关机广播以后断开蓝牙的连接等。
**在蓝牙程序中，这两条广播一般使用的较少。**

####(4)**BluetoothHeadsetClient.ACTION_CONNECTION_STATE_CHANGED**

>在讲这条广播之前，又需要前置一些别的东西，那就是profile的概念。在第一条广播的时候就曾经讲到过，其实当ACL的连接建立以后，只是准备好了一些最基本的连接。但是，一般我们使用的功能都是建立在Profile之上的，只有当这些profile都连接成功之后，我们才可以通过蓝牙来进行一些高级功能的操作。

**下面接收几种常用的Profile:**

 - 1.HSP（Handset Profile）、HFP（Hands-free Profile） HSP即耳机模式，用于支持蓝牙耳机与移动电话之间使用。
   HFP耳机模式，HFP在HSP的基础上增加了某些扩展功能。让蓝牙设备可以控制电话，如接听、挂断、拒接、语音拨号等，拒接、语音拨号要视蓝牙耳机及电话是否支持。即如果你的蓝牙程序需要支持蓝牙电话的功能，则需要连接这个profile。
 - 2.PAN（个人局域网配置文件） PAN 描述了两个或更多个 Bluetooth设备如何构成一个即时网络，以及如何使用同一机制通过网络接入点接入远程网络。配置文件角色包括网络接入点、组即时网络及个人局域网用户。即你的蓝牙程序需要连接以后支持类似网盘的功能，需要连接实现这个profile。
 - 3.A2DP（Advanced Audio Distribution Profile） 蓝牙音频传输模型协议，蓝牙立体声，和蓝牙耳机听歌有关。即如果你的蓝牙程序需要实现连接上以后播放歌曲这个功能的话，就要连接这个profile。这个profile也是使用最多的profile，一般的运动蓝牙耳机就只实现了这个profile，用来在运动时连接手机，可以听到手机端传过来的歌曲。
 - 4.AVRCP （音/视频遥控协议） 用来听歌时暂停，上下歌曲选择的。即如果你的蓝牙运动耳机在放歌的同时，还可以进行换歌、暂停等一系列的操作，那么它就一定实现了这个profile。
 - 5.PBAP（Phonebook Access Profile） 电话号码簿访问协议，此协议的功能就是如果你的蓝牙程序实现了蓝牙电话以后，需要读取到与你连接的手机的联系人信息，则需要连接这个profile。

在有了以上这个几个profile的概念以后，我们继续这个广播的了解。

>**这个广播的含义是，当我们的设备与另一个设备通过蓝牙连接时，我们也连接了HFP这个profile时，当这个profile的连接状态发生变化时，我们就会收到这条广播。即这条广播的作用是用来通知我们该profile的连接状态发生了变化，需要我们对此变化做出相应的反应**。

这里需要***注意***的是，如果你的蓝牙程序需要实现的是蓝牙电话的功能，那么可以通过这个广播来设置蓝牙连接成功的标志信号。即当收到这个状态变化的广播时，我们取profile的连接状态，如果其状态为**BluetoothProfile.STATE_CONNECTED**，则我们就可以在UI上标明蓝牙连接成功，因为这时，我们才可以真正通过蓝牙来接听或拨打电话，前面提到的问题也得到解决。否则，我们用ACL连接作为标志，那是HFP可能还未连接，我们接打电话当然会出现异常。

####(5)**BluetoothDevice.ACTION_BOND_STATE_CHANGED**
>    当收到这条广播时，证明你正在连接的两个蓝牙设备之间的配对信息发生了改变。
    
设备之间的配对状态分为以下三种：
 1. BOND_BONDED
 2. BOND_BONDING
 3. BOND_NONE

**这三种状态分别代表了两个设备已经配对、正在配对和无配对。**
    
>    当收到这条广播时，证明你的两个蓝牙设备的配对状态从这三种中的一种变成了另一种。如果有些操作必须配对以后才能执行，我们就可以在收到这条广播之后对设备的配对状态进行一次读取和判断，当其状态是BOND_BONDED时，再进行下面的操作。

####(6)**BluetoothHeadsetClient.ACTION_CALL_CHANGED**
>这里首先要说的不是这个广播，而是当连接HFP时，在打电话的过程中，有可能出现的几种状态：
(这些状态都是在当你手机通过蓝牙连接到某一个可以执行蓝牙电话功能的设备上，并且连接了HFP这个profile之后，在通话的过程中可能产生的)
1.BluetoothHeadsetClientCall.CALL_STATE_ACTIVE
当主叫或者被叫电话被接通时，会上报为这个状态。
2.BluetoothHeadsetClientCall.CALL_STATE_DIALING
当主叫时，在拨出号码的最开始一端时间内，会上报为这个状态。
3.BluetoothHeadsetClientCall.CALL_STATE_ALERTING
当主叫时，在拨出号码之后，别人未接通之前，会上报这个状态。
4.BluetoothHeadsetClientCall.CALL_STATE_INCOMING
当被叫时，在你未接通电话之前，会上报这个状态。
5.BluetoothHeadsetClientCall.CALL_STATE_HELD
当你正在通话中，有第三方来电且接通后，则刚才你为挂断的电话，会上报为这个状态。
6.BluetoothHeadsetClientCall.CALL_STATE_TERMINATED
当主叫或者被叫电话通话中，将电话挂断后，会上报这个状态。


**现在回到这条广播上来，当收到这条广播时，证明你的通话状态发生了改变，就是在上面我们说到的这几种状态中变化。这时我们就可以取到当前的状态，然后根据当前状态来执行需要执行的操作。**

比如，我们在收到这条广播时，立刻判定当前状态，如果当前状态是有来电，那么我们就初始化电话UI；或者当我们判断当前状态是挂断电话后，我们就可以销毁电话UI并添加通话记录等。

**活用这条广播和其之后的状态信息，可以让我们做许多灵活的操作。**

###3.关于蓝牙连接
>    在这个蓝牙程序中，并没有过多的去干涉底层的连接过程，当然最主要的原因还是自己菜逼。
    我所做的主要操作就是在android源生的蓝牙连接之后，报出各种广播后，我根据监听这些广播来执行一些相应的操作。
    在开发的过程中也碰到过底层协议层解析错误的问题，不过最后也不是我解掉的。在这里就不多说了。