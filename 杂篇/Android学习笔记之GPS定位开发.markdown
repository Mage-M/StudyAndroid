# Android学习笔记之GPS定位开发

标签（空格分隔）：Android

---
##1.需要一个LocationManager类的实例
```
    private LocationManager lm;
    lm = (LocationManager)getSystemService(LOCATION_SERVICE);
```
##2.需要criteria查询条件
```
    Criteria criteria = new Criteria();
    criteria.setAccuracy(Criteria.ACCURACY_FINE);//获取准确位置
    criteria.setCostAllowed(true);//允许产生开销
    String name = lm.getBestProvider(criteria, true);
    System.out.println("最好的位置提供者:" + name);
```
##3.给LocationManager类注册一个LocationListener
```
	private MyListener listener;
	listener = new MyListener();
    lm.requestLocationUpdates(name, 0, 0, listener);
	private class MyListener implements LocationListener {

		@Override
		public void onLocationChanged(Location location) {
			StringBuilder sb = new StringBuilder();
			sb.append("accuracy:" + location.getAccuracy() + "\n");
			sb.append("speed:" + location.getSpeed() + "\n");
			sb.append("jingdu:" + location.getLongitude() + "\n");
			sb.append("weidu:" + location.getLatitude() + "\n");
			String result = sb.toString();
			SharedPreferences sp = getSharedPreferences("config", MODE_PRIVATE);
			String safenumber = sp.getString("safephone", "");
			SmsManager.getDefault().sendTextMessage(safenumber, null, result, null, null);
			stopSelf();
		}

		//当位置提供者状态发生变化时调用的方法
		@Override
		public void onStatusChanged(String provider, int status, Bundle extras) {
			
		}

		//当某个位置提供者可用的时候调用的方法
		@Override
		public void onProviderEnabled(String provider) {
			
		}

		//当某个位置提供者不可用的时候调用的方法
		@Override
		public void onProviderDisabled(String provider) {
			
		}
		
	}
```	
以下是完整的代码展示，在这里监听到手机位置变化后，将发送一条短信至安全号码。
```
public class GPSLocationService extends Service{

	private LocationManager lm;
	private MyListener listener;
	
	@Override
	public IBinder onBind(Intent intent) {
		return null;
	}

	@Override
	public void onCreate() {
		super.onCreate();
		lm = (LocationManager)getSystemService(LOCATION_SERVICE);
		listener = new MyListener();
		//criteria查询条件
		//true只返回可用的位置提供者
		Criteria criteria = new Criteria();
		criteria.setAccuracy(Criteria.ACCURACY_FINE);//获取准确位置
		criteria.setCostAllowed(true);//允许产生开销
		String name = lm.getBestProvider(criteria, true);
		System.out.println("最好的位置提供者:" + name);
		lm.requestLocationUpdates(name, 0, 0, listener);
	}
	
	private class MyListener implements LocationListener {

		@Override
		public void onLocationChanged(Location location) {
			StringBuilder sb = new StringBuilder();
			sb.append("accuracy:" + location.getAccuracy() + "\n");
			sb.append("speed:" + location.getSpeed() + "\n");
			sb.append("jingdu:" + location.getLongitude() + "\n");
			sb.append("weidu:" + location.getLatitude() + "\n");
			String result = sb.toString();
			SharedPreferences sp = getSharedPreferences("config", MODE_PRIVATE);
			String safenumber = sp.getString("safephone", "");
			SmsManager.getDefault().sendTextMessage(safenumber, null, result, null, null);
			stopSelf();
		}

		//当位置提供者状态发生变化时调用的方法
		@Override
		public void onStatusChanged(String provider, int status, Bundle extras) {
			
		}

		//当某个位置提供者可用的时候调用的方法
		@Override
		public void onProviderEnabled(String provider) {
			
		}

		//当某个位置提供者不可用的时候调用的方法
		@Override
		public void onProviderDisabled(String provider) {
			
		}
		
	}

	@Override
	public void onDestroy() {
		super.onDestroy();
		lm.removeUpdates(listener);
		listener = null;
	}
	
	
	
}
```




