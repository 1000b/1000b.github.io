---
layout: post
title: 读写文件编码方式不一致导致文件乱码的解决方案
category: Android
tags: Android
keywords: Android,加密,解密，文件操作
description: 读写文件编码方式不一致导致文件乱码的解决方案
---

&emsp;&emsp;这几天在弄一个android应用的数据加密功能，为了避免加密、解密算法被破解，我将加密和解密的核心算法用JNI封装起来，只把接口暴露给java层。

&emsp;&emsp;工作流程是这样的：

- 通过自己写的加密解密工具将数据加密；
- 将加密的数据放在android的asserts文件夹下；
- 在首次使用数据时将asserts文件夹下的数据拷贝到一个隐藏文件夹下；
- 解密隐藏文件夹下的文件。

&emsp;&emsp;在用加密工具将数据加密好了，在程序解密这个数据文件的过程中，发现解密出来的文件是原来文件大小的2倍，并且全是乱码，跟踪发现，主要问题出现在第3步，读写文件的编码方式不一致导致了文件乱码，之前我是用如下的方法来读取asserts文件夹下的内容的：

	public static String readFileFromAssets(Context context, String fileName) throws IOException {
		if (null == context || TextUtils.isEmpty( fileName )){
			return null;
		}
		
		AssetManager assetManager = context.getAssets();
		InputStream input = assetManager.open(fileName);
		ByteArrayOutputStream output = new ByteArrayOutputStream();
		byte[] buffer = new byte[1024];
		int length = 0;
		while ((length = input.read(buffer)) != -1) {
			output.write(buffer, 0, length);
		}
		output.close();
		input.close();
		
		return output.toString();
	}

&emsp;&emsp;问题就出在我将读取出来的内容转换成了字符串，ByteArrayOutputStream的toString方法将文件内容转换成utf-8的编码了，但用C语言读写文件默认都是asii编码方式，读写文件的编码方式不一致导致了乱码问题，问题找到了，解决方法也就出来了，如下：

	public static byte[] readFileFromAssets(Context context, String fileName) throws IOException {
		if (null == context || TextUtils.isEmpty( fileName )){
			return null;
		}
		
		AssetManager assetManager = context.getAssets();
		InputStream input = assetManager.open(fileName);
		ByteArrayOutputStream output = new ByteArrayOutputStream();
		byte[] buffer = new byte[1024];
		int length = 0;
		while ((length = input.read(buffer)) != -1) {
			output.write(buffer, 0, length);
		}
		output.close();
		input.close();
		
		return output.toByteArray( );
		//return output.toString();
	}

&emsp;&emsp;将读取asserts文件的内容以byte数组的形式返回就可以了。