# Android学习笔记之单位dp与px互转的工具类

标签（空格分隔）： Android

---

下面这个类实现了dp与px的相互转化：
```
public class DensityUtil {

	//dp转换像素px
	public static int dip2px(Context context,float dpValue) {
		try {
			final float scale = context.getResources().getDisplayMetrics().density;
			return (int) (dpValue * scale + 0.5f);
		} catch(Exception e) {
			e.printStackTrace();
		}
		return (int)dpValue;
	}
	
	//像素px转换为dp
	public static int px2dip(Context context,float pxValue) {
		try {
			final float scale = context.getResources().getDisplayMetrics().density;
			return (int) (pxValue / scale + 0.5f);
		} catch(Exception e) {
			e.printStackTrace();
		}
		return (int)pxValue;
	}
}
```



