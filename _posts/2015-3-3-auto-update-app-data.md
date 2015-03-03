---
layout: post
title: 通过观察者模式监听媒体库的变化实现APP本地数据自动更新
category: Android
tags: Android
keywords: Android，媒体库，ContentObserver，MediaStore
description:  通过观察者模式监听媒体库的变化实现APP本地数据自动更新
---

### 前言
&emsp;&emsp;当我在使用音乐播放器和各种小说APP的过程中，感觉非常不好的一个体验就是你需要通过手动点击全盘检索后，新下载的数据、从磁盘拷贝的数据才会更新显示在列表上，这对于我们来说看上去没有什么不对，但从用户的角度出发这是一个非常不好的体验，因为多数人是根本不知道全盘检索这个概念的，手动更新APP的本地数据无形之中增加了用户使用APP的学习成本。

&emsp;&emsp;我们之前做的小系统APP也是通过手动检索这种方式来刷新程序中的本地数据列表的，在接触到Android的媒体库后，发现这个问题能够通过观察者模式监听Android媒体数据库变化的方式来实现APP本地数据的自动更新，在成功实现这个功能后，现在将其总结下来方便后面查看。

### Android媒体库

- **媒体库是什么？：**在Android系统中，为了提高应用检索数据的效率，Android会将存储在文件系统中的文件信息保存在一个数据库文件中，这样在应用中就可以通过读取该数据库来快速查找满足APP需求的文件列表，比如一个电子书阅读APP，通过如下方法就可以获取到媒体库中存在的电子书文件列表，保存哪些格式的文件是可以通过修改Android原来来调整的，不过对于多媒体文件来说，Android原生系统默认就保存在媒体库中了：
	
		/**
		 * 从媒体库中获取指定后缀的文件列表
		 * 
		 * @param searchFileSuffix 文件后缀列表，eg: new String[]{"epub","mobi","pdf","txt"};
		 * @return 指定后缀的文件列表
		 * */
		public static ArrayList<String> getSupportFileList(Context context, String[] searchFileSuffix) {
			ArrayList<String> searchFileList = null;
			if (null == context || null == searchFileSuffix
					|| searchFileSuffix.length == 0) {
				return null;
			}
	
			String searchPath = "";
			int length = searchFileSuffix.length;
			for (int index = 0; index < length; index++) {
				searchPath += (MediaStore.Files.FileColumns.DATA + " LIKE '%" + searchFileSuffix[index] + "' ");
				if ((index + 1) < length) {
					searchPath += "or ";
				}
			}
			searchFileList = new ArrayList<String>();
			Uri uri = MediaStore.Files.getContentUri("external");
			Cursor cursor = context.getContentResolver().query(
					uri,new String[] { MediaStore.Files.FileColumns.DATA,MediaStore.Files.FileColumns.SIZE }, searchPath, null,null);
	
			if (cursor == null) {
				System.out.println("Cursor 获取失败!");
			} else {
				if (cursor.moveToFirst()) {
					do {
						String filepath = cursor.getString(cursor.getColumnIndex(MediaStore.Files.FileColumns.DATA));
						if (isFileExist(filepath)) {
							try {
								searchFileList.add(new String(filepath.getBytes("UTF-8")));
							} catch (UnsupportedEncodingException e) {
								e.printStackTrace();
							}
						}
	
					} while (cursor.moveToNext());
				}
	
				if (!cursor.isClosed()) {
					cursor.close();
				}
			}
	
			return searchFileList;
		}
		
		/**
		 * 判断SD卡上的文件夹是否存在
		 * 
		 * @param fileName 文件名
		 * @return true 文件存在，false 文件不存在
		 */
		private static boolean isFileExist(String filePath) {
			File file = null;
			boolean isExist = false;
	
			if (null != filePath) {
				file = new File(filePath);
				isExist = (null != file && file.isFile()) ? file.exists() : false;
				if (isExist && null != file && 0 == file.length()) {
					isExist = false;
				}
			}
	
			return isExist;
		}

- **媒体库更新时机：**Android系统会在系统开机、USB插拔、TF卡插拔的时候自动更新媒体库（将新增的文件添加到媒体库中，移除不存在的文件数据记录），除了Android系统会自动更新媒体库文件外，开发者也可以在程序中手动更新媒体库，这样能够在文件系统中有新的文件或者通过程序删掉某些文件时能够将动态及时更新到媒体库，保证媒体库中的文件信息是实时的，更新的具体方式如下：

	context.sendBroadcast( new Intent( Intent.ACTION_MEDIA_SCANNER_SCAN_FILE, Uri.parse( "file://" + filePath ) ) );

**注：**上面介绍是的更新单个文件的方式，Android没有提供直接更新整个文件夹的方式，如果是整个文件夹，可以先得到文件夹下的所有文件路径列表，然后挨个更新，对于这个如果有更先进的方法欢迎提出。

### APP本地数据自动更新的具体实现

&emsp;&emsp;类似于上述的音视频播放器、小说阅读APP，如果我们需要实现本地数据APP自动更新的功能，只要保持APP支持文件列表的数据库和媒体库中的对应格式的文件同步就可以了，所以我们需要做的是：监听媒体库中文件列表的变化，然后将变化告知APP即可，原理是：

- 通过广播监听USB插拔、TF卡插拔，如果检查到在APP运行过程中有这些操作，直接通过APP全盘检索；

- 通过观察者模式监听媒体库中的文件变化，如果有变化，每隔五秒钟将APP现存列表和媒体库中检索到对应格式的文件列表做比较，如果列表有变化，则将变化的列表更新给APP；

- 在APP进入、退出时注册/反注册广播、打开/关闭计时器。

&emsp;&emsp;整个代码非常简单，一个类搞定，具体代码如下：

	import java.io.UnsupportedEncodingException;
	import java.util.ArrayList;
	import java.util.Timer;
	
	import android.content.BroadcastReceiver;
	import android.content.Context;
	import android.content.Intent;
	import android.content.IntentFilter;
	import android.database.ContentObserver;
	import android.database.Cursor;
	import android.net.Uri;
	import android.os.AsyncTask;
	import android.os.Handler;
	import android.provider.MediaStore;
	
	/**
	 * 自动更新书架
	 * 
	 * */
	public class AutoRefreshBookShelf {
	    public AutoRefreshBookShelf( Context context, AutoRefreshListener autoRefreshListener, String[] supportSuffix ) throws NullPointerException{
	        if( null == context || null == autoRefreshListener || null == supportSuffix ){
	            throw new NullPointerException( "传非空的参数进来！" );
	        }
	        
	        mContext = context;
	        mAutoRefreshListener = autoRefreshListener;
	        mSupportSuffix = supportSuffix;
	        
	        initAutoRefreshBookShelf( );
	    }
	    
	    // 不在本界面停止后台检索
	    public void onPause( ){
	        stopCheckFileTimer( );
	    }
	    
	    // 返回界面恢复后台检索
	    public void onResume( ){
	        startCheckFileTimer( );
	    }
	    
	    /**
	     * 注销广播
	     * 
	     * */
	    public void unregisterAutoRefreshBookShelf( ) throws NullPointerException{
	        if( null == mBroadcastReceiver || null == mMediaStoreChangeObserver || null == mContext ){
	            throw new NullPointerException( "没有初始化" );
	        }
	        mContext.unregisterReceiver( mBroadcastReceiver );
	        mContext.getContentResolver( ).unregisterContentObserver( mMediaStoreChangeObserver );
	        stopCheckFileTimer( );
	    }
	    
	    /**
	     * 得到变化的文件列表
	     * 
	     * */
	    public void getChangedFileList( ){
	        System.out.println( "toast ================= getChangedFileList " );
	        startCheckFileTimer( );
	    }
	    
	    private void initAutoRefreshBookShelf( ){
	        startMediaFileListener( );
	        observerMediaStoreChange( );
	    }
	    
	    private void observerMediaStoreChange( ){
	        if( null == mMediaStoreChangeObserver ){
	            mMediaStoreChangeObserver = new MediaStoreChangeObserver( );
	        }
	        mContext.getContentResolver( ).registerContentObserver( MediaStore.Files.getContentUri("external"), false, mMediaStoreChangeObserver );
	    }
	    
	    /**
	     * 监听USB的状态，更新书架书本信息
	     * 
	     * */
	    private void startMediaFileListener( ){
	        if( null != mBroadcastReceiver ){
	            return;
	        }
	        
	        IntentFilter intentFilter = new IntentFilter( );
	        intentFilter.addAction( Intent.ACTION_MEDIA_SCANNER_FINISHED );
	        intentFilter.addAction( Intent.ACTION_MEDIA_MOUNTED );
	        intentFilter.addAction( Intent.ACTION_MEDIA_EJECT );
	        intentFilter.addDataScheme( "file" );
	          
	        mBroadcastReceiver = new BroadcastReceiver(){
	            @Override
	            public void onReceive(Context context,Intent intent){
	                String action = intent.getAction( );
	                if( Intent.ACTION_MEDIA_SCANNER_FINISHED.equals( action ) ){
	                    System.out.println( "toast ================= ACTION_MEDIA_SCANNER_FINISHED " );
	                    mTimerWorking = false;
	                    startCheckFileTimer( );
	                }else if( action.equals( Intent.ACTION_MEDIA_MOUNTED ) ){
	                    System.out.println( "toast ================= ACTION_MEDIA_MOUNTED or ACTION_MEDIA_EJECT " );
	                    mTimerWorking = true;
	                    mAutoRefreshListener.onBookScan( );
	                }else if( action.equals( Intent.ACTION_MEDIA_EJECT ) ){
	                    mAutoRefreshListener.onBookScan( );
	                }
	            }
	        };
	        mContext.registerReceiver( mBroadcastReceiver, intentFilter );//注册监听函数
	    }
	    
	    /**
	     * 媒体数据库变更观察类
	     * 
	     * */
	    class MediaStoreChangeObserver extends ContentObserver{
	        public MediaStoreChangeObserver( ) {
	            super( new Handler( ) );
	        }
	
	        @Override
	        public void onChange(boolean selfChange) {
	            super.onChange(selfChange);
	            startCheckFileTimer( );
	        }
	    }
	    
	    private void startCheckFileTimer( ){
	        if( mTimerWorking ){
	            return;
	        }
	        
	        mCheckFileTimer = new Timer( );
	        mCheckFileTimer.schedule( new CheckFileChangeTimerTask( ), 5000 );
	        mTimerWorking = true;
	    }
	    
	    private void stopCheckFileTimer( ){
	        if( null != mCheckFileTimer ){
	            mCheckFileTimer.cancel( );
	            mCheckFileTimer = null;
	            
	            mTimerWorking = false;
	        }
	    }
	    
	    /**
	     * 得到新增的文件列表
	     * 
	     * */
	    public ArrayList<String> getChangedFileList( Context context, String[] searchFileSuffix, ArrayList<String> existFileList ){
	        ArrayList<String> changedFileList = null;
	        if( null == context || null == searchFileSuffix ){
	            return changedFileList;
	        }
	        
	        ArrayList<String> supportFileList = getSupportFileList( context, searchFileSuffix );
	        changedFileList = getDifferentFileList( supportFileList, existFileList );
	        if( null == changedFileList || changedFileList.size( ) == 0 ){
	            changedFileList = null;
	        }
	        
	        return changedFileList;
	    }
	    
	    /**
	     * 获取新增的文件列表
	     * 
	     * */
	    private ArrayList<String> getDifferentFileList( ArrayList<String> newFileList, ArrayList<String> existFileList ){
	        ArrayList<String> differentFileList = null;
	        if( null == newFileList || newFileList.size( ) == 0 ){
	            return differentFileList;
	        }
	        
	        differentFileList = new ArrayList<String>( );
	        boolean isExist = false;
	        if( null == existFileList ){
	            // 如果已存在文件为空，那肯定是全部加进来啦。
	            for( String newFilePath : newFileList ){
	                differentFileList.add( newFilePath );
	            }
	        }else{
	            for( String newFilePath : newFileList ){
	                isExist = false;
	                for( String existFilePath : existFileList ){
	                    if( existFilePath.equals( newFilePath ) ){
	                        isExist = true;
	                        break;
	                    }
	                }
	                
	                if( !isExist ){
	                    differentFileList.add( newFilePath );
	                }
	            }
	        }
	        
	        return differentFileList;
	    }
	    
	    /**
	     * 从媒体库中获取指定后缀的文件列表
	     * 
	     * */
	    public ArrayList<String> getSupportFileList( Context context, String[] searchFileSuffix ) {
	        ArrayList<String> searchFileList = null;
	        if( null == context || null == searchFileSuffix || searchFileSuffix.length == 0 ){
	            return null;
	        }
	        
	        String searchPath = "";
	        int length = searchFileSuffix.length;
	        for( int index = 0; index < length; index++ ){
	            searchPath += ( MediaStore.Files.FileColumns.DATA + " LIKE '%" + searchFileSuffix[ index ] + "' " );
	            if( ( index + 1 ) < length ){
	                searchPath += "or ";
	            }
	        }
	        
	        searchFileList = new ArrayList<String>();
	        Uri uri = MediaStore.Files.getContentUri("external");
	        Cursor cursor = context.getContentResolver().query(
	                uri, new String[] { MediaStore.Files.FileColumns.DATA, MediaStore.Files.FileColumns.SIZE, MediaStore.Files.FileColumns._ID },
	                searchPath, null, null);
	        
	        String filepath = null;
	        if (cursor == null) {
	            System.out.println("Cursor 获取失败!");
	        } else {
	            if (cursor.moveToFirst()) {
	                do {
	                    filepath = cursor.getString(cursor.getColumnIndex(MediaStore.Files.FileColumns.DATA));
	                    try {
	                        searchFileList.add(new String(filepath.getBytes("UTF-8")));
	                    } catch (UnsupportedEncodingException e) {
	                        e.printStackTrace();
	                    }
	
	                } while (cursor.moveToNext());
	            }
	
	            if (!cursor.isClosed()) {
	                cursor.close();
	            }
	        }
	
	        return searchFileList;
	    }
	    
	    /**
	     * 得到媒体库更新的文件
	     * 
	     * */
	    class GetMediaStoreDataTask extends AsyncTask< Void , Void , Void>{
	        @Override
	        protected Void doInBackground(Void... arg0) {
	            ArrayList<String> changedFileList = getChangedFileList( mContext, mSupportSuffix, mAutoRefreshListener.onGetBookPathList( ) );
	            if( null != changedFileList && changedFileList.size( ) > 0 ){
	                mAutoRefreshListener.onBookRefresh( changedFileList );
	            }
	            mTimerWorking = false;
	            
	            return null;
	        }
	    }
	    
	    class CheckFileChangeTimerTask extends java.util.TimerTask{
	        @Override
	        public void run() {
	            new GetMediaStoreDataTask( ).execute( );
	        }
	    }
	    
	    /**
	     * 书架自动刷新接口
	     * 
	     * */
	    public interface AutoRefreshListener{
	        public ArrayList<String> onGetBookPathList( ); // 得到书架书本列表
	        public void onBookRefresh( ArrayList<String> bookInfoList );// 刷新书架
	        public void onBookScan( );//全盘扫描书架
	    }
	    
	    private boolean mTimerWorking = false;
	    private Context mContext = null;
	    private String[] mSupportSuffix = null;
	    private BroadcastReceiver mBroadcastReceiver = null;
	    private MediaStoreChangeObserver mMediaStoreChangeObserver = null;
	    private AutoRefreshListener mAutoRefreshListener = null;
	    private Timer mCheckFileTimer = null;
	}


**注意：**建议该功能只在APP运行时开启，因为现实的文件列表只有你真正在使用APP时才会去查看，所以没有必要通过这种方式在后台操作。







