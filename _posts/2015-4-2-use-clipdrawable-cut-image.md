---
layout: post
title: Android通过ClipDrawable实现图片裁剪功能
category: Android
tags: Android
keywords: Android,ClipDrawable,图片裁剪
description: Android通过ClipDrawable实现图片裁剪功能
---
## 前言

&emsp;&emsp;最近需要实现一个显示下载进度的功能，下载进度的实现很简单，用ProgressBar就可以，但我想尝试通过图片裁剪来实现，学习一下ClipDrawable这个类的使用。

## ClipDrawable简介

&emsp;&emsp;ClipDrawable是通过设置一个Drawable的当前显示比例来裁剪出另一张Drawable，你可以通过调节这个比例来控制裁剪的宽高，以及裁剪内容占整个容器的权重，通过ClipDrawable的setLevel()方法调节显示比例可以实现类似Progress进度条的效果。ClipDrawable的level值范围在[0,10000]，level的值越大裁剪的内容越少，如果level为10000时则完全显示。

## ClipDrawable的使用方法

1. 在工程的res/drawable文件夹下新建一个文件名为"download_data_drawable_clip.xml"的xml文件，里面包含如下信息，clip元素中只有android:drawable、android:clipOrientation和android:gravity三个属性，其中android:drawable为需要裁剪的原始图片，android:clipOrientation为裁剪的方向，可以按照垂直(vertical)或者水平(horizontal)方向进行裁剪,android:gravity为指定从哪里开始裁剪，这个可以通过或操作设置多个属性。

		<?xml version="1.0" encoding="utf-8"?>
		<clip
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    android:drawable="@drawable/drawable_resource"
	    android:clipOrientation=["horizontal" | "vertical"]
	    android:gravity=["top" | "bottom" | "left" | "right" | "center_vertical" |
	                     "fill_vertical" | "center_horizontal" | "fill_horizontal" |
	                     "center" | "fill" | "clip_vertical" | "clip_horizontal"] />
	

2. 在布局文件中定义一个视图，将1中新建的xml文件作为该视图的背景：

		<ImageView
	        android:id="@+id/downloadProgressId"
	        android:layout_width="match_parent"
	        android:layout_height="match_parent"
	        android:src="@drawable/download_data_drawable_clip"
	        android:contentDescription="@string/app_name"
	        />

3. 在程序中通过如下代码实现对该视图的裁剪：

		private ImageView mDownloadProgressImg = null;

		private void initDownloadProgressImg(){
			mDownloadProgressImg = (ImageView)findViewById(R.id.downloadProgressId);
			mDownloadProgressDrawable = (ClipDrawable)mDownloadProgressImg.getDrawable();
			mDownloadProgressDrawable.setLevel(0);
		}
		
		private void updateDownloadProgress(int progress){
			// 下载进度为百分制，但level的范围为[0,10000]，所以这里需要*100
			mDownloadProgressDrawable.setLevel(progress*100);
		}
**注意：**在给clip元素中android:drawable属性设置背景图片时，图片不能是9图，因为这涉及到裁剪这张图片，如果设置为九图，裁剪的实际情况会与想要的效果不一样。

## More

- [Clip Drawable](http://developer.android.com/guide/topics/resources/drawable-resource.html#Clip)

- [(Android)How can I show a part of image?](http://stackoverflow.com/questions/18073588/androidhow-can-i-show-a-part-of-image)

- [Android Drawable Resource学习（九）、ClipDrawable](http://blog.csdn.net/lonelyroamer/article/details/8244777)
	