---
layout: post
title: Java中获取文件名、类名、方法名、行号的方法
category: Java
tags: Java
keywords: Java,文件名,类名,方法名,行号
description:  Java中获取文件名、类名、方法名、行号的方法
---

&emsp;&emsp;在C语言中，可以通过宏__FILE__、__LINE__来获取文件名和行号，在Java语言中，则可以通过StackTraceElement类来获取文件名、类名、方法名、行号，具体代码如下：

	public static int getLineNumber( ){
		StackTraceElement[] stackTrace = new Throwable().getStackTrace();
		
		return stackTrace[1].getLineNumber( );
	}
	
	public static String getMethodName( ){
		StackTraceElement[] stackTrace = new Throwable().getStackTrace();
		
		return stackTrace[1].getMethodName( );
	}
	
	public static String getFileName( ){
		StackTraceElement[] stackTrace = new Throwable().getStackTrace();
		
		return stackTrace[1].getFileName( );
	}
	
	public static String getClassName( ){
        StackTraceElement[] stackTrace = new Throwable().getStackTrace();
        
        return stackTrace[1].getClassName();
	}
