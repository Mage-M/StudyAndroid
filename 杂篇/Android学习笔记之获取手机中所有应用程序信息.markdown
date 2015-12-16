# Android学习笔记之获取手机中所有应用程序信息

标签（空格分隔）： Android

---
以下代码展示了如何通过PackageManager类获取android手机中所有应用程序的以下信息：

 - 应用程序包名
 - 应用程序图标
 - 应用程序名称
 - 应用程序路径
 - 应用程序大小
 - 是否是手机存储
 - 是否是用户自安装应用

```
public class AppInfoParser {

	public static List<AppInfo> getAppInfos(Context context) {
		PackageManager pm = context.getPackageManager();
		List<PackageInfo> packInfos = pm.getInstalledPackages(0);
		List<AppInfo> appinfos = new ArrayList<AppInfo>();
		for(PackageInfo packInfo : packInfos) {
			AppInfo appinfo = new AppInfo();
			String packname = packInfo.packageName;
			appinfo.packageName = packname;
			Drawable icon = packInfo.applicationInfo.loadIcon(pm);
			appinfo.icon = icon;
			String appname = packInfo.applicationInfo.loadLabel(pm).toString();
			appinfo.appName = appname;
			//应用程序apk包的路径
			String apkpath = packInfo.applicationInfo.sourceDir;
			appinfo.apkPath = apkpath;
			File file = new File(apkpath);
			long appSize = file.length();
			appinfo.appSize = appSize;
			//应用程序安装的位置
			int flags = packInfo.applicationInfo.flags;//二进制映射
			if((ApplicationInfo.FLAG_EXTERNAL_STORAGE & flags) != 0) {
				//外部存储
				appinfo.isInRoom = false;
			} else {
				//手机内存
				appinfo.isInRoom = true;
			}
			
			if((ApplicationInfo.FLAG_SYSTEM & flags) != 0) {
				//系统应用
				appinfo.isUserApp = false;
			} else {
				//用户应用
				appinfo.isUserApp = true;
			}
			
			appinfos.add(appinfo);
			appinfo = null;
		}
		return appinfos;
	}
}
```
其中的AppInfo为应用程序实体类：
```
//应用程序实体类
public class AppInfo {

	//应用程序包名
	public String packageName;
	//应用程序图标
	public Drawable icon;
	//应用程序名称
	public String appName;
	//应用程序路径
	public String apkPath;
	//应用程序大小
	public long appSize;
	//是否是手机存储
	public boolean isInRoom;
	//是否是用户应用
	public boolean isUserApp;
	//是否选中，默认都为false
	public boolean isSelected = false;
	
	//拿到app位置字符串
	public String getAppLocation(boolean isInRoom) {
		if(isInRoom) {
			return "手机内存";
		} else {
			return "外部存储";
		}
	}
}
```



