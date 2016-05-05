---
layout: post
title: Android通过.nomedia文件禁止多媒体库扫描指定文件夹下的多媒体文件
category: Android
tags: Android
keywords: Android,MediaStore,nomedia,媒体库
description: Android通过.nomedia文件禁止多媒体库扫描指定文件夹下的多媒体文件

---

&emsp;&emsp;Android默认情况下会将每个多媒体文件的信息保存在一个数据库中（在系统收到某些消息，比如开机、插拔SD卡、设备连接上电脑这种涉及到可能更改文件系统内容的情况下，会触发系统扫描文件系统中的多媒体文件变化情况并同步到媒体数据库中；或者[应用发送更新多媒体库广播](http://zmywly8866.github.io/2015/03/28/user-toast-scan-mediafile.html)时，也会触发多媒体数据库的更新），应用在需要读取设备内指定格式的多媒体文件信息时，可以直接读取这个数据库，相比于文件全盘检索效率会高很多。

&emsp;&emsp;但是，有时候我们并不希望某些多媒体文件被媒体库扫描到，比如：

- 应用的音效不希望被音乐播放器扫描到；

- 有些游戏的介绍视频不希望被视频播放器扫描到；

- 应用缓存的图片不希望被相册扫描到；

&emsp;&emsp;这种情况可以在不希望被保存到多媒体数据库中的文件夹下新建一个隐藏文件，文件名为".nomedia"即可。官网并没有明确介绍.nomedia文件的使用，但可以通过搜索关键词，在[Storage Options](http://developer.android.com/intl/zh-cn/guide/topics/data/data-storage.html)的页面中找到对.nomedia文件的解释，我的理解是有.nomedia文件的文件夹下的多媒体文件信息不会保存到多媒体数据库中，在系统更新媒体数据库时会视这个文件夹不见：

>	Include an empty file named .nomedia in your external files directory (note the dot prefix in the filename). This prevents media scanner from reading your media files and providing them to other apps through the MediaStore content provider. 

&emsp;&emsp;对Android多媒体库的详细介绍网上资料比较少，这篇文章介绍得比较全面，值得一读：[Android扫描多媒体文件剖析](http://droidyue.com/blog/2014/07/12/scan-media-files-in-android-chinese-edition/)
