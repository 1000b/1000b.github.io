---
layout: post
title: Android模拟键盘输入功能的实现
category: Android
tags: Android
keywords: Android
description: Android模拟键盘输入功能的实现
---

　　在做关于输入框的操作指引时，用动态的输入效果比用静态的图片指示效果会好很多，本文结合最近需要实现的一个搜索输入操作指引的功能介绍一下android平台模拟键盘输入的实现。在android上不知道怎么录制gif的动态图，直接截图看一下效果吧，具体看demo就行啦。

![效果图](http://img.blog.csdn.net/20141112095050343?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZWtldXk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)　　

　　实现起来很简单，开一个线程，通过sleep控制输入字符的间隔时间，封装一个模拟键盘输入的方法，最终代码是这样子的：



	public class TypeInActivity extends Activity {
		@Override
		public void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_type_in_layout);
			
			showGuide( );
		}
		
		public void onClick( View v ){
			switch( v.getId( ) ){
			case R.id.searchBtnId:{
				
			}
			break;
			default:{
				
			}
			break;
			}
		}
		
		private void showGuide( ){
			new Thread( new Runnable( ) {
				@Override
				public void run() {
					try {
						Thread.sleep( 1000 );
					} catch (InterruptedException e1) {
						e1.printStackTrace();
					}
					
					// “旋转”的拼音
					int[] keyCodeArray = new int[]{KeyEvent.KEYCODE_X,KeyEvent.KEYCODE_U,KeyEvent.KEYCODE_A,KeyEvent.KEYCODE_N,KeyEvent.KEYCODE_SPACE,KeyEvent.KEYCODE_Z,KeyEvent.KEYCODE_H,KeyEvent.KEYCODE_U,KeyEvent.KEYCODE_A,KeyEvent.KEYCODE_N};
					for( int keycode : keyCodeArray ){
						try {
							typeIn( keycode );
							Thread.sleep( 200 );
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
					}
				}
			}).start( );
		}
		
		private void typeIn( final int KeyCode ){
			try {
				Instrumentation inst = new Instrumentation();
				inst.sendKeyDownUpSync( KeyCode );
			} catch (Exception e) {
				Log.e("Exception when sendKeyDownUpSync", e.toString());
			}
		}
	}


　　再找一个模拟打字的音效，在模拟输入的时候播放打字音效，效果还是可以的。。。