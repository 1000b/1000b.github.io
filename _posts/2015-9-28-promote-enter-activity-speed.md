---
layout: post
title: 提升进入界面的速度
category: Android
tags: Android
keywords: Android,AnimationDrawable,布局,onPause,onCreate
description: 提升进入界面的速度
---

&emsp;&emsp;应用除了有内存占用、内存泄露、内存抖动等看不见的性能问题外，还有很多看得见的性能问题，比如进入界面慢、点击反应慢、页面卡顿等等，这些看得见的体验问题会严重影响用户使用APP心情，但用户的情绪又无法通过异常采集、数据分析来发现，尽早优化APP的性能体验问题非常重要，会在一定程度上提升用户的留存率。

&emsp;&emsp;本文结合最近一段时间对项目中APP各界面进入速度的优化，总结一下进入界面慢的优化方案。

## 先从Activity的生命周期说起

&emsp;&emsp;从一个界面FirstActivity跳转到另外一个界面SecondActivity，两个Activity的生命周期流程是这样的：

![页面跳转生命周期的流程](http://ww3.sinaimg.cn/large/6d17e381gw1ewidzulvwgj209j04p3ze.jpg)

&emsp;&emsp;应用必须在走完FirstActivity的onPause方法后才会跑SecondActivity的onCreate方法，FirstActivity的onStop和onDestory方法不会影响到进入SecondActivity的速度。如果我们要优化从FirstActivity跳转到SecondActivity的速度，需要从FristActivity的onPause和SecondActivity的onCreate、onStart和onResume方法入手。onStart方法通常干的事情比较少，页面之间跳转慢主要是因为在FirstActivity的onPause和SecondActivity的onCreate、onResume方法耗时导致，这个过程需要执行的操作主要有：

- 保存FirstActivity界面中的一些状态；

- 加载SecondActivity的布局；

- 初始化SecondActivity。

&emsp;&emsp;针对上面的分析我们可以从如下四个方面入手：

- 耗时任务异步处理；

- 布局文件优化；

- 不可见视图需要时加载；

- 应用内慎用多进程。

## 优化实践

### 耗时任务异步处理

&emsp;&emsp;除了Android明令禁止在UI线程中执行网络操作外，还有一些耗时的操作也不能在UI线程中执行，比如IO操作、耗时较长的逻辑操作（比如算法），在Android中可以通过如下几种方式来实现异步任务：

- AsyncTask

- Thread

- Timer，TimerTask

- Handler

&emsp;&emsp;如果是在执行异步任务后需要更新界面，优先考虑使用AsyncTask和Handler，它们提供了刷新UI的方案；如果是定时任务可以考虑使用Handler和Timer,TimerTask；如果是使用Thread和Timer,TimerTask，更新UI时可以通过执行当前Activity的runOnUiThread方法实现更新UI操作。

### 布局文件优化 ViewStub

&emsp;&emsp;在优化过程中发现有的界面光是加载布局就需要500ms左右，再加上界面的初始化和上一个界面的状态保存操作，页面跳转时会有严重的迟滞感，对于布局文件的优化网上有很多有价值的文章，最重要的两条是：

- 布局文件不要嵌套太深；

- 对于不需要进入界面就需要显示的视图，强烈建议使用ViewStub。

&emsp;&emsp;布局文件嵌套太深标示着需要更多次的布局、测量和绘制，会导致耗时更多，这个可以使用android自带的“hierarchyviewer”查看，边优化边看效果；但有时候即使布局足够扁平，加载布局文件时还是会比较耗时，因为布局文件中的视图太多了，此时对于不需要进入界面就需要显示的视图，可以使用ViewStub来延迟加载，比如加载的进度条、特定状态下出现的倒计时和动画等，ViewStub的使用方式如下：

	/**
	* 在需要使用下载进度条的地方调用该方法加载下载进度条的布局
	*/
	private void initDownloadProgress() {
		if(null == mDownloadViewStub){
			mDownloadViewStub = (ViewStub)findViewById(R.id.downProgressViewStubId);
			View view = mDownloadViewStub.inflate();
			mDownloadProgressLayout = (RelativeLayout) view.findViewById(R.id.progressBackLayoutId);
			mDownLoadProgressBar = (ArrowProgressBar) view.findViewById(R.id.arrowProgressBarId);
			mDownloadProgressLayout.setVisibility(View.GONE);
			mDownloadProgressLayout.setOnTouchListener(new OnTouchListener() {
				@Override
				public boolean onTouch(View v, MotionEvent event) {
					return true;
				}
			});
		}
	}

### 不可见视图需要时加载

&emsp;&emsp;除了布局文件的优化外，代码中不需要立即显示的视图和动画都做成延迟加载，比如AnimationDrawable、TypedArray数组、Typeface、addView等，值得一提的是，初始化AnimationDrawable、TypedArray数组和Typeface会很耗时，并且AnimationDrawable特别耗内存，如果不是进入界面就需要使用，强烈建议在需要使用的地方再初始化，分开初始化可以大大减小页面初始化的耗时。

### 应用内慎用多进程

&emsp;&emsp;从FirstActivity跳转到SecondActivity，如果这两个界面不属于同一个进程，首次跳转的时候会创建一个新的进程，创建进程是比较耗时的，比跳转到同一进程内的新页面耗时更多，如果不是必须要在应用内使用多进程，强烈建议不要在应用内使用多进程。

## 总结

&emsp;&emsp;性能优化是一个持续的过程，界面跳转效率只是一个性能指标，更快地跳转对于用户来说有着更好地体验，优化界面跳转速度的步骤如下：

- 打印执行每一段代码执行需要的时间；

- 找到耗时较多的代码段，可能是setContentView，也有可能是在UI线程中的其它耗时操作；

- 根据耗时的代码段找解决办法；

- 优化后运行看效果。

**特别说明：**

- 初始化AnimationDrawable、TypedArray数组和Typeface会很耗时，并且AnimationDrawable特别耗内存，一定要注意他们的初始化时机；

- 不要迷信网上的一些优化技巧，一定要结合亲身实践，看数据说话；

- 只要认真分析，很多地方都会有优化空间，将优化的经验总结出来，并运用到后续的开发中；

- 优化APP的性能问题在一定程度上能够提高用户的留存率，是一件很有价值的事情。



