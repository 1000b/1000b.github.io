---
layout: post
title: 与Android应用程序相关的各种文件存储路径介绍
category: Android
tags: Android
keywords: Android
description: 与Android应用程序相关的各种文件存储路径介绍, getFilesDir,getCacheDir,getDir,getExternalCacheDir,getExternalFilesDir,getDatabasePath,getPackageCodePath,getPackageResourcePath,getObbDir
---

## 方法介绍：
每个Android应用程序都可以通过Context来获取与应用程序相关的目录，这些目录的功能各异，每一个目录都有自己的特点，有时候可能会搞混淆，本文结合android源码注释和实际操作，详细介绍一下每个方法：  

**方法：** getFilesDir   
**释义：** 返回通过Context.openFileOutput()创建和存储的文件系统的绝对路径，应用程序文件，这些文件会在程序被卸载的时候全部删掉。  

**方法：** getCacheDir  
**释义：** 返回应用程序指定的缓存目录，这些文件在设备内存不足时会优先被删除掉，所以存放在这里的文件是没有任何保障的，可能会随时丢掉。  

**方法：** getDir  
**释义：** 这是一个可以存放你自己应用程序自定义的文件，你可以通过该方法返回的File实例来创建或者访问这个目录，注意该目录下的文件只有你自己的程序可以访问。  

**方法：** getExternalCacheDir  
**释义：** 使用这个方法需要写外部存储的权限“<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />”，调用该方法会返回应用程序的外部文件系统（Environment.getExternalStorageDirectory()）目录的绝对路径，它是用来存放应用的缓存文件，它和getCacheDir目录一样，目录下的文件都会在程序被卸载的时候被清除掉。   

**方法：** getExternalFilesDir  
**释义：** 使用这个方法需要写外部存储的权限“<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />”，这个目录是与应用程序相关的外部文件系统，它和getExternalCacheDir不一样的是只要应用程序存在它就会一直存在，这些文件只属于你的应用，不能被其它人访问。同样，这个目录下的文件在程序被卸载时也会被一同删除。  

**方法：** getExternalFilesDir  
**释义：** 和上面的方法一样，只是返回的是其目录下某一类型的文件，这些类型可以是：  
     Environment#DIRECTORY_MUSIC 音乐
     Environment#DIRECTORY_PODCASTS 音频
     Environment#DIRECTORY_RINGTONES 铃声
     Environment#DIRECTORY_ALARMS 闹铃
     Environment#DIRECTORY_NOTIFICATIONS 通知铃声
       Environment#DIRECTORY_PICTURES 图片
     Environment#DIRECTORY_MOVIES 视频

**方法：** getDatabasePath  
**释义：** 保存通过Context.openOrCreateDatabase 创建的数据库文件  

**方法：** getPackageCodePath  
**释义：** 返回android 安装包的完整路径，这个包是一个zip的压缩文件，它包括应用程序的代码和assets文件。  

**方法：** getPackageResourcePath  
**释义：** 返回android 安装包的完整路径，这个包是一个ZIP的要锁文件，它包括应用程序的私有资源。  

**方法：** getObbDir  
**释义：** 返回应用程序的OBB文件目录（如果有的话），注意如果该应用程序没有任何OBB文件，这个目录是不存在的。  

## 测试程序：  

### 测试代码如下：  
```

	private StringBuilder getFilePath( ){
		StringBuilder filePathBuilder = new StringBuilder( );
		
		// 返回通过Context.openFileOutput()创建和存储的文件系统的绝对路径，应用程序文件，这些文件会在程序被卸载的时候全部删掉。
		filePathBuilder.append( "getFilesDir == " ).append( getFilesDir( ) ).append( "\n" );
		// 返回应用程序指定的缓存目录，这些文件在设备内存不足时会优先被删除掉，所以存放在这里的文件是没有任何保障的，可能会随时丢掉。
		filePathBuilder.append( "getCacheDir == " ).append( getCacheDir( ) ).append( "\n" );
		// 这是一个可以存放你自己应用程序自定义的文件，你可以通过该方法返回的File实例来创建或者访问这个目录，注意该目录下的文件只有你自己的程序可以访问。
		filePathBuilder.append( "getDir == " ).append( getDir("test.txt", Context.MODE_WORLD_WRITEABLE) ).append( "\n" );
		
		/* 需要写文件权限 <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" /> */
		// 调用该方法会返回应用程序的外部文件系统（Environment.getExternalStorageDirectory()）目录的绝对路径，它是用来存放应用的缓存文件，它和getCacheDir目录一样，目录下的文件都会在程序被卸载的时候被清除掉。 
		filePathBuilder.append( "getExternalCacheDir == " ).append( getExternalCacheDir( ) ).append( "\n" );
		// 这个目录是与应用程序相关的外部文件系统，它和getExternalCacheDir不一样的是只要应用程序存在它就会一直存在，这些文件只属于你的应用，不能被其它人访问。同样，这个目录下的文件在程序被卸载时也会被一同删除。
		filePathBuilder.append( "getExternalFilesDir == " ).append( getExternalFilesDir( "/" ) ).append( "\n" );
		
		/**
		 * 和上面的方法一样，只是返回的是其目录下某一类型的文件，这些类型可以是：
		 * Environment#DIRECTORY_MUSIC 音乐
	     * Environment#DIRECTORY_PODCASTS 音频
	     * Environment#DIRECTORY_RINGTONES 铃声
	     * Environment#DIRECTORY_ALARMS 闹铃
	     * Environment#DIRECTORY_NOTIFICATIONS 通知铃声
 	     * Environment#DIRECTORY_PICTURES 图片
	     * Environment#DIRECTORY_MOVIES 视频
		 * 
		 * */
		filePathBuilder.append( "getExternalFilesDir == " ).append( getExternalFilesDir( Environment.DIRECTORY_PICTURES ) ).append( "\n" );
		
		// 保存通过Context.openOrCreateDatabase 创建的数据库文件
		filePathBuilder.append( "getDatabasePath == " ).append( getDatabasePath( DATA_BASE_NAME ) ).append( "\n" );
		// 返回android 安装包的完整路径，这个包是一个zip的压缩文件，它包括应用程序的代码和assets文件
		filePathBuilder.append( "getPackageCodePath == " ).append( getPackageCodePath( ) ).append( "\n" );
		// 返回android 安装包的完整路径，这个包是一个ZIP的要锁文件，它包括应用程序的私有资源。
		filePathBuilder.append( "getPackageResourcePath == " ).append( getPackageResourcePath( ) ).append( "\n" );
		// 返回应用程序的OBB文件目录（如果有的话），注意如果该应用程序没有任何OBB文件，这个目录是不存在的。
		filePathBuilder.append( "getObbDir == " ).append( getObbDir( ) ).append( "\n" );
		
		return filePathBuilder;
	}

```  

### 测试效果如下：
![效果图](http://img.blog.csdn.net/20140928191557031?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZWtldXk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)