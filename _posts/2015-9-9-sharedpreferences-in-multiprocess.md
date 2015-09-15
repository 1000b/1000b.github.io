---
layout: post
title: SharedPreferences在多进程中的使用及注意事项
category: Android
tags: Android
keywords: SharedPreferences
description: SharedPreferences在多进程中的使用及注意事项
---

###　通过SharedPreferences实现进程间数据共享

&emsp;&emsp;之前为了解决应用的内存压力，在同一个应用中使用了多进程，但在程序自测的过程中发现不同进程之间的SharedPreferences数据不能共享，但应用内很多数据都是通过SharedPreferences来保存的，如果改成其它多进程通信的方式改动比较大。通过查看源码发现，在API Level>=11即Android 3.0可以通过Context.MODE_MULTI_PROCESS属性来实现SharedPreferences多进程共享，具体使用方式如下：
	
	public class PreferencesUtils {
		public static String PREFERENCE_NAME = "SharedPreferencesDemo";
		
		private PreferencesUtils(){
	
		}
	
		public static boolean putString(Context context, String key, String value) {
			SharedPreferences settings = context.getSharedPreferences(PREFERENCE_NAME, Context.MODE_MULTI_PROCESS);
			SharedPreferences.Editor editor = settings.edit();
			editor.putString(key, value);
			return editor.commit();
		}
	
		public static String getString(Context context, String key, String defaultValue) {
			SharedPreferences settings = context.getSharedPreferences(PREFERENCE_NAME, Context.MODE_MULTI_PROCESS);
			return settings.getString(key, defaultValue);
		}
	}

### SharedPreferences时间进程间数据共享会导致的问题

&emsp;&emsp;本来以为通过MODE_MULTI_PROCESS属性使用SharedPreferences就可以解决不同进程之间不能共享数据的问题了，但SQA总是反馈一些随机但出现频率比较大的bug，比如在使用过程中没有清除程序数据的前提下，会出现欢迎界面和操作指引，这是通过保存在SharedPreferences的标志来判断用户是否是第一次启动程序的，分析发现保存在SharedPreferences中的数据丢失了，但代码中并没有去清除这些数据，所以推测可能是不同进程同一时间对SharedPreferences操作导致的，经验证确实如此，去掉多进程就不会再出现这个问题了。


### 解决方案

&emsp;&emsp;由于进程间是不能内存共享的，每个进程操作的SharedPreferences都是一个单独的实例，上述的问题并不能通过锁来解决，这导致了多进程间通过SharedPreferences来共享数据是不安全的，这个问题只能通过多进程间其它的通信方式或者是在确保不会同时操作SharedPreferences数据的前提下使用SharedPreferences来解决。


### 参考资料

- [Use SharedPreferences on multi-process mode](http://stackoverflow.com/questions/27827678/use-sharedpreferences-on-multi-process-mode)

