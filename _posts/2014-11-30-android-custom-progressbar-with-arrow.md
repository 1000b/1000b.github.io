---
layout: post
title: Android实现带箭头的自定义Progressbar
category: Android
tags: Android
keywords: Android,Progressbar,Custom
description: Android实现带箭头的自定义Progressbar
---

### 引言
Android原生的进度条可以根据不同的主题有不同的视觉效果，但任何一种主题下的进度条和应用程序的视觉配合起来都显得格格不入，所以多数时候我们需要自定义Progressbar，最简单的是在布局文件中通过“android:progressDrawable”为Progressbar换背景和进度图片，换图后的效果类似于这样：
![Progressbar](http://img.blog.csdn.net/20140912124731250?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZWtldXk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
但你会发现，进度图片像是被截断了一样，看上去同样不美观，所以现在很多应用都会在进度条上玩花样，做出各种各样的效果，本例介绍的是在进度条的头部加上光晕箭头的效果，最终效果类似于这样，知道如何做这个效果和，其它效果（一个动画带着进度条跑、火箭进度条等）自然而然也就会了：
![CustomProgressbar](http://img.blog.csdn.net/20140912124742375?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZWtldXk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 实现原理
实现上述自定义的Progressbar需要做两件事情:  

- 给ProgressBar换前景和背景图，这个在布局文件中定义Progressbar的时候直接设置其progressDrawableJ就可以了，eg:

		<ProgressBar
			    android:id="@+id/downloadProgressId"
				style="?android:attr/progressBarStyleHorizontal"
				android:layout_width="893.0dp"
				android:layout_height="14.0dp"
				android:layout_centerInParent="true"
				android:progressDrawable="@drawable/arrow_progress_bg"
				android:max="100"
				android:progress="0"
				/>

arrow_progress_bg.xml如下：

	<?xml version="1.0" encoding="UTF-8"?>
	<layer-list xmlns:android="http://schemas.android.com/apk/res/android" >
	    <!-- 背景 -->
	    <item
	        android:id="@android:id/background"
	        android:drawable="@drawable/arrow_progress_bar_bottom_layer"
	        />
	    <!-- 进度 -->
	    <item
	        android:id="@android:id/progress"
	        android:drawable="@drawable/arrow_progress_bar_top_layer"
	        />
	</layer-list>

- 给进度加上光晕箭头效果，本例是通过在布局文件中定义一个ImageView，在Progressbar进度改变时，通过LayoutParams时动态改变ImageView的位置。

### 核心代码：
我将上述效果封装在一个类，形成一个自定义的Progressbar,如下：  

	public class ArrowProgressBar extends RelativeLayout {
		public ArrowProgressBar(Context context, AttributeSet attrs, int defStyle) {
			super(context, attrs, defStyle);
			initArrowProgressBar( context );
		}
	
		public ArrowProgressBar(Context context, AttributeSet attrs) {
			super(context, attrs);
			initArrowProgressBar( context );
		}
	
		public ArrowProgressBar(Context context) {
			super(context);
			initArrowProgressBar( context );
		}
		
		private void initArrowProgressBar( Context context ){
			LayoutInflater layoutInflater = LayoutInflater.from( context );
			View view = layoutInflater.inflate(R.layout.arrow_progress_bar_layout, null);
			
			mProgressBar = ( ProgressBar )view.findViewById( R.id.downloadProgressId );
			mProgressTxt = ( TextView )view.findViewById( R.id.progressTxtId );
			mArrowImg = ( ImageView )view.findViewById( R.id.arrowImgId );
			mArrowImg.setVisibility( ImageView.GONE );
			
			addView( view );
		}
		
		public void setProgress( int progress ){
			if( progress < PROGRESS_MAX ){
				LayoutParams arrowParams = ( LayoutParams )mArrowImg.getLayoutParams( );
				float leftPosition = ( ( mProgressBar.getWidth( )/PROGRESS_MAX ) * ( progress - 2 ) ) + mProgressBar.getLeft();
				arrowParams.leftMargin = ( int )Math.ceil( leftPosition );
				
				mArrowImg.setLayoutParams( arrowParams );
			}else{
				mArrowImg.setVisibility( ImageView.GONE );
			}
			
			mProgressBar.setProgress( progress );
			mProgressTxt.setText( progress + "%" );
		}
		
		private ProgressBar mProgressBar = null;
		private TextView mProgressTxt = null;
		private ImageView mArrowImg = null;
		private static final float PROGRESS_MAX = 100.0f;
	}

arrow_progress_bar_layout.xml如下：

	<?xml version="1.0" encoding="utf-8"?>
	<RelativeLayout
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="fill_parent"
	    android:layout_height="fill_parent">
	    <RelativeLayout
	        android:id="@+id/progressLayoutId"
	        android:layout_width="fill_parent"
	        android:layout_height="43.0dp"
	        android:layout_centerHorizontal="true">
	        <ProgressBar
		        android:id="@+id/downloadProgressId"
		        style="?android:attr/progressBarStyleHorizontal"
		        android:layout_width="893.0dp"
		        android:layout_height="14.0dp"
		        android:layout_centerInParent="true"
		        android:progressDrawable="@drawable/arrow_progress_bg"
		        android:max="100"
		        android:progress="0"
		        />
		    <ImageView
		        android:id="@+id/arrowImgId"
		        android:layout_width="wrap_content"
		        android:layout_height="wrap_content"
		        android:background="@drawable/arrow_progress_bar_arrow"
		        android:contentDescription="@string/app_name"
		        />
	    </RelativeLayout>
	    <TextView
	        android:id="@+id/progressTxtId"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:layout_centerHorizontal="true"
	        android:gravity="center"
	        android:layout_below="@id/progressLayoutId"
	        android:layout_marginTop="10.0dp"
	        android:background="@drawable/arrow_progress_text_background"
	        android:textColor="@android:color/white"
	        android:textSize="30.0dp"
	        />
	</RelativeLayout>

### Demo程序
[Android自定义带箭头的Progressbar](http://download.csdn.net/detail/zmywly/7902515)