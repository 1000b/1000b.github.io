---
layout: post
title: 解决JPinyin在APK被加密后不能正常使用的问题
category: Android
tags: Android
keywords: JPinyin，汉字转拼音
description: 解决JPinyin在APK被加密后不能正常使用的问题
---

&emsp;&emsp;之前写过一篇博客[汉字转拼音开源工具包Jpinyin介绍](http://blog.csdn.net/ekeuy/article/details/40079475)，介绍过JPinyin的使用，因为它实在是太方便了，在项目一直用它，但是最近在做项目的时候，发现使用了JPinyin的工程，在IDE中编译的APK能正常使用，但是APK被加密后再安装，使用JPinyin时会报汉字转拼音时读取数据失败，下面记录一下问题的解决过程。

&emsp;&emsp;加密后导致使用Jpinyin报异常的原因肯定是加密导致了Jpinyin中的某些内容变化导致（解密后数据不再是未加密前的数据了），为了跟踪这个问题，下载[JPinyin](https://github.com/stuxuhai/jpinyin)的源码发现汉字转拼音是使用了三个“数据库”文件，分别是：

- chinese.db

- mutil_pinyin.db

- pinyin.db

&emsp;&emsp;起初我以为是数据被加密了，加密APK的时候相当于这些数据被再次加密导致了JPinyin不能正常使用，但是JPinyin中的代码是这样读取这三个文件的，读取文件的类为：PinyinResource.java

	package com.github.stuxuhai.jpinyin;

	import java.io.IOException;
	import java.io.InputStream;
	import java.util.Properties;
	
	/**
	 * 资源文件加载类
	 *
	 * @author stuxuhai (dczxxuhai@gmail.com)
	 * @version 1.0
	 */
	public class PinyinResource {
	
		private static Properties getResource(String resourceName) {
			InputStream is = PinyinResource.class.getResourceAsStream(resourceName);
			Properties props = new Properties();
			try {
				props.load(is);
			} catch (IOException e) {
				throw new RuntimeException(e);
			} finally {
				try {
					is.close();
				} catch (IOException e) {
					throw new RuntimeException(e);
				}
			}
			return props;
		}
	
	
		protected static Properties getPinyinTable() {
			String resourceName = "/data/pinyin.db";
			return getResource(resourceName);
		}
	
	
		protected static Properties getMutilPintinTable() {
			String resourceName = "/data/mutil_pinyin.db";
			return getResource(resourceName);
		}
	
	
		protected static Properties getChineseTable() {
			String resourceName = "/data/chinese.db";
			return getResource(resourceName);
		}
	}

&emsp;&emsp;文件读取过程中根本没有数据库的解密过程，代码中将db文件当做“.properties”文件处理了，遂在网上查询".properties"为何物：[JAVA操作properties文件](http://www.cnblogs.com/panjun-donet/archive/2009/07/17/1525597.html)，直接把这三个db文件的后缀改成".properties"，然后打开的内容是这样的：

![chinese.properties](http://ww1.sinaimg.cn/large/6d17e381gw1eu1dkf63vpj207p0kwjun.jpg)

&emsp;&emsp;于是可以确定这三个文件的原始格式为".properties"，只是作者将其改为".db"文件了，于是尝试通过将db格式改为.properties,来解决加密APK导致JPinyin不能正常使用的问题，修改后的代码是这样的（这个工程是在Android项目中被使用的，所以我将这三个.properties文件放在了assets文件夹下）：

	/**
	 * 资源文件加载类
	 *
	 * @author stuxuhai (dczxxuhai@gmail.com)
	 * @version 1.0
	 */
	public class PinyinResource {
		private static Properties getResource(Context context, String resourceName) {
			InputStream is = null;
			Properties props = null;
			try {
				is = context.getAssets().open(resourceName);
				props = new Properties();
				props.load(is);
			} catch (IOException e) {
				throw new RuntimeException(e);
			} finally {
				try {
					if(null != is){
						is.close();
					}
				} catch (IOException e) {
					throw new RuntimeException(e);
				}
			}
			return props;
		}
	
		protected static Properties getPinyinTable(Context context) {
			String resourceName = "pinyin.properties";
			return getResource(context, resourceName);
		}
	
		protected static Properties getMutilPintinTable(Context context) {
			String resourceName = "mutil_pinyin.properties";
			return getResource(context, resourceName);
		}
	
		protected static Properties getChineseTable(Context context) {
			String resourceName = "chinese.properties";
			return getResource(context, resourceName);
		}
	}

&emsp;&emsp;JPinyin的其它几个类也需要修改，加上context参数，修改后重新打包JPinyin.jar，在IDE中运行没问题，加密后使用也没有问题。将properties文件改成db文件导致加密后不能正常使用的问题我还没有跟踪到具体原因，但解决这个问题的过程和方法值得记录一下，遇到这种问题最好的办法是查看源码，大胆猜测并尝试，比在不看源码的情况下去猜测要靠谱得多。

	