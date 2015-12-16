# Android学习笔记之获取手机和SD卡剩余内存

标签（空格分隔）： Android

---
一下这段代码用于获取手机内存和SD卡内存的剩余值
```
private void getMemoryFromPhone() {
		long avail_sd = Environment.getExternalStorageDirectory().getFreeSpace();
		long avail_rom = Environment.getDataDirectory().getFreeSpace();
		//格式化内存
		String str_avail_sd = Formatter.formatFileSize(this, avail_sd);
		String str_avail_rom = Formatter.formatFileSize(this,avail_rom);
		mPhoneMemoryTV.setText("剩余手机内存:" + str_avail_rom);
		mSDMemoryTV.setText("剩余SD卡内存:" + str_avail_sd);
	}
```

第5,6行是为了将其byte格式化为多少M或者多少G，会自动匹配单位



