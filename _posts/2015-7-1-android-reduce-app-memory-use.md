---
layout: post
title: Android APP内存优化之图片优化
category: Android
tags: Android
keywords: Android,内存优化
description: Android APP内存优化之图片优化
---

&emsp;&emsp;网上有很多大拿分享的关于Android性能优化的文章，主要是通过各种工具分析，使用合理的技巧优化APP的体验，提升APP的流畅度，但关于内存优化的文章很少有看到。在Android设备内存动不动就上G的情况下，的确没有必要去太在意APP对Android系统内存的消耗，但在实际工作中我做的是教育类的小学APP，APP中的按钮、背景、动画变换基本上全是图片，在2K屏上（分辨率2048*1536）一张背景图片就会占用内存12M，来回切换几次内存占用就会增涨到上百兆，为了在不影响APP的视觉效果的前提下，有必要通过各种手段来降低APP对内存的消耗，下面是我在实践过程中使用的一些方法，很多都是不太成熟的项目，也不够深入，只是将其作为一种处理方式分享给大家。

&emsp;&emsp;通过DDMS的APP内存占用查看工具分析发现，APP中占用内存最多的是图片，每个Activity中图片占用内存占大半，本文重点分享对图片的内存优化。

### 不要将Button的背景设置为selector

&emsp;&emsp;在布局文件和代码中，都可以为Button设置background为selector，这样方便实现按钮的正反选效果，但实际跟踪发现，如果是将Button的背景设置为selector，在初始化Button的时候会将正反选图片都加载在内存中（具体可以查看Android源码，在类Drawable.java的createFromXmlInner方法中对图片进行解析，最终调用Drawable的inflate方法），相当于一个按钮占用了两张相同大小图片所使用的内存，如果一个界面上按钮很多或者是按钮很大，光是按钮占用的内存就会很大，可以通过在布局文件中给按钮只设置正常状态下的背景图片，然后在代码中监听按钮的点击状态，当按下按钮时为按钮设置反选效果的图片，抬起时重新设置为正常状态下的背景，具体实现方式如下：

		public class ImageButtonClickUtils {
			private ImageButtonClickUtils(){
			
			}
			
			/**
			 * 设置按钮的正反选效果
			 * 
			 * */
			public static void setClickState(View view, final int normalResId, final int pressResId){
				view.setOnTouchListener(new OnTouchListener() {
					@Override
					public boolean onTouch(View v, MotionEvent event) {
						switch(event.getAction()){
						case MotionEvent.ACTION_DOWN:{
							v.setBackgroundResource(pressResId);
						}
						break;
						case MotionEvent.ACTION_MOVE:{
							v.setBackgroundResource(pressResId);
						}
						break;
						case MotionEvent.ACTION_UP:{
							v.setBackgroundResource(normalResId);
						}
						break;
						default:{
							
						}
						break;
						}
						
						// 为了不影响监听按钮的onClick回调，返回值应为false
						return false;
					}
				});
			}
	}


&emsp;&emsp;通过上面这种方式就可以解决同一个按钮占用两倍内存的问题，如果你觉得为一个按钮提供正反选两张图片会导致APK的体积变大，可以通过如下方式实现按钮点击的反选效果，这种方式既不会存在Button占用两倍内存的情况，又减小了APK的体积（Android 5.0中的tintColor也可以实现类似的效果）：

		ImageButton personalInfoBtn = (ImageButton)findViewById(R.id.personalBtnId);
		personalInfoBtn.setOnTouchListener(new OnTouchListener() {
			@SuppressLint("ClickableViewAccessibility")
			@Override
			public boolean onTouch(View v, MotionEvent event) {
				int action = event.getAction();
				
				if(action == MotionEvent.ACTION_DOWN){
					((ImageButton)v).setColorFilter(getResources().getColor(0X50000000));
				}else if(action == MotionEvent.ACTION_UP || action == MotionEvent.ACTION_CANCEL){
					((ImageButton)v).clearColorFilter();
				}
				
				// 为了不影响监听按钮的onClick回调，返回值应为false
				return false;
			}
		});

### 将背景图片放在非UI线程绘制，提升APP的效率

&emsp;&emsp;在高分辨率的平板设备上，绘制大背景的图片会影响程序的运行效率，严重情况下就和没有开硬件加速的时候使用手写功能一样，相当地卡，最后我们的解决方案是将背景图片通过SurfaceView来绘制，这样相当于是在非UI线程绘制，不会影响到UI线程做其它事情：

	import android.content.Context;
	import android.content.res.TypedArray;
	import android.graphics.Bitmap;
	import android.graphics.BitmapFactory;
	import android.graphics.Canvas;
	import android.graphics.Matrix;
	import android.graphics.PixelFormat;
	import android.util.AttributeSet;
	import android.util.DisplayMetrics;
	import android.view.SurfaceHolder;
	import android.view.SurfaceView;
	
	import com.eebbk.hanziLearning.activity.R;
	
	public class RootSurfaceView extends SurfaceView implements SurfaceHolder.Callback, Runnable{
		private float mViewWidth = 0;
		private float mViewHeight = 0;
		private int mResourceId = 0;
		private Context mContext = null;
		private volatile boolean isRunning = false;
		private SurfaceHolder mSurfaceHolder = null;
		
		public RootSurfaceView(Context context, AttributeSet attrs, int defStyleAttr) {
			super(context, attrs, defStyleAttr);
			initRootSurfaceView(context, attrs, defStyleAttr, 0);
		}
		
		public RootSurfaceView(Context context, AttributeSet attrs) {
			super(context, attrs);
			initRootSurfaceView(context, attrs, 0, 0);
		}
		
		private void initRootSurfaceView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes){
			mContext = context;
			DisplayMetrics displayMetrics = context.getResources().getDisplayMetrics();
			TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.RootSurfaceView, defStyleAttr, defStyleRes);
			int n = a.getIndexCount();
			mViewWidth = displayMetrics.widthPixels;
			mViewHeight = displayMetrics.heightPixels;
			for(int index=0; index<n; index++){
				int attr = a.getIndex(index);
				switch(attr){
				case R.styleable.RootSurfaceView_background:{
					mResourceId = a.getResourceId(attr, 0);
				}
				break;
				case R.styleable.RootSurfaceView_view_width:{
					mViewWidth = a.getDimension(attr, displayMetrics.widthPixels);
				}
				break;
				case R.styleable.RootSurfaceView_view_height:{
					mViewHeight = a.getDimension(attr, displayMetrics.heightPixels);
				}
				break;
				default:{
					
				}
				break;
				}
			}
			a.recycle();
			mSurfaceHolder = getHolder();
			mSurfaceHolder.addCallback(this);
			mSurfaceHolder.setFormat(PixelFormat.TRANSLUCENT);
		}
	
		private Bitmap getDrawBitmap(Context context, float width, float height) {
			Bitmap bitmap = BitmapFactory.decodeResource(getResources(), mResourceId);
			Bitmap resultBitmap = zoomImage(bitmap, width, height);
			return resultBitmap;
		}
	
		@Override
		public void surfaceChanged(SurfaceHolder arg0, int arg1, int arg2, int arg3) {
			System.out.println("RootSurfaceView surfaceChanged");
		}
	
		@Override
		public void surfaceCreated(SurfaceHolder holder) {
			drawBackGround(holder);
			System.out.println("RootSurfaceView surfaceCreated");
		}
	
		@Override
		public void surfaceDestroyed(SurfaceHolder holder) {
			isRunning = false;
			System.out.println("RootSurfaceView surfaceDestroyed");
		}
		
		@Override
		protected void onAttachedToWindow() {
			super.onAttachedToWindow();
			System.out.println("RootSurfaceView onAttachedToWindow");
		}
		
		@Override
		protected void onDetachedFromWindow() {
			super.onDetachedFromWindow();
			System.out.println("RootSurfaceView onDetachedFromWindow");
		}
		
		@Override
	    public void run(){  
	        while(isRunning){  
	        	synchronized (mSurfaceHolder) { 
	        		if(!mSurfaceHolder.getSurface().isValid()){
	                	continue;
	                }
	                drawBackGround(mSurfaceHolder);
	        	}
	            isRunning = false;
	            break;
	        }  
	    }
		
		private void drawBackGround(SurfaceHolder holder) {
	        Canvas canvas = holder.lockCanvas();
	        Bitmap bitmap = getDrawBitmap(mContext, mViewWidth, mViewHeight);
	        canvas.drawBitmap(bitmap, 0, 0, null);
	        bitmap.recycle();
	        holder.unlockCanvasAndPost(canvas);
		}
		
		public static Bitmap zoomImage( Bitmap bgimage , float newWidth , float newHeight ) {
			float width = bgimage.getWidth( );
			float height = bgimage.getHeight( );
			Matrix matrix = new Matrix();
			float scaleWidth = newWidth/width;
			float scaleHeight = newHeight/height;
			matrix.postScale( scaleWidth, scaleHeight );
			Bitmap bitmap = Bitmap.createBitmap( bgimage, 0, 0, ( int ) width , ( int ) height, matrix, true );
			if( bitmap != bgimage ){
				bgimage.recycle();
				bgimage = null;
			}
			return bitmap;
		}
	}

&emsp;&emsp;在res/values/attr.xml文件中定义自定义View的自定义属性：

	<declare-styleable name="RootSurfaceView">
        <attr name="background" format="reference" />
        <attr name="view_width" format="dimension" />
        <attr name="view_height" format="dimension" />
    </declare-styleable>

### 没有必要使用硬件加速的界面建议关掉硬件加速

&emsp;&emsp;通过DDMS的heap跟踪发现，相比于关闭硬件加速，在打开硬件加速的情况下会消耗更多的内存，但有的界面打开或者关闭硬件加速对程序的运行效率并没有太大的影响，此种情况下可以考虑在AndroidManifest.xml文件中关闭掉对应Activity的硬件加速，like this：

	<!-- 设置界面 -->
    <activity
        android:name=".SettingActivity"
        android:hardwareAccelerated="false"
        android:screenOrientation="sensorLandscape"
        android:theme="@style/Translucent_NoTitle">
    </activity>

**注意：**如果使用到WebView、视频播放、手写、动画等功能时，关掉硬件加速会严重音效程序的运行效率，这种情况可以只关闭掉Activity中某些view的硬件加速，整个Activity的硬件加速不关闭。

&emsp;&emsp;如果Activity中某个View需要关闭硬件加速，但整个Activity不能关闭，可以调用view层级关闭硬件加速的方法：
	
	// view.setLayerType || 在定义view的构造方法中调用该方法
	setLayerType(View.LAYER_TYPE_SOFTWARE, null);


### 尽量少用AnimationDrawable，如果必须要可以自定义图片切换器代替AnimationDrawable

&emsp;&emsp;AnimationDrawable也是一个耗内存大户，图片帧数越多耗内存越大，具体可以查看AnimationDrawable的源码，在AnimationDrawable实例化的时候，Drawable的createFromXmlInner方法会调用AnimationDrawable的inflate方法，该方法里面有一个while循环去一次性将所有帧都读取出来，也就是在初始化的时候就将所有的帧读在内存中了，有多少张图片，它就要消耗对应大小的内存。

&emsp;&emsp;虽然可以通过如下方式释放AnimationDrawable占用的内存，但是当退出使用AnimationDrawable的界面，再次进入使用其播放动画时，会报使用已经回收了的图片的异常，这个应该是Android对图片的处理机制导致的，虽然Activity被finish掉了，但是这个Activity中使用到的图片还是在内存中，如果被回收，下次进入时就会报异常信息：
	
	/**
	 * 释放AnimationDrawable占用的内存
	 * 
	 * 
	 * */
	@SuppressWarnings("unused")
	private void freeAnimationDrawable(AnimationDrawable animationDrawable) {
		animationDrawable.stop(); 
    	for (int i = 0; i < animationDrawable.getNumberOfFrames(); ++i){
    	    Drawable frame = animationDrawable.getFrame(i);
    	    if (frame instanceof BitmapDrawable) {
    	        ((BitmapDrawable)frame).getBitmap().recycle();
    	    } 
    	    frame.setCallback(null);
    	} 
    	
    	animationDrawable.setCallback(null);
	}

&emsp;&emsp;通常情况下我会自定义一个ImageView来实现AnimationDrawable的功能，根据图片之间切换的时间间隔来定时设置ImageView的背景图片，这样始终只是一个ImageView实例，更换的只是其背景，占用内存会比AnimationDrawable小很多：

	/**
	 * 图片动态切换器
	 * 
	 * */
	public class AnimImageView {
		private static final int MSG_START = 0xf1;
		private static final int MSG_STOP  = 0xf2;
		private static final int STATE_STOP = 0xf3;
		private static final int STATE_RUNNING = 0xf4;
		
		/* 运行状态*/
		private int mState = STATE_RUNNING;
		private ImageView mImageView;
		/* 图片资源ID列表*/
		private List<Integer> mResourceIdList = null;
		/* 定时任务*/
		private Timer mTimer = null;
		private AnimTimerTask mTimeTask = null;
		/* 记录播放位置*/
		private int mFrameIndex = 0;
		/* 播放形式*/
		private boolean isLooping = false;
		
		public AnimImageView( ){
			mTimer = new Timer();
		}
		
		/**
		 * 设置动画播放资源
		 * 
		 * */
		public void setAnimation( HanziImageView imageview, List<Integer> resourceIdList ){
			mImageView = imageview;
			mResourceIdList = resourceIdList;
		}
		
		/**
		 *  开始播放动画
		 *  @param loop 时候循环播放
		 *  @param duration 动画播放时间间隔
		 * */
		public void start(boolean loop, int duration){
			stop();
			isLooping = loop;
			mFrameIndex = 0;
			mState = STATE_RUNNING;
			mTimeTask = new AnimTimerTask( );
			mTimer.schedule(mTimeTask, 0, duration);
		}
		
		/**
		 * 停止动画播放
		 * 
		 * */
		public void stop(){
			if (mTimeTask != null) {
				mFrameIndex = 0;
				mState = STATE_STOP;
				mTimer.purge();
				mTimeTask.cancel();
				mTimeTask = null;
				mImageView.setBackgroundResource(0);
			}
		}
		
		/**
		 * 定时器任务
		 * 
		 * 
		 */
		class AnimTimerTask extends TimerTask {
			@Override
			public void run() {
				if(mFrameIndex < 0 || mState == STATE_STOP){
					return;
				}
				
				if( mFrameIndex < mResourceIdList.size() ){
					Message msg = AnimHanlder.obtainMessage(MSG_START,0,0,null);
					msg.sendToTarget();
				}else{
					mFrameIndex = 0;
					if(!isLooping){
						Message msg = AnimHanlder.obtainMessage(MSG_STOP,0,0,null);
						msg.sendToTarget();
					}
				}
			}
		}
		
		private Handler AnimHanlder = new Handler(){
			 public void handleMessage(android.os.Message msg) {
		            switch (msg.what) {
		            case MSG_START:{
		            	if(mFrameIndex >=0 && mFrameIndex < mResourceIdList.size() && mState == STATE_RUNNING){
		            		mImageView.setImageResource(mResourceIdList.get(mFrameIndex));
		            		mFrameIndex++;
			            }
		            }
		            	break;
		            case MSG_STOP:{
		            	if (mTimeTask != null) {
		            		mFrameIndex = 0;
		        			mTimer.purge();
		        			mTimeTask.cancel();
		        			mState = STATE_STOP;
		        			mTimeTask = null;
		        			mImageView.setImageResource(0);
		        		}
		            }
		            	break;
		            default:
		            	break;
		            }
			 }
		};
	}

### 其它优化方式

- 尽量将Activity中的小图片和背景合并，一张小图片既浪费布局的时间，又平白地增加了内存占用；

- 不要在Activity的主题中为Activity设置默认的背景图片，这样会导致Activity占用的内存翻倍：

	<!--千万不要在主题中为Activity设置默认背景 -->
	<style name="Activity_Style" parent="@android:Theme.Holo.Light.NoActionBar">
        <item name="android:background">@drawable/***</item>
    </style>

- 对于在需要时才显示的图片或者布局，可以使用ViewStub标签，通过sdk/tools目录下的hierarchyviewer.bat查看布局文件会发现，使用viewstub标签的组件几乎不消耗布局的时间，在代码中当需要显示时再去实例化有助于提高Activity的布局效率和节省Activity消耗的内存。