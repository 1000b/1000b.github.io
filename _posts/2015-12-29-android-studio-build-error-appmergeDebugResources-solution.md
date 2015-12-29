---
layout: post
title: Android Studio编译过程中mergeDebugResources时报“png-cruncher_*”异常的解决方案
category: Android
tags: Android
keywords: Android Studio，mergeDebugResources，png-cruncher_，SLAVE_AAPT_TIMEOUT
description: Android Studio编译过程中mergeDebugResources时报“png-cruncher_*”异常的解决方案
---

&emsp;&emsp;这两天遇到了一个AS编译过程中报Exception的问题：
	
		...//省略
		:app:mergeDebugResources
		Exception in thread "png-cruncher_5" java.lang.RuntimeException: Timed out while waiting for slave aapt process, try setting environment variable SLAVE_AAPT_TIMEOUT to a value bigger than 5 seconds
		    at com.android.builder.png.AaptProcess.waitForReady(AaptProcess.java:104)
		    at com.android.builder.png.QueuedCruncher$1.creation(QueuedCruncher.java:107)
		    at com.android.builder.tasks.WorkQueue.run(WorkQueue.java:206)
		    at java.lang.Thread.run(Thread.java:745)
		Exception in thread "png-cruncher_10" java.lang.RuntimeException: Timed out while waiting for slave aapt process, try setting environment variable SLAVE_AAPT_TIMEOUT to a value bigger than 5 seconds
		    at com.android.builder.png.AaptProcess.waitForReady(AaptProcess.java:104)
		    at com.android.builder.png.QueuedCruncher$1.creation(QueuedCruncher.java:107)
		    at com.android.builder.tasks.WorkQueue.run(WorkQueue.java:206)
		    at java.lang.Thread.run(Thread.java:745)
		...//省略

&emsp;&emsp;解决这个问题花了我一天时间，网上根本找不到解决方案，按照网上提示的设置环境变量“SLAVE_AAPT_TIMEOUT”，延长超时的时间、更改build-tools的版本、更新build-tools、升级AS、关掉杀毒软件都不管用。

&emsp;&emsp;最后只好结合编译时的日志进行分析，肯定是在编译时打包资源时出了问题，这就涉及到aapt打包资源的问题，所以最终还是怀疑到了build-tools上面，因为aapt.exe这个文件就在每个版本的build-tools文件夹下，但之前更新过build-tools、更改过build-tools的版本并不管用，再结合网上提示有可能是杀毒软件的问题，想到是不是杀毒软件将build-tools中的文件标记位病毒导致的，最后找同事要了一份他电脑上的build-tools，对比发现里面的很多文件内容的确不同，拷贝过来，重新编译，成功！！！

&emsp;&emsp;这是一个很简单的问题，可解决这个问题的方式出了问题，导致浪费了大把时间，如果早先就自己分析，而不是直接在网上去找答案，可能解决起来会更快。但也不尽然，对于这种问题排除法、调试、二分法这些都不管用，只能一个一个地去试，并且根据自己的经验分析判断才能够快速解决。