---
layout: post
title: Android开发经验总结
category: Android
tags: Android
keywords: Android,Android developer,Android developer experencies
description: Android开发经验总结
---

- 在Android library中不能使用switch-case语句访问资源ID：[在Android library中不能使用switch-case语句访问资源ID的原因分析及解决方案](http://link.zhihu.com/?target=http%3A//zmywly8866.github.io/2014/12/24/android-can-not-use-switch-to-load-resource-in-libproject-solution.html)

- 不能在Activity没有完全显示时显示PopupWindow和Dialog：[popupwindow - Problems creating a Popup Window in Android Activity](http://stackoverflow.com/questions/4187673/problems-creating-a-popup-window-in-android-activity)

- 在多进程之间不要用SharedPreferences共享数据，虽然可以（MODE_MULTI_PROCESS），但极不稳定：[android - MODE_MULTI_PROCESS for SharedPreferences isn't working](http://stackoverflow.com/questions/22129717/mode-multi-process-for-sharedpreferences-isnt-working)

- 有些时候不能使用Application的Context，不然会报错（比如启动Activity，显示Dialog等）：

![](http://ww4.sinaimg.cn/large/6d17e381gw1eywnfqyrzbj20go087wh4.jpg)


- 同一个应用的JNI代码，不要轻易换NDK编译的版本，否则会有很多问题（主要是一些方法实现不一样，并且高版本对代码的检测更严格），比如r8没有问题，但到r9就有问题了，这是个大坑；

- Android的JNI代码中，有返回类型的函数没有返回值编译的时候也不会报错；

- 当前Activity的onPause方法执行结束后才会执行下一个Activity的onCreate方法，所以在onPause方法中不适合做耗时较长的工作，这会影响到页面之间的跳转效率；

- 谨慎使用Android的透明主题，透明主题会导致很多问题，比如：如果新的Activity采用了透明主题，那么当前Activity的onStop方法不会被调用；在设置为透明主题的Activity界面按Home键时，可能会导致刷屏不干净的问题；进入主题为透明主题的界面会有明显的延时感；

- 不要在非UI线程中初始化ViewStub，否则会返回null；

- 公共接口一定要考虑到代码重入的情况，能设计为单例就尽量用单例；

- 不要通过Bundle传递大块的数据，否则会报TransactionTooLargeException异常：[java - Issue: Passing large data to second Activity](http://stackoverflow.com/questions/12819617/issue-passing-large-data-to-second-activity)

- 尽量不要通过Application缓存数据，这不稳定：[不要在Android的Application对象中缓存数据!](http://zmywly8866.github.io/2014/12/26/android-do-not-store-data-in-the-application-object.html)

- 尽量不要使用AnimationDrawable，它在初始化的时候就将所有图片加载到内存中，特别占内存，并且还不能释放，释放之后下次进入再次加载时会报错；
9图不能通过tinypng压缩，不然会有问题；

- genymotion模拟器快是因为它是基于x86架构的，如果你的应用中用到了so，但没有x86架构的so，只能放弃使用它；Android Studio的模拟器也一样；

- Eclipse的Android开发环境配置好后不要轻易升级ADT和build tools，不然会浪费你很多时间，还有就是一个workspace中的工程不要太多，不然每次启动都会很慢；

- Android studio每个版本、gradle每个版本差别都比较大（我是这样认为的），对于jni代码的编译建议在Eclipse中进行，如果在Android studio中开发jni会浪费很多时间，主要是编译脚本的配置比较麻烦；

- Eclipse中的Lint太不靠谱，特别是主工程中依赖library的时候，很多提示都是有问题的，建议使用Android Studio的工程清理工具，特别推荐；

- 不同API版本的AsyncTask实现不一样，有的是可以同时执行多个任务，有的API中只能同时执行一个线程，所以在程序中同时执行多个AsyncTask时有可能遇到一个AsyncTask的excute方法后很久都没有执行。[调用AsyncTask的excute方法不能立即执行程序的原因分析及改善方案](http://zmywly8866.github.io/2015/09/29/android-call-asynctask-excute-not-run.html)

- 同一个应用，相同的图片分别放在drawable-xxhdpi、drawable-xhdpi、drawable-hdpi、drawable-mdpi、drawable-ldpi中，在同一设备中占用的内存会大不一样（设备的dpi是固定的，图片放在不同的dpi文件夹下，在设备上显示时需要将图片转换成和当前屏幕一样dpi后在设备中显示，所以即使该图片在不同dpi文件夹下大小一样，但放在内存中的大小却不是一样的，并不一定是长*宽*4），做应用的内存优化之前可以先看一看你的工程是如何做屏幕适配的，是否有优化的空间。强烈推荐这个屏幕适配视频教程，花两个半小时就能看完：[Android-屏幕适配全攻略](http://link.zhihu.com/?target=http%3A//www.imooc.com/learn/484)

- 谨慎对待数据库升级（比如需要在原数据库中增加字段），避免数据丢失或者操作数据库异常的情况，数据库升级方法可以查阅《第一行代码》P263；

- 多个程序共用一套代码（一套代码，在桌面上多个图标）时需要处理好不同入口进入时的堆栈问题；

- 使用Adapter的时候，如果你使用了ViewHolder做缓存，在getView的方法中无论这项的每个视图是否需要设置属性(比如TextView设置的属性可能为null，item的某一个按钮的背景为透明、某一项的颜色为透明等)，都需要为每一项的所有视图设置属性（textview的属性为空也需要设置setText("")，背景透明也需要设置），否则在滑动的过程中会出现内容的显示错乱。

- 谨慎使用Android的多进程，多进程虽然能够降低主进程的内存压力，但会遇到如下问题：
（1）不能实现完全退出所有Activity的功能（如果有同行在应用内采用多进程成功实现过完全退出程序欢迎沟通交流）；
（2）首次进入新启动进程的页面时会有延时的现象（有可能黑屏、白屏几秒，是白屏还是黑屏和新Activity的主题有关）；
（3）应用内多进程时，新启动一个进程都会重新跑一次Application的onCreate方法，不上重新创建一个Application，但会重新跑Application的onCreate，这样就不能在Application中缓存数据作为内存共享的途径了；
（4）多进程间通过SharedPreferences共享数据时不稳定，具体可以查阅《Android开发艺术探索》。

- 使用Toast时，建议定义一个全局的Toast对象，这样可以避免连续显示Toast时不能取消上一次Toast消息的情况（如果你有连续弹出Toast的情况，避免使用Toast.makeText）；

- View的面积越大绘制的时间就越长，透明通道对View的绘制速度影响很大；

- 不要通过Msg传递大的对象，会导致内存问题；

- 关于AS的使用经验，参见：[Android Studio使用过程中需要弄明白的一些问题](http://zhuanlan.zhihu.com/zmywly8866/20375410)

- Eclipse的工程转成AS的版本后，在同一个机器中安装会报”INSTALL_FAILED_VERSION_DOWNGRADE“这个错误，原因是因为as除了可以在Manifest.xml文件中设置apk的版本名和版本号，还可以在build.gradle文件中设置apk的版本名和版本号，记得修改build.gralde中的版本名和版本号到最新就可以了；

- 通常情况下，在插入USB之后可能会跳转到一个新的界面，这时候可能你本来是横屏的，突然跳转到这个新界面是竖屏的，虽然你的界面被压在下面，但是还是会被强制横竖屏切换一次，如果这时候你的界面不做处理就会重载，如果你的界面里面有很多fragment，这时候的重载更加复杂，难以处理。所以建议不做横竖屏切换的界面都弄一下横竖屏切换不重载。

- 如果你在 manifest 中把一个 activity 设置成 android:windowSoftInputMode="adjustResize"，那么 ScrollView（或者其它可伸缩的 ViewGroups）会缩小，从而为软键盘腾出空间。但是，如果你在 activity 的主题中设置了 android:windowFullscreen="true"，那么 ScrollView 不会缩小。这是因为该属性强制 ScrollView 全屏显示。然而在主题中设置 android:fitsSystemWindows="false" 也会导致 adjustResize 不起作用；

- 做自定义手写功能时，底层上报的点并不会都在MotionEvent中能够及时接收到，比如底层一秒钟200个点，上层收到的可能只有几十个点，为了提高手写的流畅度，在onTouchEvent中，通过MotionEvent中的getHistorySize能够获取到从底层传输到上层过程中所有的点。




	















