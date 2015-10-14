title: Android基础整理
date: 2015-10-09 12:12:33
categories: Android
tags:
- Android

---

### Intent

####
用来启动Activity:

	startActivity();
	
	//带返回结果
	startActivityForResult();
	
	//获取返回结果的方法
	onActivityResult();
	
用来启动Service:

	//启动服务
	startService();
	
	//绑定服务
	bindService();
	
传递广播:

	sendBroadcast();
	
	sendOrderedBroadcast();
	
	sendStickyBroadcast();