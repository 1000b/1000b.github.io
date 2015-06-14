---
layout: post
title: Android应用内多进程的使用及注意事项
category: Android
tags: Android
keywords: Android,process
description: Android应用内多进程的使用及注意事项
---

## Android应用内多进程介绍及使用

&emsp;&emsp;一个应用默认只有一个进程，这个进程（主进程）的名称就是应用的包名，进程是系统分配资源和调度的基本单位，每个进程都有自己独立的资源和内存空间，其它进程不能任意访问当前进程的内存和资源，系统给每个进程分配的内存会有限制。

&emsp;&emsp;如果一个进程占用内存超过了这个内存限制，就会报OOM的问题，很多涉及到大图片的频繁操作或者需要读取一大段数据在内存中使用时，很容易报OOM的问题，如果此时在程序中人为地使用GC会严重影响程序运行的流畅性，并且有时候并没有什么卵用，多数时候我们可以在android:minSdkVersion="11"及以上的应用中，给AndroidManifest.xml的Application标签增加"android:largeHeap="true""这句话，请求系统给该应用分配更多可申请的内存：

![1](http://pic4.zhimg.com/ef5ef35b5dd4737575a1ad13b47459d1_r.jpg)

&emsp;&emsp;但是这种做法的弊端有：

- 有时候并不能彻底解决问题，比如API Level小于11时，或者是应用需要的内存比largeHeap分配的更大的时候；
当该应用在后台时，仍然占用着的内存，系统总的内存就那么多，如果每个应用都向系统申请更多的内存，会影响整机运行效率。

- 为了彻底地解决应用内存的问题，Android引入了多进程的概念，它允许在同一个应用内，为了分担主进程的压力，将占用内存的某些页面单独开一个进程，比如Flash、视频播放页面，频繁绘制的页面等。Android多进程使用很简单，只需要在AndroidManifest.xml的声明四大组件的标签中增加"android:process"属性即可，process分私有进程和全局进程，私有进程的名称前面有冒号，全局进程没有，like this：

![2](http://pic4.zhimg.com/e4c8ce8c4ef4df9da3550f9f713c8137_b.jpg)

&emsp;&emsp;为了节省系统内存，在退出该Activity的时候可以将其杀掉(如果没有人为杀掉该进程，在程序完全退出时该进程会被系统杀掉)，like this：

![3](http://pic4.zhimg.com/005652c73b68b39e3cc5fb3254bd3c9e_b.jpg)

## 使用多进程会遇到的一些问题：

### 断点调试

&emsp;&emsp;调试就是跟踪程序运行过程中的堆栈信息，正如前面所讲，每个进程都有自己独立的资源和内存空间，每个进程的堆栈信息也是独立的，如果要在不同的进程间调试，是实现不了的，不过可以通过如下两种方式进行调试：

&emsp;&emsp;调试的时候去掉AndroidManifest.xml文件中Activity的android:process标签，这样保证调试状态下是在同一进程中，堆栈信息是连贯的，在调试完成后记得复原该属性；
通过打印进行调试，但这种效率比较低。

### Activity管理：

&emsp;&emsp;通常我们为了完全退出一个应用，会在Application里面实现ActivityLifecycleCallbacks接口，监听Activity的生命周期，通过LinkedList来管理所有的Activity：

	public class TestApplication extends Application implements ActivityLifecycleCallbacks{
		@Override
		public void onCreate() {
			super.onCreate();
			registerActivityLifecycleCallbacks(this);
		}
	
		@Override
		public void onActivityCreated(Activity activity, Bundle arg1) {
			if (null != mExistedActivitys && null != activity) {
	            // 把新的 activity 添加到最前面，和系统的 activity 堆栈保持一致
	            mExistedActivitys.offerFirst(new ActivityInfo(activity,ActivityInfo.STATE_CREATE));
	        }
		}
	
		@Override
		public void onActivityDestroyed(Activity activity) {
			if (null != mExistedActivitys && null != activity) {
	            ActivityInfo info = findActivityInfo(activity);
	            if (null != info) {
	                mExistedActivitys.remove(info);
	            }
	        }
		}
	
		@Override
		public void onActivityStarted(Activity activity) {
			
		}
	
		@Override
		public void onActivityResumed(Activity activity) {
			
		}
	
		@Override
		public void onActivityPaused(Activity activity) {
			
		}
	
		@Override
		public void onActivityStopped(Activity activity) {
			
		}
	
		@Override
		public void onActivitySaveInstanceState(Activity activity, Bundle outState) {
			
		}
	
		public void exitAllActivity() {
	        if (null != mExistedActivitys) {
	            // 先暂停监听（省得同时在2个地方操作列表）
	        	unregisterActivityLifecycleCallbacks( this );
	
	            // 弹出的时候从头开始弹，和系统的 activity 堆栈保持一致
	            for (ActivityInfo info : mExistedActivitys) {
	                if (null == info || null == info.mActivity) {
	                    continue;
	                }
	
	                try {
	                    info.mActivity.finish();
	                } catch (Exception e) {
	                    e.printStackTrace();
	                }
	            }
	
	            mExistedActivitys.clear();
	            // 退出完之后再添加监听
	            registerActivityLifecycleCallbacks( this );
	        }
	    }
		
		private ActivityInfo findActivityInfo(Activity activity) {
	        if (null == activity || null == mExistedActivitys) {
	            return null;
	        }
	
	        for (ActivityInfo info : mExistedActivitys) {
	            if (null == info) {
	                continue;
	            }
	
	            if (activity.equals(info.mActivity)) {
	                return info;
	            }
	        }
	
	        return null;
	    }
		
		class ActivityInfo {
	        private final static int STATE_NONE = 0;
	        private final static int STATE_CREATE = 1;
	
	        Activity mActivity;
	        int mState;
	
	        ActivityInfo() {
	            mActivity = null;
	            mState = STATE_NONE;
	        }
	
	        ActivityInfo(Activity activity, int state) {
	            mActivity = activity;
	            mState = state;
	        }
	    }
		
		private LinkedList<ActivityInfo> mExistedActivitys = new LinkedList<ActivityInfo>();
	}


&emsp;&emsp;但是如果应用内有多个进程，每创建一个进程就会跑一次Application的onCreate方法，每个进程内存都是独立的，所以通过这种方式无法实现将应用的Activity放在同一个LinkedList中，不能实现完全退出一个应用。

### 内存共享：

&emsp;&emsp;不同进程之间内存不能共享，最大的弊端是他们之间通信麻烦，不能将公用数据放在Application中，堆栈信息、文件操作也是独立的，如果他们之间传递的数据不大并且是可序列化的，可以考虑通过Bundle传递， 如果数据量较大，则需要通过AIDL或者文件操作来实现。

## 结语

&emsp;&emsp;通过多进程可以分担应用内主进程的压力，但这是下下策，最好的解决方案还是要做好性能优化。

## More

- [Processes and Threads](http://developer.android.com/guide/components/processes-and-threads.html)

- [One Android application's activity running in the process of another; what does `android:multiprocess` do?](http://stackoverflow.com/questions/18832542/one-android-applications-activity-running-in-the-process-of-another-what-does)

- [[问答]Android开发中何时使用多进程？ · Issue #7 · android-cn/android-discuss · GitHub](https://github.com/android-cn/android-discuss/issues/7)

- [Usage of android:process](http://stackoverflow.com/questions/7142921/usage-of-androidprocess)



