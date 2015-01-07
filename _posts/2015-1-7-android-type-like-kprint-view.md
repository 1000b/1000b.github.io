---
layout: post
title: Android模仿打字机效果的自定义View实现
category: Android
tags: Android
keywords: Android,打字机,自定义view
description: Android模仿打字机效果的自定义View实现
---
## 前言

&emsp;&emsp;在做splash界面的时候，需要做类似于打字机打字的效果，字一个一个地蹦出来，显示每一个字都带有打字的声音。

## 效果演示

&emsp;&emsp;本例自定义View的演示效果如下（PS：一直不知道在Android上怎么录制gif格式的动画，索性在PC上跑Genymotion Android模拟器，然后用[LICEcap](http://www.appinn.com/licecap/)录屏就可以了。）。

![TypeTextView](http://ww1.sinaimg.cn/large/6d17e381gw1eo0uvwrfo1g20hj0pwgpz.gif)

## 实现原理：

&emsp;&emsp;这个其实不难实现，通过一个定时器不断调用TextView的setText就行了，在setText的时候播放打字的音效。具体代码如下：
	
	import java.util.Timer;
	import java.util.TimerTask;
	
	import android.content.Context;
	import android.media.MediaPlayer;
	import android.text.TextUtils;
	import android.util.AttributeSet;
	import android.widget.TextView;
	
	import com.uperone.typetextview.R;
	
	/**
	 * 模拟打字机效果
	 * 
	 * */
	public class TypeTextView extends TextView {
		private Context mContext = null;
		private MediaPlayer mMediaPlayer = null;
		private String mShowTextString = null;
		private Timer mTypeTimer = null;
		private OnTypeViewListener mOnTypeViewListener = null;
		private static final int TYPE_TIME_DELAY = 80;
		private int mTypeTimeDelay = TYPE_TIME_DELAY; // 打字间隔
		
		public TypeTextView(Context context, AttributeSet attrs, int defStyle) {
			super(context, attrs, defStyle);
			initTypeTextView( context );
		}
	
		public TypeTextView(Context context, AttributeSet attrs) {
			super(context, attrs);
			initTypeTextView( context );
		}
	
		public TypeTextView(Context context) {
			super(context);
			initTypeTextView( context );
		}
		
		public void setOnTypeViewListener( OnTypeViewListener onTypeViewListener ){
			mOnTypeViewListener = onTypeViewListener;
		}
		
		public void start( final String textString ){
			start( textString, TYPE_TIME_DELAY );
		}
		
		public void start( final String textString, final int typeTimeDelay ){
			if( TextUtils.isEmpty( textString ) || typeTimeDelay < 0 ){
				return;
			}
			post( new Runnable( ) {
				@Override
				public void run() {
					mShowTextString = textString;
					mTypeTimeDelay = typeTimeDelay;
					setText( "" );
					startTypeTimer( );
					if( null != mOnTypeViewListener ){
						mOnTypeViewListener.onTypeStart( );
					}
				}
			});
		}
		
		public void stop( ){
			stopTypeTimer( );
			stopAudio();
		}
		
		private void initTypeTextView( Context context ){
			mContext = context;
		}
		
		private void startTypeTimer( ){
			stopTypeTimer( );
			mTypeTimer = new Timer( );
			mTypeTimer.schedule( new TypeTimerTask(), mTypeTimeDelay );
		}
		
		private void stopTypeTimer( ){
			if( null != mTypeTimer ){
				mTypeTimer.cancel( );
				mTypeTimer = null;
			}
		}
		
		private void startAudioPlayer() {
			stopAudio();
			playAudio( R.raw.type_in );
		}
		
		private void playAudio( int audioResId ){
			try{
				stopAudio( );
	            mMediaPlayer = MediaPlayer.create( mContext, audioResId );
	            mMediaPlayer.start( );
	        }catch( Exception e ){
	            e.printStackTrace();
	        }
		}
		
		private void stopAudio( ){
			if( mMediaPlayer != null && mMediaPlayer.isPlaying( ) ){
	            mMediaPlayer.stop( );
	            mMediaPlayer.release( );
	            mMediaPlayer = null;
	        }
		}
		
		class TypeTimerTask extends TimerTask{
			@Override
			public void run() {
				post(new Runnable( ) {
					@Override
					public void run() {
						if( getText( ).toString( ).length( ) < mShowTextString.length( ) ){
							setText( mShowTextString.substring(0, getText( ).toString( ).length( ) + 1 ) );
							startAudioPlayer();
							startTypeTimer( );
						}else{
							stopTypeTimer( );
							if( null != mOnTypeViewListener ){
								mOnTypeViewListener.onTypeOver( );
							}
						}
					}
				});
			}
		}
		
		public interface OnTypeViewListener{
			public void onTypeStart( );
			public void onTypeOver( );
		}
	}

## 使用说明：

1. 在xml文件中定义：

		<com.uperone.typetext.view.TypeTextView
	        android:id="@+id/typeTxtId"
	        android:layout_width="fill_parent"
	        android:layout_height="wrap_content"
	        android:layout_centerVertical="true" />

2. 在代码中实例化：
	
		mTypeTextView = ( TypeTextView )findViewById(R.id.typeTxtId);
		mTypeTextView.setOnTypeViewListener( new OnTypeViewListener( ) {
			@Override
			public void onTypeStart() {
				print( "onTypeStart" );
			}
			
			@Override
			public void onTypeOver() {
				print( "onTypeOver" );
			}
		});	

3. 调用start方法：
		
		mTypeTextView.start( TEST_DATA );

## 下载地址

&emsp;&emsp;我将这个demo上传到github了，可以在这里下载，随便改。[TypeTextView](https://github.com/zmywly8866/TypeTextView)