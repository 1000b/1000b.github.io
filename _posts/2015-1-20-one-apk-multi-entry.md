---
layout: post
title: Android一个APK多个入口（多个桌面图标）的实现
category: Android
tags: Android
keywords: 多入口，多个桌面图标，一个APK多个入口
description: Android一个APK多个入口（多个桌面图标）的实现
---

## 前言

&emsp;&emsp;Android应用一般都是一个APK一个桌面图标，但有时候我们需要实现一个APK在桌面上有多个图标（比如BAT的某些应用，有桌面快捷方式），对于这种一个APK需要在桌面上显示多个图标的，通常有两种方法来实现：

- 进入程序后生成桌面快捷方式，这个适合于在程序运行之后生成桌面图标；

- 在AndroidManifest.xml文件中配置多个入口，这个适合于程序安装完成后就在桌面上显示多个图标。

&emsp;&emsp;在桌面生成快捷方式，具体实现方式可参见：

- [总结：android 创建快捷方式的两种方式+判断是否已经创建+删除快捷方式](http://blog.csdn.net/panda1234lee/article/details/9042873)

- [Android create shortcuts on the home screen](http://stackoverflow.com/questions/6337431/android-create-shortcuts-on-the-home-screen)

&emsp;&emsp;本文介绍第二种方式，在程序安装完成后就会在桌面显示多个图标，这种方式适合于多个模块功能一样，但里面内容不一样（比如教材数据）的情况。


## 具体实现
&emsp;&emsp;在Android应用程序中，我们是通过给Activity标签中加入下面的intent-filter来指定程序的入口的，如果一个APK要有多个入口，自然而然地想到AndroidManifest.xml文件中会存在多个包含如下标签的Activity，所以我们需要做的仅仅是如何区分每一个桌面图标对应哪一个入口。

	<intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>

&emsp;&emsp;Activity有一个重要的属性process，这个属性是指定Activity运行时所在的进程。没有指定此属性的话，所有程序组件运行在应用程序默认的进程中，这个进程名跟应用程序的包名一致。在AndroidManifest.xml文件中所有组件元素的process属性能够为该组件设定一个新的默认值。但是任何组件都可以覆盖这个默认值，允许你将你的程序放在多进程中运行。如果这个属性被分配的名字以:开头， 当这个Activity运行时, 一个新的专属于这个程序的进程将会被创建。所以可以通过给每一个Activity指定标签、图标和进程名来区分不同的入口，具体实现如下：

        <activity
            android:name=".PreSchoolChildActivity"
            android:label="@string/pre_school_child_app_name"
            android:process=":process.main"
            android:icon="@drawable/preschoolchild_launcher" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        
        <activity
            android:name=".EnglishSpellActivity"
            android:label="@string/english_spell_app_name"
            android:process=":process.sub"
            android:launchMode ="singleInstance"
            android:icon="@drawable/englishspell_launcher" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

&emsp;&emsp;需要注意的是，为Activity指定process属性后，还必须为其指定launchMode为singleInstance，这样才有效。除了为Activity指定android:process属性外，还可以通过为Activity指定别名的方式实现同样的效果，具体参见：[activity-alias](http://developer.android.com/guide/topics/manifest/activity-alias-element.html)。

## 存在的问题

- 因为多个图标共用一个包名，所以只要卸载一个程序，与这个APK包名相同的程序也都会在桌面上消失；

- 从其它应用跳转到该APK时，需要通过ACTION区分跳转到具体哪一个模块（比如从资源管理器选择一个数据时，到底打开哪一个应用，需要通过action加以区分）；

- 由于多个图标、splash、标题等需要区分的资源都放在一个APK中，这无形之中增加了APK的大小。
