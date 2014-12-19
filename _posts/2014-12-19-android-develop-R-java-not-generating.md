---
layout: post
title: Eclipse下Android项目不能生成R.java的解决方法汇总
category: Android
tags: Android
keywords: Eclipse,R.java，ADT
description: Eclipse下Android项目不能生成R.java的解决方法汇总
---

&emsp;&emsp;R.java是Android将每个资源文件映射成一个数值ID的一个静态资源文件类，它是编译过程中自动生成的，不允许人为编辑（虽然你强制编辑它也是可以的），如果没有它，在代码中就通过资源ID来引用资源文件就会报错。通常会有如下几种情况导致编译时无法正常生成R.java：

1. xml文件中存在错误，有的情况即使xml文件中存在错误Eclipse也是不会显式的报出来的；

2. 正在使用的android sdk中Android SDK Tool、Android SDK Platform Tool或Android SDK Build Tool不是最新（android在ADT 22中的Android SDK 构建工具增加了编译机制）；

&emsp;&emsp;多数情况是由于第1种情况导致，但我今天遇到了第二种情况（更新了ADT），还好有StackOverflow，问题速度解决了。

&emsp;&emsp;对于这个问题，有如下几个解决方案：

1. 首先检查xm文件中是否有错，没必要一个文件一个文件地检查，你只要查看控制台的**Problems**标签就可以了，从这里可以看到xml文件中的错误，修改它，clean就可以了；

2. 如果第1步不能解决，打开Android SDK Manager，找到你当前使用的SDK版本，更新对应的Android SDK Tool、Android SDK Platform Tool或Android SDK Build Tool。

**参考资料：**

- [Developing for Android in Eclipse: R.java not generating](http://stackoverflow.com/questions/2757107/developing-for-android-in-eclipse-r-java-not-generating)






