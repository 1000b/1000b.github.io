---
layout: post
title: Android性能测试工具列表
category: Android
tags: Android
keywords: Android,Android Performance,性能测试
description: Android性能测试工具列表
---

### 测试应用的启动时间

adb shell am start -W packagename/activity，eg：adb shell am start -W com.tencent.mm/.ui.LauncherUI，显示的结果中，**thisTime**和**totalTime**的含义分别为：

- **thisTime:** just current activity launched time

- **totalTime:**the activity you started may be on the bottom of activity stack. So it refers to the total time from activity searching to current activity launched. inal long thisTime = curTime - displayStartTime; final long totalTime = stack.mLaunchStartTime != 0? (curTime - stack.mLaunchStartTime) : thisTime;

### 实时显示程序的内存消耗

- 讯飞Android应用性能测试工具：[iTest](http://itest.iflytesting.com/?p=1)

- Android Studio-Android Monitor-Memory/CPU|GPU通过观测程序运行过程中的内存状态可以粗略地检测到哪些界面存在内存泄漏、哪些地方存在内存抖动（内存抖动时可能触发GC，导致程序出现卡顿的现象）、优化效果等。

### FPS查看工具

- [FpsService](https://github.com/1860yk/FpsService)，一个实时查看帧率的工具，需要集成到代码中才能使用。

### 内存泄漏查询工具

- [leakcanary](https://github.com/square/leakcanary)，这个需要集成到代码中才能正常使用，Github上也有[Eclipse的版本](https://github.com/teffy/LeakcanarySample-Eclipse)。当在操作程序的过程中有内存泄漏时会弹出内存泄漏详细的通知信息，在使用这个工具的时候程序会存在卡顿的现象，因为这个工具就是通过触发系统GC来检测哪些对象没有释放确认是否有内存泄漏的，java并没有严格意义的内存泄漏，只是某些对象持有的时间太长导致了系统的内存不能够立即释放，导致运存不足。关于Leakcanry的参考资料可以看看：[LeakCanary 中文使用说明](http://www.liaohuqiu.net/cn/posts/leak-canary-read-me/)、[LeakCanary: 让内存泄露无所遁形](http://www.liaohuqiu.net/cn/posts/leak-canary/)

### 静态代码质量检测工具

- Android Studio—>Analyze—>Inspect Code通过静态代码质量检测工具可以删掉工程中无用的资源文件、发现潜在的内存泄漏问题、明显的代码问题、简化代码等等。

### 检测应用耗时工具

- [StrictMode](http://android-performance.com/android/2014/04/24/android-strict-mode.html)

### 性能测试移动端工具

- 讯飞Android应用性能测试工具：[iTest](http://itest.iflytesting.com/?p=1)

- 腾讯开发的[GT](http://gt.tencent.com/)

- Android 5.0原生系统设置中的开发者模式，里面内置了一系列的性能测试工具，可以在程序运行的过程中测试各界面显示的效率、布局的性能问题、内存问题、ANR等问题。

### 还没有使用过的性能测试工具

- [APT](https://code.csdn.net/Tencent/apt)

- [Emmagee](https://github.com/NetEase/Emmagee)

### 性能优化的参考资料

- [一个只关注安卓性能优化以及最佳实践的Blog](http://android-performance.com/)

- [Android Performance Patterns中文版](http://hukai.me/blog/archives/)

- [Android Performance Patterns](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)

- [Best Practices for Performance](http://developer.android.com/training/best-performance.html)

- [行者无疆](http://wuche.info/)