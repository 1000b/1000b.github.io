---
layout: post
title: 调用AsyncTask的excute方法不能立即执行程序的原因分析及改善方案
category: Android
tags: Android
keywords: Android,AsyncTask,excute,AsyncTask not run
description: 调用AsyncTask的excute方法不能立即执行程序的原因分析及改善方案
---

&emsp;&emsp;最近在项目中遇到一个关于调用AsyncTask的excute方法不能立即执行程序的问题，项目的targetSdkVersion是15，最后分析发现是AsyncTask的运行机制导致，特地总结出来以免后面再犯同样的错误。

## 通过源码分析AsyncTask的工作原理

### AsyncTask官方文档解读

#### AsyncTask介绍

&emsp;&emsp;AsyncTask是一个Android SDK中轻量级的异步任务类，它在线程池中执行后台任务，把执行进度和执行结果返回给主线程，并在主线程更新UI，AsyncTask实质上是对Thread和Handler的封装，通过AsyncTask能够更方便地在执行后台任务的过程中和结束后实现更新UI操作。

#### AsyncTask的用法

&emsp;&emsp;AsyncTask是一个抽象类，它需要被实现后才能正常使用，子类必须要复写doInBackground方法，如果需要在执行完后台任务后更新UI，则需要实现onPostExecute方法，下面是一个AsyncTask使用实例：

	class DownloadFilesTask extends AsyncTask<URL, Integer, Long>{
		@Override
		protected Void doInBackground(Integer... params) {
			int count = urls.length;
         	long totalSize = 0;
          	for (int i = 0; i < count; i++) {
              	totalSize += Downloader.downloadFile(urls[i]);
              	publishProgress((int) ((i / (float) count) * 100));
         	}

          	return totalSize;
		}
		
		protected void onProgressUpdate(Integer... progress) {
          	setProgressPercent(progress[0]);
      	}
 
      	protected void onPostExecute(Long result) {
          	showDialog("Downloaded " + result + " bytes");
      	}
	}

	// 调用方式如下
	new DownloadFilesTask().excute(url1, url2, url3);


#### 参数说明

&emsp;&emsp;AsyncTask的三个参数<Params, Progress, Result>说明：

- **Params**：执行AsyncTask时传递的参数类型；

- **Progress**:执行后台任务时更新进度的进度值类型；

- **Result**:后台任务执行完成后的返回值类型。

**注意：**并不是所有的参数都需要指明类型，如果某一个参数你没有用到，改成Void类型即可。

#### 执行流程

&emsp;&emsp;在AsyncTask执行的过程中，会经历如下四个步骤：

- **onPreExecute**：UI线程中调用，在调用excute方法后会立即被调用，这个方法适用于初始化task，比如显示后台任务进度条；

- **doInBackground**：非UI线程中调用，在onPreExecute执行后调用，这个方法适用于执行耗时的操作，excute方法中的Params参数就是通过这个方法传递的，该方法的返回值就是Task执行的结果，在doInBackground方法执行过程中，可以通过publishProgress方法来更新后台任务的执行进度；

- **onProgressUpdate**：UI线程中调用，在publishProgress方法后被调用，这个方法用于在后台任务执行过程中显示任务的进度UI，比如它可以用于显示进度条动画或者显示进度文本；

- **onPostExecute**：UI线程中调用，后台任务执行完成后被调用，Task返回的结果以参数的形式传递到该方法中。

#### 取消Task

&emsp;&emsp;AsyncTask可以在任何时候通过cancel(boolean)方法取消，调用这个方法后，isCancelled()方法的返回值会为true，当执行这个cancle方法后，doInBackground方法执行完成后不会调用onPostExecute方法而是执行onCancelled回调。

#### 注意

&emsp;&emsp;在使用AsyncTask的过程中必须要遵守如下原则：

- AsyncTask必须在UI线程中实例化；

- excute方法必须要在UI线程中调用；

- 不要人为地调用AsyncTask的回调方法：onPreExecute、onPostExecute、doInBackground和onProgressUpdate；

- 一个AsyncTask实例只能执行一次，如果调用多次，将会报异常。

### 源码解读

&emsp;&emsp;执行AsyncTask很简单，先实例化，然后调用excute方法，excute方法的代码如下：

	public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        return executeOnExecutor(sDefaultExecutor, params);
    }

&emsp;&emsp;通过查看sDefaultExecutor的代码发现，AsyncTask默认自己维护一个静态的线程池，而该线程池只允许同时执行一个线程，也就是说，不管多少个AsyncTask,只要是调用execute()方法，都是共享这个默认进程池的，你的任务必须在之前的任务执行完以后，才能执行。可以理解为，默认情况下，所有的AsyncTask在一个独立于UI线程的线程中执行，任务需要排队，先execute的先执行，后面的只能等。具体的源码如下：

	private static class SerialExecutor implements Executor {
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        Runnable mActive;

        public synchronized void execute(final Runnable r) {
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        r.run();
                    } finally {
						// 上一个任务执行完成后才会执行下一个
                        scheduleNext();
                    }
                }
            });
            if (mActive == null) {
                scheduleNext();
            }
        }

        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }

&emsp;&emsp;除了excute方法外，还可以调用executeOnExecutor（其实excute方法也是调用的executeOnExecutor方法，只是线程池是在AsyncTask中默认定义好的），如果使用executeOnExecutor方法，可以在外部自定义线程池，解决不能并发执行异步任务的问题。

## AsyncTask在不同SDK版本中的区别

&emsp;&emsp;通过查阅官方文档发现，AsyncTask首次引入时，异步任务是在一个独立的线程中顺序地执行，也就是说一次只能执行一个任务，不能并行地执行，从1.6开始，AsyncTask中引入了线程池，支持同时执行5个异步任务，也就是说同时只能有5个线程运行，超过的线程只能等待，等待前面的线程某个执行完了才被调度和运行。换句话说，如果一个进程中的AsyncTask实例个数超过5个，那么假如前5个都运行很长时间的话，那么第6个只能等待机会了。这是AsyncTask的一个限制，而且对于2.3以前的版本无法解决。如果你的应用需要大量的后台线程去执行任务，那么你只能放弃使用AsyncTask，自己创建线程池来管理Thread，或者干脆不用线程池直接使用Thread也无妨。不得不说，虽然AsyncTask较Thread使用起来比较方便，但是它最多只能同时运行5个线程，这也大大局限了它的实力，你必须要小心的设计你的应用，错开使用AsyncTask的时间，尽力做到分时，或者保证数量不会大于5个，否则就可能遇到上面提到的问题。可能是Google意识到了AsyncTask的局限性了，从Android 3.0开始对AsyncTask的API做出了一些调整：每次只启动一个线程执行一个任务，完成之后再执行第二个任务，也就是相当于只有一个后台线程在执行所提交的任务，可以通过代码在不同sdk版本中执行的具体情况来验证官方文档的说法：

**测试代码**：

	private void taskTest(){
		for(int index = 0; index < 10; index++){
			new LoadTask().execute(index);
		}
	}
	
	class LoadTask extends AsyncTask<Integer, Void, Void>{
		@Override
		protected Void doInBackground(Integer... params) {
			try {
				Thread.sleep(1000);
				SimpleDateFormat df = new SimpleDateFormat("yyyy-mm-dd hh:mm:ss");
				System.out.println("targetSdkVersion == " + getTargetSdkVersion() + " LoadTask#" + params[0] + " time == " + df.format(new Date()));
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			return null;
		}
		
		@Override
		protected void onPostExecute(Void result) {
			super.onPostExecute(result);
		}
	}
	
	private String getTargetSdkVersion(){
		int targetSdkVersion = 0;
		try { 
            PackageInfo packageInfo = getPackageManager().getPackageInfo(getPackageName(), 0);
            targetSdkVersion = packageInfo.applicationInfo.targetSdkVersion;
        }catch (PackageManager.NameNotFoundException e) {
            Log.e(TAG, e.getMessage());
        }
		
		return targetSdkVersion+"";
	}

**测试结果**:

- 2.2中：

![](http://ww1.sinaimg.cn/large/6d17e381gw1ewjfceb9doj20c6072tba.jpg)

- 4.0.3中：

![](http://ww4.sinaimg.cn/large/6d17e381gw1ewjfd5pisdj20dd06zdie.jpg)

&emsp;&emsp;测试程序是同时执行10个AsyncTask任务，从不同sdk版本的打印结果来看，2.2中是一次执行5个任务，在4.0.3中一次执行一个任务，等上一个任务执行完成后才执行下一个任务。

## excute方法不能立即执行的改善方案

&emsp;&emsp;习惯了参照官方文档和线程的代码，没有认真研读源码导致踩了AsyncTask的这个坑，如果知道使用executeOnExecutor方法，自己定义线程池就不会出现Task任务没有立即执行的情况，这再次印证了阅读android源码的重要性，最后具体的解决方式如下：

	LinkedBlockingQueue<Runnable> blockingQueue = new LinkedBlockingQueue<Runnable>();
	ExecutorService exec = new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, blockingQueue);
	new LoadTask().executeOnExecutor(1);

## 参考资料

- [Android实战技巧：深入解析AsyncTask](http://blog.csdn.net/hitlion2008/article/details/7983449)

- [Android开发:AsyncTask在调用execute()后没有马上执行的问题](http://www.pocketdigi.com/20140408/1291.html)

- [AsyncTask源码](https://github.com/android/platform_frameworks_base/blob/master/core/java/android/os/AsyncTask.java)




