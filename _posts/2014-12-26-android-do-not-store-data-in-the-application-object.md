---
layout: post
title: 不要在Android的Application对象中缓存数据!
category: Android
tags: Android
keywords: Android，Application,save Data
description: 不要在Android的Application对象中缓存数据!
---
## 说明

&emsp;&emsp;这是[翻译老外的一篇文章](http://www.developerphil.com/dont-store-data-in-the-application-object/)，我之前有遇到过这个问题，并且看到有人在[Segmentfault](http://segmentfault.com/q/1010000002387994/a-1020000002402369)上问，最主要我在StackOverflow上居然没搜到累死问题，所以觉得有必要翻译过来以便后面不会再这样处理。

## 前言
&emsp;&emsp;在你的App中的很多地方都需要使用到数据信息，它可能是一个session token，一次费时计算的结果等等，通常为了避免Activity之间传递数据的开销，会将这些数据通过持久化来存储。

&emsp;&emsp;有人建议将这些数据放在Application对象中方便所有的Activity访问，这个解决方案简单、优雅并且是......完全错误的。

&emsp;&emsp;你如果你将数据缓存到Application对象中，那么有可能你的程序最终会由于一个NullPointerException异常而崩溃掉。

## 一个简单的测试程序

&emsp;&emsp;这是自定义Application的代码：

	// access modifiers omitted for brevity
	class MyApplication extends Application {
	 
	    String name;
	 
	    String getName() {
	        return name;
	    }
	 
	    void setName(String name) {
	        this.name = name;
	    }
	}

&emsp;&emsp;在第一个Activity中，我们将用户信息存储在Application对象中：

	// access modifiers omitted for brevity
	class WhatIsYourNameActivity extends Activity {
	 
	    void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.writing);
	 
	        // Just assume that in the real app we would really ask it!
	        MyApplication app = (MyApplication) getApplication();
	        app.setName("Developer Phil");
	        startActivity(new Intent(this, GreetLoudlyActivity.class));
	 
	    }
	 
	}

&emsp;&emsp;然后在第二个Activity中通过Application获取存储的用户信息：

	// access modifiers omitted for brevity
	class GreetLoudlyActivity extends Activity {
	 
	    TextView textview;
	 
	    void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	 
	        setContentView(R.layout.reading);
	        textview = (TextView) findViewById(R.id.message);
	    }
	 
	    void onResume() {
	        super.onResume();
	 
	        MyApplication app = (MyApplication) getApplication();
	        textview.setText("HELLO " + app.getName().toUpperCase());
	    }
	}

## 测试步骤

1. 打开这个APP；

2. 在WhatIsYourNameActivity中，你按要求输入用户名并将其缓存到MyApplication这个对象中；

3. 接着在GreetLoudlyActivity中，程序从MyApplication对象中取出用户名并显示出来；

4. 用户按了Home按键离开了该APP；

5. 数小时之后，系统由于内存不足（用户在体验其它APP呢，前台的任务总是优先的嘛）会在后台将你的程序杀掉；在你重新启动该APP之前一切看上去很好，但是.....；

6. 用户重新打开了这个APP；

7. Android会重新创建一个之前被Kill掉的MyApplication实例并恢复GreetLoudlyActivity；

8. GreetLoudlyActivity去获取用户名时，会因为获取的为空值报NullPointerException而崩溃掉。

## 为什么会这样？

&emsp;&emsp;在上面这个例子中，程序之所以会崩溃掉是因为恢复之后APP的Application对象是全新的，所以缓存在Application中的用户名成员变量为空值，在程序调用String的toUpperCase()方法时由于NullPointerException而崩溃掉。

&emsp;&emsp;导致这个问题的主要原因是：Application对象并不是始终在内存中的，它有可能会由于系统内存不足而被杀掉。但Android在你恢复这个应用时并不是重新开始启动这个应用，它会创建一个新的Application对象并且启动上次用户离开时的activity以造成这个app从来没有被kill掉得假象。

&emsp;&emsp;我们以为可以通过Application来缓存数据，却没想到恢复APP时直接跑了B Activity而不是先启动A Activity，最终导致的结果是程序意外的崩溃掉了。

## 有哪些替代方法可用呢？

&emsp;&emsp;对于数据缓存问题我也没有比较好的办法，但你可以按照下面其中一种方式来处理：

- 通过Intent在Activity之间来传递数据（但是请别传递大量数据，这有可能导致程序异常或者ANR）；

- 使用[官方推荐的方法](http://developer.android.com/guide/topics/data/data-storage.html)中的一种将数据持久化，存储在磁盘中；

- 在使用数据和句柄的时候做空值检测；

## 如何模拟应用程序被杀掉？

**更新：**[Daniel Lew](http://daniel-codes.blogspot.ca/)指出，最简单的方法是在DDMS中点击"Stop Porcess"杀掉你的程序，在你调试程序的时候可以这样做。

&emsp;&emsp;你可以通过模拟器或者一个Root过的真机来测试实际效果：

1. 按Home按键退出你的程序；

2. 在控制台，敲入如下命令（Windows系统下 WIN + R -> cmd -> 回车）

		# 找到该APP的进程ID
		adb shell ps
		# 找到你APP的报名
		 
		# Mac/Unix: save some time by using grep:
		adb shell ps | grep your.app.package
		 
		# 按照上述命令操作后，看起来是这样子的: 
		# USER      PID   PPID  VSIZE  RSS     WCHAN    PC         NAME
		# u0_a198   21997 160   827940 22064 ffffffff 00000000 S your.app.package
		 
		# 通过PID将你的APP杀掉
		adb shell kill -9 21997
		 
		# APP现在被杀掉啦

3. 现在在桌面长按Home按键通过后台任务管理器打开你的APP，此时系统就会重新创建一个MyApplication实例了。

## 总结

&emsp;&emsp;不要在Application对象中缓存数据化，这有可能会导致你的程序崩掉。请使用Intent在各组件之间传递数据，抑或是将数据存储在磁盘中,然后在需要的时候取出来。

&emsp;&emsp;并不仅仅只有Application对象是这样的，其它的单例或者公有静态类也有可能会由于系统内存而被杀掉，谨记。






	