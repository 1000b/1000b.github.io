---
layout: post
title: 使用Python脚本批量卸载第三方应用和清除log缓存
category: Android
tags: Android
keywords: Android,Python,批量卸载应用，清除logcat缓存
description: 使用Python脚本批量卸载第三方应用和清除log缓存
---

## 前言

&emsp;&emsp;APK默认情况下是安装在ROM里面的，但由于ROM优先，当第三方APK安装过多、没有清理机器内的log缓存时，安装程序时会报如下异常：

	[2014-04-11 20:27:04 - xxxx] Installation error: INSTALL_FAILED_INSUFFICIENT_STORAGE
	[2014-04-11 20:27:04 - xxx] Please check logcat output for more details.
	[2014-04-11 20:27:04 - xxx] Launch canceled!

&emsp;&emsp;通常有如下几种方式来解决这个问题：

- 在AndroidManifest.xml文件中增加如下属性，让APK优先安装在外置存储中：

	android:installLocation="preferExternal"

- 卸载掉无关的第三方应用；

- 清除logcat的缓存；

- 重启机器。

&emsp;&emsp;上面的四种方法只能暂时解决问题，所以我们需要将上述的操作自动化，这样在遇到这个问题时通过程序搞定，可以节省很多时间，面对这种需求我写的python脚本就应运而生了。

## 基本要求和测试环境

### 基本要求：

&emsp;&emsp;由于该脚本是通过操作DOS命令来完成的，所以在使用该脚本之前需要配置好ADB的环境变量，切确保你的设备ADB开着，并和电脑正常连接。

### 测试环境：

**操作系统：**Winows 7 64位

**Python版本：**Python 2.7

## 具体实现：

&emsp;&emsp;代码很简单，主要是通过DOS命令来操作，如果你想某些第三方应用不被卸载掉，可以通过字符串匹配相应的包名前缀，这个自己添加一下就OK：

	# -*- coding: utf-8 -*-

	import os;
	
	def uninstall( ):
	    print 'start uninstall...'
	    os.popen('adb wait-for-device');
	    packages = os.popen('adb shell pm list packages -3');
	    for package in packages.readlines():
	        packageName = package.split(':')[-1].splitlines()[0];
	        os.popen('adb uninstall ' + packageName );
	        print packageName,'uninstall successed';
	    print 'uninstall all successed...'
	
	def clearlog( ):
	    print 'start clear logcat buffer...'
	    os.popen('adb wait-for-device');
	    os.popen('adb logcat -c');
	    print 'clear logcat buffer success...'
	
	clearlog();
	uninstall();

## 参考资料：

- [Android ADB常用命令](http://segmentfault.com/a/1190000000426049)

- [pm list 常見的用法](http://imsardine.simplbug.com/note/android/adb/commands/pm.html)

