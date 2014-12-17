---
layout: post
title: 去掉SrollView、GrdiView、ListView、ViewPager等滑动到边缘的光晕效果
category: Android
tags: Android
keywords: SrollView、GrdiView、ListView、ViewPager、OVER_SCROLL_NEVER、setOverScrollMode
description: 去掉SrollView、GrdiView、ListView、ViewPager等滑动到边缘的光晕效果
---

&emsp;&emsp;当我们使用SrollView、GrdiView、ListView、ViewPager带有滑动功能的组件时，滑动到边缘时总会出现类似于下图的光晕效果。

![光晕效果](http://img.my.csdn.net/uploads/201412/17/1418809795_3401.jpg)

&emsp;&emsp;这是用于提示用户已经滑动到了组件的边缘，不能再滑动了，但有时候我们并不需要这个，比如在viewpager中只有一个页面时；scrollview、listview的内容刚好只有一屏时等等，这种情况可以通过view的setOverScrollMode方法来去掉光晕效果：

	setOverScrollMode( View.OVER_SCROLL_NEVER );