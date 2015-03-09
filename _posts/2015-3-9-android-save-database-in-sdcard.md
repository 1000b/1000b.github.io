---
layout: post
title: Android将数据库保存到SD卡的实现
category: Android
tags: Android
keywords: Android,Sqlite,SDCard,数据库
description:  Android将数据库保存到SD卡的实现
---

&emsp;&emsp;有时候为了需要，会将数据库保存到外部存储或者SD卡中（对于这种情况可以通过加密数据来避免数据被破解），比如一个应用支持多个数据，每个数据都需要有一个对应的数据库，并且数据库中的信息量特别大时，这显然更应该将数据库保存在外部存储或者SD卡中，因为RAM的大小是有限的；其次在写某些测试程序时将数据库保存在SD卡更方便查看数据库中的内容。

&emsp;&emsp;Android通过SQLiteOpenHelper创建数据库时默认是将数据库保存在'/data/data/应用程序名/databases'目录下的，只需要在继承SQLiteOpenHelper类的构造函数中传入数据库名称就可以了，但如果将数据库保存到指定的路径下面，都需要通过重写继承SQLiteOpenHelper类的构造函数中的context，因为：在阅读SQLiteOpenHelper.java的源码时会发现：创建数据库都是通过Context的openOrCreateDatabase方法实现的，如果我们需要在指定的路径下创建数据库，就需要写一个类继承Context，并复写其openOrCreateDatabase方法，在openOrCreateDatabase方法中指定数据库存储的路径即可，下面为类SQLiteOpenHelper中getWritableDatabase和getReadableDatabase方法的源码，SQLiteOpenHelper就是通过这两个方法来创建数据库的。
	
		/**
	     * Create and/or open a database that will be used for reading and writing.
	     * The first time this is called, the database will be opened and
	     * {@link #onCreate}, {@link #onUpgrade} and/or {@link #onOpen} will be
	     * called.
	     *
	     * <p>Once opened successfully, the database is cached, so you can
	     * call this method every time you need to write to the database.
	     * (Make sure to call {@link #close} when you no longer need the database.)
	     * Errors such as bad permissions or a full disk may cause this method
	     * to fail, but future attempts may succeed if the problem is fixed.</p>
	     *
	     * <p class="caution">Database upgrade may take a long time, you
	     * should not call this method from the application main thread, including
	     * from {@link android.content.ContentProvider#onCreate ContentProvider.onCreate()}.
	     *
	     * @throws SQLiteException if the database cannot be opened for writing
	     * @return a read/write database object valid until {@link #close} is called
	     */
	    public synchronized SQLiteDatabase getWritableDatabase() {
	        if (mDatabase != null) {
	            if (!mDatabase.isOpen()) {
	                // darn! the user closed the database by calling mDatabase.close()
	                mDatabase = null;
	            } else if (!mDatabase.isReadOnly()) {
	                return mDatabase;  // The database is already open for business
	            }
	        }
	
	        if (mIsInitializing) {
	            throw new IllegalStateException("getWritableDatabase called recursively");
	        }
	
	        // If we have a read-only database open, someone could be using it
	        // (though they shouldn't), which would cause a lock to be held on
	        // the file, and our attempts to open the database read-write would
	        // fail waiting for the file lock.  To prevent that, we acquire the
	        // lock on the read-only database, which shuts out other users.
	
	        boolean success = false;
	        SQLiteDatabase db = null;
	        if (mDatabase != null) mDatabase.lock();
	        try {
	            mIsInitializing = true;
	            if (mName == null) {
	                db = SQLiteDatabase.create(null);
	            } else {
	                db = mContext.openOrCreateDatabase(mName, 0, mFactory, mErrorHandler);
	            }
	
	            int version = db.getVersion();
	            if (version != mNewVersion) {
	                db.beginTransaction();
	                try {
	                    if (version == 0) {
	                        onCreate(db);
	                    } else {
	                        if (version > mNewVersion) {
	                            onDowngrade(db, version, mNewVersion);
	                        } else {
	                            onUpgrade(db, version, mNewVersion);
	                        }
	                    }
	                    db.setVersion(mNewVersion);
	                    db.setTransactionSuccessful();
	                } finally {
	                    db.endTransaction();
	                }
	            }
	
	            onOpen(db);
	            success = true;
	            return db;
	        } finally {
	            mIsInitializing = false;
	            if (success) {
	                if (mDatabase != null) {
	                    try { mDatabase.close(); } catch (Exception e) { }
	                    mDatabase.unlock();
	                }
	                mDatabase = db;
	            } else {
	                if (mDatabase != null) mDatabase.unlock();
	                if (db != null) db.close();
	            }
	        }
	    }
	
	    /**
	     * Create and/or open a database.  This will be the same object returned by
	     * {@link #getWritableDatabase} unless some problem, such as a full disk,
	     * requires the database to be opened read-only.  In that case, a read-only
	     * database object will be returned.  If the problem is fixed, a future call
	     * to {@link #getWritableDatabase} may succeed, in which case the read-only
	     * database object will be closed and the read/write object will be returned
	     * in the future.
	     *
	     * <p class="caution">Like {@link #getWritableDatabase}, this method may
	     * take a long time to return, so you should not call it from the
	     * application main thread, including from
	     * {@link android.content.ContentProvider#onCreate ContentProvider.onCreate()}.
	     *
	     * @throws SQLiteException if the database cannot be opened
	     * @return a database object valid until {@link #getWritableDatabase}
	     *     or {@link #close} is called.
	     */
	    public synchronized SQLiteDatabase getReadableDatabase() {
	        if (mDatabase != null) {
	            if (!mDatabase.isOpen()) {
	                // darn! the user closed the database by calling mDatabase.close()
	                mDatabase = null;
	            } else {
	                return mDatabase;  // The database is already open for business
	            }
	        }
	
	        if (mIsInitializing) {
	            throw new IllegalStateException("getReadableDatabase called recursively");
	        }
	
	        try {
	            return getWritableDatabase();
	        } catch (SQLiteException e) {
	            if (mName == null) throw e;  // Can't open a temp database read-only!
	            Log.e(TAG, "Couldn't open " + mName + " for writing (will try read-only):", e);
	        }
	
	        SQLiteDatabase db = null;
	        try {
	            mIsInitializing = true;
	            String path = mContext.getDatabasePath(mName).getPath();
	            db = SQLiteDatabase.openDatabase(path, mFactory, SQLiteDatabase.OPEN_READONLY,
	                    mErrorHandler);
	            if (db.getVersion() != mNewVersion) {
	                throw new SQLiteException("Can't upgrade read-only database from version " +
	                        db.getVersion() + " to " + mNewVersion + ": " + path);
	            }
	
	            onOpen(db);
	            Log.w(TAG, "Opened " + mName + " in read-only mode");
	            mDatabase = db;
	            return mDatabase;
	        } finally {
	            mIsInitializing = false;
	            if (db != null && db != mDatabase) db.close();
	        }
	    }

&emsp;&emsp;通过上面的分析可以写出一个自定义的Context类，该类继承Context即可，但由于Context中有除了openOrCreateDatabase方法以外的其它抽象函数，所以建议使用非抽象类ContextWrapper，该类继承自Context，自定义的DatabaseContext类源码如下：

		public class DatabaseContext extends ContextWrapper {
		public DatabaseContext(Context context){
	        super( context );
	    }
	 
	    /**
	     * 获得数据库路径，如果不存在，则创建对象对象
	     * @param    name
	     * @param    mode
	     * @param    factory
	     */
	    @Override
	    public File getDatabasePath(String name) {
	        //判断是否存在sd卡
	        boolean sdExist = android.os.Environment.MEDIA_MOUNTED.equals(android.os.Environment.getExternalStorageState());
	        if(!sdExist){//如果不存在,
	            return null;
	        }else{//如果存在
	            //获取sd卡路径
	            String dbDir= FileUtils.getFlashBPath();
	            dbDir += "DB";//数据库所在目录
	            String dbPath = dbDir+"/"+name;//数据库路径
	            //判断目录是否存在，不存在则创建该目录
	            File dirFile = new File(dbDir);
	            if(!dirFile.exists()){
	            	dirFile.mkdirs();
	            }
	            
	            //数据库文件是否创建成功
	            boolean isFileCreateSuccess = false; 
	            //判断文件是否存在，不存在则创建该文件
	            File dbFile = new File(dbPath);
	            if(!dbFile.exists()){
	                try {                    
	                    isFileCreateSuccess = dbFile.createNewFile();//创建文件
	                } catch (IOException e) {
	                    e.printStackTrace();
	                }
	            }else{
	            	isFileCreateSuccess = true;
	            }
	            
	            //返回数据库文件对象
	            if(isFileCreateSuccess){
	            	return dbFile;
	            }else{
	            	return null;
	            }
	        }
	    }
	 
	    /**
	     * 重载这个方法，是用来打开SD卡上的数据库的，android 2.3及以下会调用这个方法。
	     * 
	     * @param    name
	     * @param    mode
	     * @param    factory
	     */
	    @Override
	    public SQLiteDatabase openOrCreateDatabase(String name, int mode, SQLiteDatabase.CursorFactory factory) {
	        SQLiteDatabase result = SQLiteDatabase.openOrCreateDatabase(getDatabasePath(name), null);
	        return result;
	    }
	    
	    /**
	     * Android 4.0会调用此方法获取数据库。
	     * 
	     * @see android.content.ContextWrapper#openOrCreateDatabase(java.lang.String, int, 
	     *              android.database.sqlite.SQLiteDatabase.CursorFactory,
	     *              android.database.DatabaseErrorHandler)
	     * @param    name
	     * @param    mode
	     * @param    factory
	     * @param     errorHandler
	     */
	    @Override
	    public SQLiteDatabase openOrCreateDatabase(String name, int mode, CursorFactory factory, DatabaseErrorHandler errorHandler) {
	        SQLiteDatabase result = SQLiteDatabase.openOrCreateDatabase(getDatabasePath(name), null);
	        
	        return result;
	    }
	}

&emsp;&emsp;在继承SQLiteOpenHelper的子类的构造函数中，用DatabaseContext的实例替代context即可：
	
	DatabaseContext dbContext = new DatabaseContext(context);
	super(dbContext, mDatabaseName, null, VERSION);

**说明：**本文很大程度上参考了网络上的其它文章，在此特别感谢这些文章的作者。

**参考资料：**

- [Android中使用SQLiteOpenHelper管理SD卡中的数据库](http://www.cnblogs.com/esrichina/p/3347036.html)