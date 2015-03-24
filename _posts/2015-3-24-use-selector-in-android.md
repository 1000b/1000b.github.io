---
layout: post
title: selector的使用方法及注意事项
category: Android
tags: Android
keywords: selector
description: selector的使用方法及注意事项
---

## selector在Android中的运用

&emsp;&emsp;做过Android开发的都知道可以通过selector来改变按钮在正常、获得焦点和点击等不同状态下的显示效果，比如要实现下面这样的显示效果：

![正常状态](http://ww2.sinaimg.cn/large/6d17e381gw1eqh4uronv9j209603vweb.jpg)

![反选状态](http://ww1.sinaimg.cn/large/6d17e381gw1eqh4uy2jvwj206f038gle.jpg)

&emsp;&emsp;需要通过selector为按钮定义背景图片、按钮颜色的正反选效果：
	
- generate_data_btn_selector.xml

		<?xml version="1.0" encoding="utf-8"?>
		<selector xmlns:android="http://schemas.android.com/apk/res/android">
		    <item android:state_pressed="true"
		          android:drawable="@drawable/generate_data_btn_press" />
		    <item android:state_focused="true"
		          android:drawable="@drawable/generate_data_btn_press" />
		    <item android:drawable="@drawable/generate_data_btn_normal" />
		</selector>

- generate_data_btn_text_selector.xml

		<?xml version="1.0" encoding="utf-8"?>
		<selector xmlns:android="http://schemas.android.com/apk/res/android">
		    <item android:state_selected="true" android:color="@android:color/black" />
		    <item android:state_focused="true" android:color="@android:color/black" />
		    <item android:state_pressed="true" android:color="@android:color/black" />
		    <item android:color="@android:color/white" />
		</selector>

- 在布局文件中给Button设置背景和文字颜色即可：

		<RelativeLayout
		    xmlns:android="http://schemas.android.com/apk/res/android"
		    xmlns:tools="http://schemas.android.com/tools"
		    android:layout_width="fill_parent"
		    android:layout_height="fill_parent">
		    <Button
		        android:id="@+id/generateBtnId"
		        android:layout_width="wrap_content"
		        android:layout_height="wrap_content"
		        android:layout_centerInParent="true"
		        android:textSize="24sp"
		        android:background="@drawable/generate_data_btn_selector"
		        android:textColor="@drawable/generate_data_btn_text_selector"
		        android:text="生成数据"
		        android:onClick="onClick"
		        />
		</RelativeLayout>

## 使用selector时需要注意的地方

&emsp;&emsp;通常情况下，使用selector无非就是改变View正反选状态下的文字颜色和背景图片，但如果遇到特殊情况，比如需要改变正反选状态下文字风格、文字大小时就不能使用selector了，只能在代码里面动态设置，具体原因我也不太清楚，[StackOverflow](http://stackoverflow.com/questions/2682051/android-how-to-make-button-text-bold-when-pressed-or-focussed)上有介绍可以通过设置Style来实现，但经验证无法实现。

## 参考资料

- [StateList](http://developer.android.com/guide/topics/resources/drawable-resource.html#StateList)

- [Android selector & text color](http://stackoverflow.com/questions/1219312/android-selector-text-color)

