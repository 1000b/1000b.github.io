---
layout: post
title: Android通过广播更新文件和文件夹到媒体库
category: Android
tags: Android
keywords: Android
description: Android通过广播更新文件和文件夹到媒体库
---

## Android媒体库

&emsp;&emsp;Android的媒体库其实就是一个数据库文件，当系统启动完成、SD卡插拔或者接收到“Intent.ACTION_MEDIA_SCANNER_SCAN_FILE”广播消息时，系统会扫描文件系统中的数据，将新增和删除的文件信息更新到这个数据库中，这样当其它程序获取文件系统中的文件信息时，直接操作这个数据库就行了，不用去文件系统中取，接收媒体库扫描广播和触发扫描文件系统的源码如下：

		public class MediaScannerReceiver extends BroadcastReceiver {
	    private final static String TAG = "MediaScannerReceiver";
	
	    @Override
	    public void onReceive(Context context, Intent intent) {
	        final String action = intent.getAction();
	        final Uri uri = intent.getData();
	        if (Intent.ACTION_BOOT_COMPLETED.equals(action)) {
	            // Scan both internal and external storage
	            scan(context, MediaProvider.INTERNAL_VOLUME);
	            scan(context, MediaProvider.EXTERNAL_VOLUME);
	
	        } else {
	            if (uri.getScheme().equals("file")) {
	                // handle intents related to external storage
	                String path = uri.getPath();
	                String externalStoragePath = Environment.getExternalStorageDirectory().getPath();
	
	                Log.d(TAG, "action: " + action + " path: " + path);
	                if (Intent.ACTION_MEDIA_MOUNTED.equals(action)) {
	                    // scan whenever any volume is mounted
	                    scan(context, MediaProvider.EXTERNAL_VOLUME);
	                } else if (Intent.ACTION_MEDIA_SCANNER_SCAN_FILE.equals(action) &&
	                        path != null && path.startsWith(externalStoragePath + "/")) {
	                    scanFile(context, path);
	                }
	            }
	        }
	    }
	
	    private void scan(Context context, String volume) {
	        Bundle args = new Bundle();
	        args.putString("volume", volume);
	        context.startService(
	                new Intent(context, MediaScannerService.class).putExtras(args));
	    }    
	
	    private void scanFile(Context context, String path) {
	        Bundle args = new Bundle();
	        args.putString("filepath", path);
	        context.startService(
	                new Intent(context, MediaScannerService.class).putExtras(args));
	    }    

## 获取媒体库中指定文件格式的文件列表

&emsp;&emsp;这个过程其实就是查询数据库，具体实现如下：

		/**
	     * 获取指定文件列表的全路径，如果对应文件存在则返回文件的路径，如果对应文件不存在，则路径为空
	     * eg:
	     * List<String> filePathList = FileUtils.getExistFiles(this,new String[]{"1.ptr","2.hst","3.dct"});
	     * 如果1.ptr和3.dct存在，2.hst不存在则返回的结果是：
	     * filePathList.get(0) = /mnt/sdcard/1.ptr
	     * filePathList.get(1) = null
	     * filePathList.get(2) = /mnt/sdcard/3.dct
	     * */
	    public static List<String> getExistFiles(final Context context, final String[] fileNames){
	    	if( null == context || null == fileNames || fileNames.length == 0 ){
	            return null;
	        }
	
	    	
	    	String filePath = null;
			String suffix = null;
			String linkType = null;
	    	List<String> filePathList = new ArrayList<String>();
	    	for(String fileName : fileNames){
	            suffix = fileName.substring(fileName.lastIndexOf('.')+1, fileName.length());
	            linkType = " LIKE '%" + suffix + "'";
	
	            ContentResolver contentResolver = context.getContentResolver( );
	            if( null != contentResolver ){
	                Uri uri = MediaStore.Files.getContentUri( "external" );
	                // 为了效率起见不应当传入null,否则默认取出所有的字段。这里应当根据自己的需求来定，比如下面只需要路径、大小信息
	                Cursor cursor = contentResolver.query( uri,
	                                	new String[ ]{ MediaStore.Files.FileColumns.DATA, MediaStore.Files.FileColumns.SIZE },
	                                	MediaStore.Files.FileColumns.DATA + linkType, null, null );
	
	                if( cursor != null ){
	                    if( cursor.moveToFirst( ) ){
	                        do{
	                        	filePath = cursor.getString( cursor.getColumnIndex( MediaStore.Files.FileColumns.DATA ) );
	                            if(filePath.substring(filePath.lastIndexOf('/') + 1, filePath.length()).equals(fileName)){
	                            	filePathList.add(filePath);
	                                break;
	                            }
	                        }while( cursor.moveToNext( ) );
	                    }
	
	                    if( !cursor.isClosed( ) ){
	                        cursor.close( );
	                    }
	                }
	            }
	    	}
	
	        return filePathList;
	    }

## Android通过广播更新文件和文件夹到媒体库

&emsp;&emsp;网上有很多介绍更新文件和文件夹到媒体库的，更新文件到媒体库很简单，直接按照下面的方式发送广播就可以：

		/**
	     * 通知媒体库更新文件
	     * @param context
	     * @param filePath 文件全路径
	     * 
	     * */
		public void scanFile(Context context, String filePath) {
			Intent scanIntent = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
			scanIntent.setData(Uri.fromFile(new File(filePath)));
			context.sendBroadcast(scanIntent);
		}

&emsp;&emsp;但更新文件夹就比较麻烦了，网上介绍通过发送"android.intent.action.MEDIA_SCANNER_SCAN_DIR"显然是不行的，通过查看MediaScannerReceiver.java的源码就会发现这根本行不通，所以只能另辟蹊径，但是从代码中发现可以通过发送“Intent.ACTION_MEDIA_MOUNTED”来更新文件夹，具体实现方式是：
	
		/**
	     * 通知媒体库更新文件夹
	     * @param context
	     * @param filePath 文件夹
	     * 
	     * */
		public void scanFile(Context context, String filePath) {
			Intent scanIntent = new Intent(Intent.ACTION_MEDIA_MOUNTED);
			scanIntent.setData(Uri.fromFile(new File(filePath)));
			context.sendBroadcast(scanIntent);
		}

## More

- [Scan Media Files in Android](http://droidyue.com/blog/2014/01/19/scan-media-files-in-android/)

- [android刷新媒体库](http://blog.csdn.net/winson_jason/article/details/8487991)

	