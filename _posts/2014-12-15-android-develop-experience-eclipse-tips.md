---
layout: post
title: Android开发经验谈-Eclipse使用技巧
category: Android
tags: Android
keywords: Android开发经验，Eclipse技巧，快捷键，主题，行号。
description: Android开发经验谈-Eclipse使用技巧
---

&emsp;&emsp;Eclipse什么都好，就是启动太慢，特别是当你的电脑配置较低或者工作空间里面有太多工程的时候，我之前经常遇到打开eclipse半个小时左右，窗口右下角仍然显示“Android sdk content loader 0%”，我想很多朋友都会遇到这个问题，通常只有重启eclipse才能解决。但如果避开这个问题不谈（你可以升级自己的电脑嘛，我现在8G的内存再也不会遇到这个问题了），eclipse真的非常强大，我们在开发的时候每天都在用到它，掌握一些使用技巧可以大大提高工作效率，现在我将我经常用到的一些技巧汇总如下。

- “Android sdk content loader 0%”的解决方案：

![Android sdk content loader 0%](http://img.my.csdn.net/uploads/201412/15/1418629003_9924.jpg)

- 设置主题：Window -> Preferences -> General -> Appearance -> Color Theme -> 在右侧选择主题，根据预览效果选择符合自己要求的主题。

![设置主题](http://img.my.csdn.net/uploads/201412/15/1418626635_7015.jpg)


- 设置字体：Window -> Preferences -> Genernal -> Color and Fonts -> Basic -> Text Font -> 选择字体、字号，点击确定之后才能预览。

![设置字体](http://img.my.csdn.net/uploads/201412/15/1418626665_3897.jpg)


- 显示行号：

(1)

![显示行号](http://img.my.csdn.net/uploads/201412/15/1418626666_2222.jpg)


（2）打开*.java或者*.c，然后在显示区域左边栏右键，选择"show line numbers"。

![显示行号](http://img.my.csdn.net/uploads/201412/15/1418626942_8227.jpg)


- 保存log信息：

![保存Log](http://img.my.csdn.net/uploads/201412/15/1418626633_4409.jpg)


- 过滤log信息：

	参见[Android开发：一分钟学会使用Logcat调试程序](http://blog.csdn.net/yanzi1225627/article/details/8577332)和[android Logcat怎么让它只显示我自己的应用程序信息](http://www.dewen.io/q/6365/android+Logcat%E6%80%8E%E4%B9%88%E8%AE%A9%E5%AE%83%E5%8F%AA%E6%98%BE%E7%A4%BA%E6%88%91%E8%87%AA%E5%B7%B1%E7%9A%84%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E4%BF%A1%E6%81%AF)

- 通过Working Sets实现工程分类：

![切换到Working Sets](http://img.my.csdn.net/uploads/201412/15/1418626682_4452.jpg)


- 正确创建xml文件的方法：

![创建Xml](http://img.my.csdn.net/uploads/201412/15/1418626634_9621.jpg)


- Eclipse快捷键：
	
	删除某一行代码：CTRL+D；

	调出Outline：CTRL+O；

	运行工程：CTRL+F11；

	导入和清理引用的包：CTRL+SHIFT+O；

	跳转到某一行：CTRL+L；

	查找关联信息：CTRL+K；

	代码格式化：选中要格式化的代码，按CTRL+SHIFT+F；

	将代码块提取为方法：ALT + SHIFT + M；

	更多实用功能，直接按：ALT+SHIFT+S；

	[Eclipse 常用快捷键 (动画讲解)](http://www.cnblogs.com/TankXiao/p/4018219.html)


- 调出各种视图：

![调出视图](http://img.my.csdn.net/uploads/201412/15/1418626665_9146.jpg)


- 导入Android源码：

	[Android 如何在Eclipse中查看Android API源码以及support包源码](http://blog.csdn.net/xiaanming/article/details/9031141)


- 截图：

![截图](http://img.my.csdn.net/uploads/201412/15/1418626635_4073.jpg)


- 修改adb连接超时时长：Window -> Preferences -> Android -> DDMS -> ADB Connection time out(ms)

![设置ADB超时时长](http://img.my.csdn.net/uploads/201412/15/1418626665_9496.jpg)


- logcat 行数修改： Window -> Preferences -> Android -> Logcat -> Maximum number of logcat message to buffer

![修改logcat行数](http://img.my.csdn.net/uploads/201412/15/1418626635_1523.jpg)



