---
layout: post
title: Android监听Home按键消息
category: Android
tags: Android
keywords: Android，home， Intent.ACTION_CLOSE_SYSTEM_DIALOGS
description: Android监听Home按键消息
---

&emsp;&emsp;Android对屏幕下方常用的四个按键消息处理是不一致的：

- 搜索按键的消息在**onKeyDown**或者**onKeyUp**中接收；
- 菜单按键的消息在**onCreateOptionsMenu、onKeyDown或onKeyUp**方法中接收；
- 返回按键的消息可以在**onBackPressed、onKeyDown或onKeyUp**方法中接收。
		
		@Override
	    public boolean onKeyDown(int keyCode, KeyEvent event) {
	    	switch( keyCode ){
	    	case KeyEvent.KEYCODE_BACK:{
	    		
	    	}
	    	break;
	    	case KeyEvent.KEYCODE_MENU:{
	    		
	    	}
	    	break;
	    	case KeyEvent.KEYCODE_SEARCH:{
	    		
	    	}
	    	break;
	    	default:{
	    		
	    	}
	    	break;
	    	}
	    	return super.onKeyDown(keyCode, event);
	    }
	    
	    @Override
	    public boolean onKeyUp(int keyCode, KeyEvent event) {
	    	switch( keyCode ){
	    	case KeyEvent.KEYCODE_BACK:{
	    		
	    	}
	    	break;
	    	case KeyEvent.KEYCODE_MENU:{
	    		
	    	}
	    	break;
	    	case KeyEvent.KEYCODE_SEARCH:{
	    		
	    	}
	    	break;
	    	default:{
	    		
	    	}
	    	break;
	    	}
	    	return super.onKeyUp(keyCode, event);
	    }
	    
	    @Override
		public boolean onCreateOptionsMenu(Menu menu) {
			return true;
		}
	    
	    @Override
	    public void onBackPressed() {
	    	super.onBackPressed();
	    }

&emsp;&emsp;对于Home按键消息的处理，既不能通过onKeyDown、onKeyUp接收到，android也没有提供专有的方法接收按键消息，个人估计home按键算是一个app异常信息处理的后门，比如ANR后，按其它按钮没有比按Home按键好使，所以android为了能够提供更好的用户体验，没有提供供用户监听home按键消息的方法。

**注：**网上提供了各种各样监听Home按键消息的方法，但只有这个比较好使。

&emsp;&emsp;但办法总是有的，在每次点击Home按键时都会发出一个action为Intent.ACTION_CLOSE_SYSTEM_DIALOGS的广播，它是关闭系统Dialog的广播，我们可以通过注册它来监听Home按键消息，我自定义了一个home按键监听工具类，代码如下，使用说明参见类名上方的使用说明：
	import android.content.BroadcastReceiver;
	import android.content.Context;
	import android.content.Intent;
	import android.content.IntentFilter;
	
	/**
	 * Home按键监听类
	 * 使用说明：
	 * 1、初始化HomeListen
	 * HomeListen homeListen = new HomeListen( this );
	 * homeListen.setOnHomeBtnPressListener( new setOnHomeBtnPressListener(){
	 * 		@Override
	 * 		public void onHomeBtnPress( ){
	 * 			// 按下Home按键回调
	 * 		}
	 * 		
	 * 		@Override
	 * 		public void onHomeBtnLongPress( ){
	 * 			// 长按Home按键回调
	 * 		}
	 * });
	 * 
	 * 2、在onResume方法中启动HomeListen广播：
	 * homeListen.start();
	 * 
	 * 3、在onPause方法中停止HomeListen广播：
	 * homeListen.stop( );
	 * */
	public class HomeListen {
		public HomeListen(Context context) {
			mContext = context;
			mHomeBtnReceiver = new HomeBtnReceiver( );
			mHomeBtnIntentFilter = new IntentFilter(Intent.ACTION_CLOSE_SYSTEM_DIALOGS);
		}
		
		public void setOnHomeBtnPressListener( OnHomeBtnPressLitener onHomeBtnPressListener ){
			mOnHomeBtnPressListener = onHomeBtnPressListener;
		}
		
		public void start( ){
			mContext.registerReceiver( mHomeBtnReceiver, mHomeBtnIntentFilter );
		}
		
		public void stop( ){
			mContext.unregisterReceiver( mHomeBtnReceiver );
		}
		
		class HomeBtnReceiver extends BroadcastReceiver{
			@Override
			public void onReceive(Context context, Intent intent) {
				receive( context, intent );
			}
		}
		
		private void receive(Context context, Intent intent){
			String action = intent.getAction();
			if (action.equals(Intent.ACTION_CLOSE_SYSTEM_DIALOGS)) {
				String reason = intent.getStringExtra( "reason" );
				if (reason != null) {
					if( null != mOnHomeBtnPressListener ){
						if( reason.equals( "homekey" ) ){
							// 按Home按键
							mOnHomeBtnPressListener.onHomeBtnPress( );
						}else if( reason.equals( "recentapps" ) ){
							// 长按Home按键
							mOnHomeBtnPressListener.onHomeBtnLongPress( );
						}
					}
				}
			}
		}
		
		public interface OnHomeBtnPressLitener{
			public void onHomeBtnPress( );
			public void onHomeBtnLongPress( );
		}
		
		private Context mContext = null;
		private IntentFilter mHomeBtnIntentFilter = null;
		private OnHomeBtnPressLitener mOnHomeBtnPressListener = null;
		private HomeBtnReceiver mHomeBtnReceiver = null;
	}

&emsp;&emsp;在Activity中做如下调用即可：

	public class HomeListenActivity extends Activity {
	    @Override  
	    protected void onCreate(Bundle savedInstanceState) {  
	        super.onCreate(savedInstanceState);  
	        setContentView(R.layout.activity_home_listen_layout);  
	        initHomeListen( );
	    }
	    
	    @Override
	    protected void onResume( ) {
	    	super.onResume();
	    	mHomeListen.start( );
	    }
	  
	    @Override  
	    protected void onPause() {  
	        super.onPause();
	        mHomeListen.stop( );
	    }
	    
	    private void initHomeListen( ){
	    	mHomeListen = new HomeListen( this );
	    	mHomeListen.setOnHomeBtnPressListener( new OnHomeBtnPressLitener( ) {
				@Override
				public void onHomeBtnPress() {
					showToast( "按下Home按键！" );
				}
				
				@Override
				public void onHomeBtnLongPress() {
					showToast( "长按Home按键！" );
				}
			});
	    }
	    
	    private void showToast( String toastInfoStr ){
	    	Toast.makeText( this, toastInfoStr, Toast.LENGTH_LONG ).show( );
	    }
	
	    private HomeListen mHomeListen = null;
	}