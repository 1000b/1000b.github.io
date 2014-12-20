---
layout: post
title: 按Home按键退出应用后重新启动该应用无法返回到最后打开页面的解决方案
category: Android
tags: Android
keywords: Android，Home，lastest activity
description: 按Home按键退出应用后重新启动该应用无法返回到最后打开页面的解决方案
---

&emsp;&emsp;比如我打开应用到MatchActivity，正常的启动流程是：SplashActivity -> MainActivity -> MatchActivity，在MatchActivity界面按HOME键返回到桌面，如果长按HOME，在最近打开应用列表中重新打开该应用，能够恢复到刚才退出的界面，但如果是点击该应用的桌面图标重新打开该应用，则会显示一下SplashActivity后再才会进入MatchActivity，这个问题的解决方案是，在BaseActivity的onCrate方法中作如下处理（BaseActivity为所有应用中所有Activity的基类）：

	public abstract class BaseActivity extends Activity{
		@Override
		protected void onCreate(Bundle savedInstanceState){
			super.onCreate(savedInstanceState);
			if ((getIntent().getFlags() & Intent.FLAG_ACTIVITY_BROUGHT_TO_FRONT) != 0) { 
	            // Activity was brought to front and not created, 
	            // Thus finishing this will get us to the last viewed activity 
	            finish(); 
	            return; 
	        } 
	
			init();
		}
	
		public void init(){
			setContentView();
			findViews();
			getData();
			showContent();
		}
		
		@Override
		public void onPause(){
			super.onPause();
		}
		
		@Override
		public void onResume(){
			super.onResume();
		}
	
		public abstract void setContentView();
	
		public abstract void findViews();
	
		public abstract void getData();
	
		public abstract void showContent();
	}
	
&emsp;&emsp;具体原因众说纷纭，只要记住这样处理就好了。

**参考资料：**

- [How to return to the latest launched activity when re-launching application after pressing HOME?](http://stackoverflow.com/questions/6337217/how-to-return-to-the-latest-launched-activity-when-re-launching-application-afte)

- [App always starts fresh from root activity instead of resuming background state (Known Bug)](http://stackoverflow.com/questions/2280361/app-always-starts-fresh-from-root-activity-instead-of-resuming-background-state)

- [下载完点击“打开应用”后，按HOME键回到桌面，再次点击应用，没有回到之前的页面，而是打开新的应用](http://blog.csdn.net/it_talk/article/details/39368137)