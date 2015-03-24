---
layout: post
title: Android清除数据、清除缓存、一键清理的区别
category: Android
tags: Android
keywords: 清除数据，清除缓存，一键清理
description: Android清除数据、清除缓存、一键清理的区别
---

## **前言**

&emsp;&emsp;在Android设备中，我们经常会看到与系统或者应用相关的清除功能有：清除数据、清除缓存、一键清理，这么多清除功能对于一个程序猿就够难理解了，偏偏很多安卓设备上都有这些功能，对于用户来说就更难理解，趁着在把玩手机的时候想到了这一点，索引追根究底了解他们的具体区别。

![清除数据和清除缓存](http://ww1.sinaimg.cn/large/6d17e381gw1eqekmfbrokj204u082aa0.jpg)

![一键清理](http://ww3.sinaimg.cn/large/6d17e381gw1eqekn7wlflj207a02xdfo.jpg)

## **清除数据、清除缓存、一键清理的区别**

### **清除数据**

&emsp;&emsp;清除数据主要是清除用户配置，比如SharedPreferences、数据库等等，这些数据都是在程序运行过程中保存的用户配置信息，清除数据后，下次进入程序就和第一次进入程序时一样；

### **清除缓存**

&emsp;&emsp;缓存是程序运行时的临时存储空间，它可以存放从网络下载的临时图片，从用户的角度出发清除缓存对用户并没有太大的影响，但是清除缓存后用户再次使用该APP时，由于本地缓存已经被清理，所有的数据需要重新从网络上获取，注意：为了在清除缓存的时候能够正常清除与应用相关的缓存，请将缓存文件存放在getCacheDir()或者 getExternalCacheDir()路径下。比如对微信清除缓存，则聊天记录、朋友圈缓存的用户头像、图片、文字等信息都会被清除掉，清除缓存后再次进入微信时你会发现消息记录被清空了，朋友圈的图片和用户头像需要加载一会才能正常显示。

### **一键清理**

&emsp;&emsp;一键清理是系统级别的功能，它主要是杀后台进程，以达到释放内存的目的，但杀掉哪些进程和清理时设置的重要值阈值有关，重要值越大说明进程重要程度越低，如果在清理时某个进程的重要值大于该阈值，该进程就会被杀掉。比如微信等应用在后台，一件清理后会将微信和与之相关的服务都杀掉（有的服务做了特殊处理，杀不死！！！）。

## **参考资料**

- [What's the difference between clear cache & clear data in android settings](http://stackoverflow.com/questions/5744104/whats-the-difference-between-clear-cache-clear-data-in-android-settings)

- [Android中系统设置中的清除数据究竟会清除哪些数据](http://droidyue.com/blog/2014/06/15/what-will-be-removed-if-you-click-clear-data-button-in-system-application-item/)

- [Android 一键清理、内存清理功能实现](http://blog.csdn.net/whu_zhangmin/article/details/19123283)

- [How to delete files created by the application on uninstall?](http://stackoverflow.com/questions/1222269/how-to-delete-files-created-by-the-application-on-uninstall)

## **说明**

&emsp;&emsp;为了让程序被卸载后不在文件系统中留下毫无关联的无用文件，建议将应用相关的配置和缓存文件存放在程序被卸载时会删掉的文件夹下面（音乐文件、视频文件、图片、电子书这种适合多个应用阅读和浏览的文件除外），具体路径有：

- /data/data/package/

- getFilesDir()

- getCacheDir()

- getExternalCacheDir()(是否能够在程序被卸载时被删除与API的等级有关)

- getExternalFilesDir()(是否能够在程序被卸载时被删除与API的等级有关)