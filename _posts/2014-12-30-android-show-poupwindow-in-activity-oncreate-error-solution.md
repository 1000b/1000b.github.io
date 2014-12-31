---
layout: post
title: 在Activity的onCreate方法中显示PopupWindow导致异常的原因分析及解决方案
category: Android
tags: Android
keywords: Android，token is null,popupwindow,onCreate
description: 在Activity的onCreate方法中显示PopupWindow导致异常的原因分析及解决方案
---

## 前言
&emsp;&emsp;在某些情况下，我们需要一进入Activity就显示PopupWindow，比如常见的选择界面。但由于PopupWindow是依附于Activity的，如果Activity没有创建完成，Activity还没完全显示出来就显示PopupWindow的话，会出现异常现象。

## 问题复现
&emsp;&emsp;我在Activity的onCreate()方法中调用如下方法：

	public void show( ){
		if( null != mPopupWindow ){
			mPopupWindow.showAtLocation(mView, Gravity.CENTER, 0, 0);
		}
	}

&emsp;&emsp;运行程序的时候出现如下异常：

![token is null](http://img.my.csdn.net/uploads/201412/30/1419942066_6251.jpg)

## 解决方案

&emsp;&emsp;在StackOverflow上搜索这个问题你会发现，都没有原因分析，但我在《Android开发精要》一书中找到了答案（P158）：

> &emsp;&emsp;PopupWindow不像对话框那样从屏幕的固定位置弹出，而是依赖于锚点控件对象的位置，所谓锚点控件对象，就是界面组件中的某个控件，PopupWindow的展示和功能都是以它为核心，作为锚点控件的扩展交互界面，以增强该控件对象的功能。
> 
> &emsp;&emsp;弹出窗口与描点控件有着紧密的联系，在构造并展示弹出窗口前，需要保证锚点控件所在的控件树已经与窗口管理服务建立连接，因为在弹出窗口的展示过程中，需要通过该窗口对象来获取相关信息。在界面组件的构造过程中，窗口连接的建立是个异步过程，也就是说，当Activity.onCreate()等函数被调用时，界面与窗口管理服务的双向通信连接尚未建立，如果在此时构造弹出窗口则会抛出异常。因此，如果期望在界面组件展现之处便构造弹出窗口，可以将弹出窗口对象构造也转换成一个异步过程。

>     // 在界面组件onCreate函数中调用
>     final View anchor = findViewById(R.id.anchor);
>     anchor.post(new Runnable(){
>        @Override
>       public void run(){
>          // 构造和展现弹出窗口
>          PopupWindow window = createWindow();
>          window.showAsDropDown(anchor);
>       }
>     });
>     
> &emsp;&emsp;在与窗口管理服务未建立连接之前，界面组件将通过View.post函数发送过来的消息放入一个静态队列当中，在通信连接建立完成后，再从该队列中读取消息并一一执行。因此，通过这样的实现模型可以保证弹出窗口展现时窗口通信连接已经构建成功。

&emsp;&emsp;所以对于上面的问题，最简单的处理方法是，异步显示PopupWindow就好了：

	public void show( ){
		mView.post( new Runnable( ) {
			@Override
			public void run() {
				if( null != mPopupWindow ){
					mPopupWindow.showAtLocation(mView, Gravity.CENTER, 0, 0);
				}
			}
		});
	}