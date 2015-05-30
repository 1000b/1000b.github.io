---
layout: post
title: Android设置应用内文字的默认颜色和大小
category: Android
tags: Android
keywords: Android,style,Toast
description: Android设置应用内文字的默认颜色和大小
---

&emsp;&emsp;有时候我们会遇到这些的问题：

- 在不给TextView或者Button中的文字设置默认颜色时，改变Application或者Activity的主题会同时改变文字的颜色；

- 想改变Toast弹出时的文字大小，除了重写Toast似乎没有其它办法。

&emsp;&emsp;借鉴给Android动态设置主题的思想，我们可以再Application或者Activity的主题中设置文字的默认颜色和大小，达到设置应用默认文字大小和颜色的效果，代码很简单：

styles.xml文件中定义主题：

	<style name="AppTheme" parent="@android:style/Theme.Light.NoTitleBar">
        <item name="android:textSize">42dp</item>
        <item name="android:textColor">#FF0000</item>
    </style>

AndroidManifest.xml文件中引用主题：

	<application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
        <activity
            android:name=".MainActivity"
            android:label="@string/app_name" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

**注：**默认文字颜色和大小只是在没有给View设置文字大小和颜色时起作用。

	


