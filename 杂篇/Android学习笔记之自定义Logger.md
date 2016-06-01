# Android学习笔记之自定义Logger

标签（空格分隔）： Android

---

###在日常的Android程序开发调试的过程中，分析Log是必不可少的一种非常有用的途径。这里我通过自己重新定义的Logger工具，封装了系统的Log方法，使打印出的Log更加的清晰和容易辨认，且每个java类打出的log都有自己的TAG，方便过滤。在后期项目上线以后，也可以通过开关变量，将所有调试log全部关闭，提高效率。

```

import android.util.Log;

public class Logger {

    // Android Tag for logging.
    private static final String TAG = "此处是项目名";

    private static final boolean DEBUG = true;

    //这四个boolean变量就是Log的开关，在开发调试阶段我们将log打开，方便调试。
    //在项目上线以后，上面的DEBUG变量置为false以后，所有的调试Log就将不再继续打出。
    private static final boolean VLOG = DEBUG && true;      
    private static final boolean DLOG = DEBUG && true;
    private static final boolean ILOG = DEBUG && true;
    private static final boolean WLOG = DEBUG && true;      //w开头的log为黄色，醒目
    private static final boolean ELOG = DEBUG && true;      //e开头的log为红色，更醒目

    /**
     * Constructor.
     */
    private Logger() {
    }

    public static void v(String tag, String msg) {
        if (VLOG) {
            Log.v(TAG, "[" + tag + "] " + msg);
        }
    }

    public static void d(String tag, String msg) {
        if (DLOG) {
            Log.d(TAG, "[" + tag + "] " + msg);
        }
    }

    public static void i(String tag, String msg) {
        if (ILOG) {
            Log.i(TAG, "[" + tag + "] " + msg);
        }
    }

    public static void w(String tag, String msg) {
        if (WLOG) {
            Log.w(TAG, "[" + tag + "] " + msg);
        }
    }

    public static void e(String tag, String msg) {
        if (ELOG) {
            Log.e(TAG, "[" + tag + "] " + msg);
        }
    }

}

```




