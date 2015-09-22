---
layout: post
title: 使用软引用解决Handler内存泄露和显示Popupwindow、Dialog时提示"Unable to add Window-token is null"的问题
category: Android
tags: Android
keywords: SoftReferences、Handler、Dialog、Popupwindow
description: 使用软引用解决Handler内存泄露和显示Popupwindow、Dialog时提示"Unable to add Window-token is null"的问题
---

## 通过软引用解决Handler内存泄露的问题

&emsp;&emsp;下面对软引用使用的方式适用于任何内部类，严格来说是通过软引用解决静态内部类无法调用当前类中的对象和方法的问题，真正解决内存泄露是需要将内部类改成静态内部类。

&emsp;&emsp;当在一个类中按照如下方式创建一个Handler内部类时，使用Lint工具检测时会给出“This Handler class should be static or leaks might occur”的警告，原因是Handler内部类可能持有当前类的引用，导致即使该类不再被使用时系统仍无法回收这给类持有的对象，Android中内存泄露很多都是由于持有类中的对象时间太长导致，如果很多地方出现类似的代码会导致应用占用的内存不断上涨，最终导致程序崩溃。

	private TextView mUserNameTxt = null;

    class LoadDataHandler extends Handler{
		@Override
		public void handleMessage(Message msg) {
			super.handleMessage(msg);
			switch(msg.what){
			case 0:{
				mUserNameTxt.setText(msg.obj.toString());
			}
			break;
			default:{
				
			}
			break;
			}
		}
    }

&emsp;&emsp;按照Lint的建议将Handler内部类改成static静态内部类后，由于不可能将当前类的所有全局对象都声明为static对象，所以会报“Cannot make a static reference to the non-static field”的错误，这时候可以使用软引用来解决这个问题，具体代码如下：

	private TextView mUserNameTxt = null;
    static class LoadDataHandler extends Handler{
    	private SoftReference<MainActivity> activitySRF = null;
    	public LoadDataHandler(MainActivity activity){
    		activitySRF = new SoftReference<MainActivity>(activity);
    	}
    	
		@Override
		public void handleMessage(Message msg) {
			super.handleMessage(msg);
			// 因为Handler是异步的，存在退出当前类之后才接收到handler消息的情况，并且软引用持有的对象会在堆内存不足时存在被回收的可能，所以这里需要判空处理
			if(null == activitySRF || null == activitySRF.get()){
				return;
			}
			
			switch(msg.what){
			case 0:{
				activitySRF.get().mUserNameTxt.setText(msg.obj.toString());
			}
			break;
			default:{
				
			}
			break;
			}
		}
    }

## 通过软引用解决销毁某个Activity后，仍然在Activity显示Popupwindow或者Dialog时提示"Unable to add Window-token is null"的问题

&emsp;&emsp;在实际项目中踩过这个坑，特地分享出来，这个问题是由于在Activity中显示PopupWindow或者Dialog时，PopupWindow、Dialog还没显示出来就销毁了当前Activity导致（主要是快速切换）。因为PopupWindow和Dialog都是依附于当前Activity的某个View上的，当前Activity被销毁后依附的view为空，此时显示PopupWindow或者Dialog时会提示这个异常。

&emsp;&emsp;这个问题也可以通过软引用来解决(通过判断软引用中的activity对象是否为空或者是否已经finish即可判断是否可以显示PopupWindow或者Dialog)，具体代码如下：
	
	private SoftReference<Activity> activitySRF = null;
	public void initDialog(Activity activity){
		activitySRF = new SoftReference<Activity>(activity);
		if(null != activitySRF&&null!=activitySRF.get()&&!activitySRF.get().isFinishing()){
			getWindow().requestFeature(Window.FEATURE_NO_TITLE);
			show();
			setContentView(R.layout.hanzi_medal_dialoglayout);
			setCanceledOnTouchOutside(true);
	        setCancelable(false);
		}
	}
	
	public void show(Activity activity, String medalName){
		initDialog(activity);
		if(null != activitySRF&&null!=activitySRF.get()&&!activitySRF.get().isFinishing()){
			TextView medalTxt = (TextView) findViewById(R.id.medalTxtId);
			if (!TextUtils.isEmpty(medalName)) {
				medalTxt.setText("恭喜您获得“" + medalName + "”勋章");
			}
			new Handler(activitySRF.get().getMainLooper()).postDelayed(new Runnable() {
				@Override
				public void run() {
					if(null != activitySRF&&null!=activitySRF.get()&&!activitySRF.get().isFinishing()){
						dismiss();
					}
				}
			}, 3000);
		}
	}


