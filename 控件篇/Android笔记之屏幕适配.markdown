# Android笔记之屏幕适配

标签（空格分隔）： Android

---

**本文介绍了Android支持不同屏幕大小的方法：**

##1.在定义View时用该多使用"wrap_content" 和 "match_parent"
wrap_content：该View的宽和高就会被设定为刚好可以包含视图中内容的最小值。
match_parent：该View的宽和高会被拉长至充满整个父布局。

多使用这两个参数，就会使你的View所在的布局正确的适应不同的屏幕。

我们使用LinearLayout加上这两个参数可以布置出足够复杂的布局。

##2.RelativeLayout的使用
在此之前讲一下盒模型，区别一下padding和margin。
盒模型
为了更加准确地控制TextView里面内容的位置，我们可以使用一系列的padding属性来控制。在使用padding属性之前，先科普一下padding和Marigin之间的区别，然后我们在通过实际的效果看看他们之间的差异。
下图所示是一个类似盒子的模型，我们将通过下面的模型来讲解Padding和Marigin之间的区别。从图中可以看出，在Container（父控件）里面有一个子控件，假设是一个TextView控件。


![效果图](https://github.com/Mage-M/StudyAndroid/raw/master/控件篇/图片/boxmodel.png)



 - 其中Margin是子控件与父控件之间的间隔大小。
 - Border是子控件的边框，它是子控件和父控件的边界。
 - Padding是指子控件中的内容(Content Area)与子控件Border的间隔大小。
 - android:layout_alignParentRight="true" 属性是子控件针对父容器的。且父容器必须是RelativeLayout。
 
对应的属性为
android:layout_marginBottom="25dip" 
android:layout_marginLeft="10dip" 
android:layout_marginTop="10dip" 
android:layout_marginRight="10dip" 
android:paddingLeft="1dip" 
android:paddingTop="1dip" 
android:paddingRight="1dip" 
android:paddingBottom="1dip"

 - 如果左右上下都是相同的设置则可以直接设置

android:layout_margin="10dip" 
android:padding="5dip"

 - 当按钮分别设置以上两个属性时，得到的效果是不一样的。

android:paddingLeft="30px"：

 - 按钮上设置的内容（例如图片）离按钮左边边界30个像素。

android:layout_marginLeft="30px"

整个按钮离左边设置的内容30个像素

 - 这二个属性是相对的，假设B是A的子控件，设置B的margin和设置A的padding能达到相同的效果

。

 - 设置padding的好处：

     如果imageview对应的图片比较小，点击不容易点中，通过增加padding可以增大点触敏感度


**如果你需要让子视图能够有更多的排列方式，而不是简单地排成一行或一列，使用RelativeLayout将会是更好的解决方案。**
*因为设置的是相对位置，所以即使屏幕的大小改变，View之间的相对位置也不会改变。*
**RelativeLayout允许布局的子控件之间使用相对定位的方式控制控件的位置，比如你可以让一个子视图居屏幕左侧对齐，让另一个子视图居屏幕右侧对齐。**


##3.使用Size限定符
虽然使用以上几种方式可以解决屏幕适配性的问题，但是那些通过伸缩控件来适应各种不同屏幕大小的布局，未必就是提供了最好的用户体验。你的应用程序应该不仅仅实现了可自适应的布局，还应该提供一些方案根据屏幕的配置来加载不同的布局，可以通过配置限定符(configuration qualifiers)来实现。配置限定符允许程序在运行时根据当前设备的配置自动加载合适的资源(比如为不同尺寸屏幕设计不同的布局)。

现在有很多的应用程序为了支持大屏设备，都会实现“two pane”模式(程序会在左侧的面板上展示一个包含子项的List，在右侧面板上展示内容)。平板和电视设备的屏幕都很大，足够同时显示两个面板，而手机屏幕一次只能显示一个面板，两个面板需要分开显示。所以，为了实现这种布局，你需要两个布局文件。

分别在：
res/layout/main.xml
    可能有一个fragment，小屏幕时加载
res/layout-large/main.xml
    可能有两个fragment，大于7寸时加载
这两个地方放置两个名称相同的布局文件，注意第二个文件夹需要我们自己来创建，而且名称必须都是相同的，这样在加载不同屏幕大小的同一个布局时才可以找到。

**请注意第二个布局的目录名中包含了large限定符，那些被定义为大屏的设备(比如7寸以上的平板)会自动加载此布局，而小屏设备会加载另一个默认的布局。**

##4.使用Smallest-width限定符
使用Size限定符有一个问题会让很多程序员感到头疼，large到底是指多大呢？很多应用程序都希望能够更自由地为不同屏幕设备加载不同的布局，不管它们是不是被系统认定为"large"。这就是Android为什么在3.2以后引入了"Smallest-width"限定符。

Smallest-width限定符允许你设定一个具体的最小值(以dp为单位)来指定屏幕。例如，7寸的平板最小宽度是600dp，所以如果你想让你的UI在这种屏幕上显示two pane，在更小的屏幕上显示single pane，你可以使用sw600dp来表示你想在600dp以上宽度的屏幕上使用two pane模式。

分别在：
res/layout/main.xml
    可能有一个fragment，小屏幕时加载
res/layout-sw600dp/main.xml
    可能有两个fragment，大于7寸时加载
    
这意味着，那些最小屏幕宽度大于600dp的设备会选择layout-sw600dp/main.xml(two-pane)布局，而更小屏幕的设备将会选择layout/main.xml(single-pane)布局。
然而，使用早于Android 3.2系统的设备将无法识别sw600dp这个限定符，所以你还是同时需要使用large限定符。这样你就需要在res/layout-large和res/layout-sw600dp目录下都添加一个相同的main.xml。

##5.使用Nine-Patch图片
支持不同屏幕大小通常情况下也意味着，你的图片资源也需要有自适应的能力。例如，一个按钮的背景图片必须能够随着按钮大小的改变而改变。
如果你想使用普通的图片来实现上述功能，你很快就会发现结果是令人失望的，因为运行时会均匀地拉伸或压缩你的图片。解决方案是使用nine-patch图片，它是一种被特殊处理过的PNG图片，你可以指定哪些区域可以拉伸而哪些区域不可以。

因而，当你设计需要在不同大小的控件中使用的图片时，最好的方法就是用nine-patch图片。为了将图片转换成nine-patch图片，你可以从一张普通的图片开始：
![效果图](https://github.com/Mage-M/StudyAndroid/raw/master/控件篇/图片/9p1.png)


然后通过SDK中带有的draw9patch工具打开这张图片(工具位置在SDK的tools目录下)，你可以在图片的左边框和上边框绘制来标记哪些区域可以被拉伸。你也可以在图片的右边框和下边框绘制来标记内容需要放置在哪个区域。结果如下图所示：

![效果图](https://github.com/Mage-M/StudyAndroid/raw/master/控件篇/图片/9p2.png)

注意图片边框上的黑色像素，在上边框和左边框的部分表示当图片需要拉伸时就拉伸黑点标记的位置。在下边框和右边框的部分表示内容将会被放置的区域。

**同时需要注意，这张图片的后缀名是 .9.png。你必须要使用这个后缀名，因为系统就是根据这个来区别nine-patch图片和普通的PNG图片的**。

当你需要在一个控件中使用nine-patch图片时(如android:background="@drawable/button")，系统就会根据控件的大小自动地拉伸你想要拉伸的部分，效果如下图所示：

![效果图](https://github.com/Mage-M/StudyAndroid/raw/master/控件篇/图片/9p3.png)



