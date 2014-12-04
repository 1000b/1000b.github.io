---
layout: post
title: Android手写开源项目和资料搜集
category: Android
tags: Android
keywords: Android，pen，write，markers，signaturepad
description: Android手写开源项目和资料搜集
---
## 引言
&emsp;&emsp;Android的手写效率一直是件头疼的事情，比如手写效率、笔锋效果、手掌抑制等等，本文搜集了关于手写的开源项目和一些相关的文章资料。

## 开源项目
- **android-signaturepad**  

**地址：**[android-signaturepad](https://github.com/zmywly8866/android-signaturepad)

**介绍：**这是一款银行手写签名的应用，通过event的getHistory方法获取存储在MotionEvent中的历史点，大大提高了手写的流畅度，通过算法实现了笔锋效果。
![signaturepad](http://img.my.csdn.net/uploads/201412/04/1417693834_9386.png)



- **markers**

**地址：**

[markers 1](https://github.com/dsandler/markers)

[markers 2](https://code.google.com/p/markers-for-android/)

**介绍：**这是一款带有笔锋效果的android手写应用，具体实现可以查看SlateView，也是通过算法实现的笔锋效果，另使用电磁笔手写时，笔锋效果更好，因为电磁笔带压感，android底层会传回真实的压力值。
![Markers](http://img.my.csdn.net/uploads/201412/04/1417694593_9993.png)



- **MultiTouchLocalResponseView**

**地址：**[MultiTouchLocalResponseView](https://github.com/zmywly8866/MultiTouchLocalResponseView)

**介绍：**实现了当整个手掌放在屏幕上时，只响应需要响应的区域，对其它区域完全无视。


## 文献资料

- [Smooth Signatures 1](http://corner.squareup.com/2010/07/smooth-signatures.html)

- [Smooth Signatures 2](http://corner.squareup.com/2012/07/smoother-signatures.html)

- [提高Android应用手写流畅度（基础篇）](http://blog.csdn.net/ekeuy/article/details/37961199)

- [Android手掌抑制功能的实现](http://blog.csdn.net/ekeuy/article/details/40828923)



