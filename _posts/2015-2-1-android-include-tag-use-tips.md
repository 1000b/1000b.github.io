---
layout: post
title: Android中include标签的使用及注意事项
category: android
tags: android
keywords: android,include
description: Android中include标签的使用及注意事项
---

## 前言
&emsp;&emsp;**include**标签可以实现在一个layout中引用另一个layout的布局，这通常适合于界面布局复杂、不同界面有共用布局的APP中，比如一个APP的顶部布局、侧边栏布局、底部Tab栏布局、ListView和GridView每一项的布局等，将这些同一个APP中有多个界面用到的布局抽取出来再通过**include**标签引用，既可以降低layout的复杂度，又可以做到布局重用（布局有改动时只需要修改一个地方就可以了）。

## 使用方法
&emsp;&emsp;**include**标签的使用很简单，只需要在布局文件中需要引用其它布局的地方，使用**layout="@layout/child_layout"**就可以了：

	<include layout="@layout/titlebar" />

&emsp;&emsp;比如，**include_voice_ctrl_bar_layout.xml**是一个可以提取出来的共用布局：

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="match_parent"
	    android:layout_height="61dp"
	    android:orientation="horizontal"
	    android:layout_alignParentBottom="true">
	    <!-- 播放、暂停 -->
	    <Button
	        android:id="@+id/voiceBtnId"
	        android:layout_width="0dp"
	        android:layout_height="match_parent"
	        android:layout_weight="100"
	        android:background="@drawable/voice_btn_selector"
	        android:gravity="center"
	        android:onClick="onClick"
	        android:text="@string/voice_stop"
	        android:textColor="@android:color/white"
	        android:textSize="24sp" />
	    <!-- 分割线 -->
	    <View
	        android:layout_width="0dp"
	        android:layout_height="match_parent"
	        android:layout_weight="1"
	        android:background="@android:color/black" />
	    <!-- 听录音 -->
	    <Button
	        android:id="@+id/listenBtnId"
	        android:layout_width="0dp"
	        android:layout_height="match_parent"
	        android:layout_weight="100"
	        android:background="@drawable/listen_btn_selector"
	        android:gravity="center"
	        android:onClick="onClick"
	        android:text="@string/voice_listen"
	        android:textColor="@android:color/white"
	        android:textSize="24sp" />
	    <!-- 分割线 -->
	    <View
	        android:layout_width="0dp"
	        android:layout_height="match_parent"
	        android:layout_weight="1"
	        android:background="@android:color/black" />
	    <!-- 下一个 -->
	    <Button
	        android:id="@+id/nextBtnId"
	        android:layout_width="0dp"
	        android:layout_height="match_parent"
	        android:layout_weight="100"
	        android:background="@drawable/next_btn_selector"
	        android:gravity="center"
	        android:onClick="onClick"
	        android:text="@string/voice_next"
	        android:textColor="@android:color/white"
	        android:textSize="24sp" />
	</LinearLayout>

&emsp;&emsp;在需要使用该共用布局的地方作如下调用即可：
	
	<RelativeLayout
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:tools="http://schemas.android.com/tools"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:background="@drawable/background_bottom_layer">
	    <RelativeLayout
	        android:layout_width="match_parent"
	        android:layout_height="match_parent"
	        android:layout_margin="10dp"
	        android:background="@drawable/background_top_layer">
	        <include layout="@layout/include_voice_text_bar_layout" />
	        <include layout="@layout/include_voice_ctrl_bar_layout" />
	    </RelativeLayout>
	</RelativeLayout>

## 注意事项

- **include**和其它组件标签（RelativeLayout、LinearLayout、TextView等）一样，都可以使用layout属性来设置布局文件的宽高和位置，但需要注意的是：必须要复写**android:layout_width**和**android:layout_height**属性才能使用其它属性（比如:android:layout_grivity、android:layout_align...、android:id等），这样可以避免**include**引用layout中的子组件属性影响到**include**的布局效果：

	
		<RelativeLayout
		    xmlns:android="http://schemas.android.com/apk/res/android"
		    xmlns:tools="http://schemas.android.com/tools"
		    android:layout_width="match_parent"
		    android:layout_height="match_parent"
		    android:background="@drawable/background_bottom_layer">
		    <RelativeLayout
		        android:layout_width="match_parent"
		        android:layout_height="match_parent"
		        android:layout_margin="10dp"
		        android:background="@drawable/background_top_layer">
		        <include layout="@layout/include_voice_text_bar_layout" />
		        <include
		            android:layout_width="match_parent"
		            android:layout_height="61dp"
		            android:layout_alignParentBottom="true"
		            layout="@layout/include_voice_ctrl_bar_layout"
		            />
		    </RelativeLayout>
		</RelativeLayout>


- 建议将给**include**标签调用布局设置宽高、位置、ID等工作放在调用布局的根标签中，这样可以避免给**include**标签设置属性不当造成的各种问题（之前遇到过给**include**标签设置**android:id**属性后，程序实例化子布局中组件失败的现象）：

&emsp;&emsp;应该这样：

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
		android:id="@+id/bottomBarLayoutId"
	    android:layout_width="match_parent"
	    android:layout_height="61dp"
	    android:orientation="horizontal"
	    android:layout_alignParentBottom="true">
	    。。。
	</LinearLayout>
&emsp;&emsp;
	<include layout="@layout/include_voice_ctrl_bar_layout" />

&emsp;&emsp;而不是这样：

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:orientation="horizontal">
	    。。。
	</LinearLayout>
&emsp;&emsp;
	<include
		android:id="@+id/bottomBarLayoutId"
		android:layout_width="match_parent"
	    android:layout_height="61dp"
		android:layout_alignParentBottom="true"
		layout="@layout/include_voice_ctrl_bar_layout"
		/>
	
## 参考资料

- [Does Android XML Layout's 'include' Tag Really Work?](http://stackoverflow.com/questions/2631614/does-android-xml-layouts-include-tag-really-work)

- [Re-using Layouts with <include/>](http://developer.android.com/training/improving-layouts/reusing-layouts.html)

