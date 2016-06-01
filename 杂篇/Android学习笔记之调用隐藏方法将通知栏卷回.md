# Android学习笔记之调用隐藏方法将通知栏卷回

标签（空格分隔）： Android

---

##在蓝牙电话的开发中，碰到了一个问题。
    在通知栏拉下的状态下，如果来电的话，必须先要将通知栏手动上拉后才能对来电进行接听。所以，这个方法就可以解决，在来电时，创建接听界面以前，系统自动将通知栏回卷，使用户可以直接接听电话。
    
    
  ###  *这个问题只是一种场景，但是此种回卷通知栏的方法还可以有其他许多用处。故在此进行记录。*
  
  
```
try {
			Object service = getSystemService("statusbar");
			Class<?> statusbarManager =
					Class.forName("android.app.StatusBarManager");
			Method test = statusbarManager.getMethod("collapsePanels");
			test.invoke(service);
		} catch(Exception e) {
			e.printStackTrace();
		}
```		

该方法通过反射的方法调用了StatusBarManager里面的collapsePanels方法，让状态栏回卷。

