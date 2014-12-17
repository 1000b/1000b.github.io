---
layout: post
title: android:descendantFocusability属性在ListView中的妙用
category: Android
tags: Android
keywords: descendantFocusability、焦点
description: android:descendantFocusability属性在ListView中的妙用
---

&emsp;&emsp;在使用ListView时，通常都需要我们自己去定义Adapter来满足开发中的个性化需求，比如每一项中有Button、CheckBox、RadioButton、TextView等组件时，显然android.jar自带的BaseAdapter无法满足我们的需求。这时我们通常会遇到listview的每一项无法响应点击的问题，因为消息还没传回每一项的viewgroup就被其子view消费了，android的消息传递机制可以参考：[Mastering the Android Touch System](https://thenewcircle.com/s/post/1567/mastering_the_android_touch_system),里面讲得非常详细。我们可以通过为该ViewGroup设置“android:descendantFocusability”属性来强制获取焦点，以便能够消费android系统传递过来的消息。“android:descendantFocusability”的详细解释如下图所示：

![descendantFocusability](http://img.my.csdn.net/uploads/201412/17/1418815517_9002.jpg)

**大意是：**

&emsp;&emsp;该属性是当一个为view获取焦点时，定义viewGroup和其子控件两者之间的关系。

&emsp;&emsp;属性的值有三种：

- **beforeDescendants：**viewgroup会优先其子类控件而获取到焦点

- **afterDescendants：**viewgroup只有当其子类控件不需要获取焦点时才获取焦点

- **blocksDescendants：**viewgroup会覆盖子类控件而直接获得焦点

&emsp;&emsp;所以我们只需要在每一项Item布局的根布局加上android:descendantFocusability=”blocksDescendants”的属性就好了

**参考资料：**

- [android:descendantFocusability用法简析](http://www.cnblogs.com/eyu8874521/archive/2012/10/17/2727882.html)