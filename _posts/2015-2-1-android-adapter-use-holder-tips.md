---
layout: post
title: 在Adapter中使用Holder的那些坑
category: android
tags: android
keywords: android,BaseAdapter,Holder
description: 在Adapter中使用Holder的那些坑
---

&emsp;&emsp;在使用GridView、ListView时，通常会在Adapter中采用Holder缓存每一项以提高效率，但如果没有用好Holder，这个缓存机制会导致许多意想不到的问题，结合自己的经验特地总结一下，以免今后再犯。

## 内容错乱

&emsp;&emsp;在Adapter的getView方法中通过position更新每一项的内容，对于根据判断条件给每一项设置属性的情况，每个判断条件下都需要给每一项的每个属性赋值，否则在滑动ListView或GridView时会导致内容错乱，比如下面这段代码是在getView中调用的一个方法，如果在使用Holder时，只在if(position == mSelectPosition)分支下给项背景设置属性，在滑动过程中会导致本来不应该是蓝色背景的项背景变成蓝色了：

	private void setFileInfo( FileInfo fileInfo, int position ){
		mFileHolder.mFilePathTxtView.setText( fileInfo.getFilePath( ) );
		mFileHolder.mVoiceTypeTxtView.setText( fileInfo.getVoiceType( ) );
		mFileHolder.mFileItemLayout.setTag(position);
		
		if(position == mSelectPosition){
			mFileHolder.mFileItemLayout.setBackgroundColor( Color.BLUE );
		}else{
			// 每一项的else分值也需要为项设置背景，否则会在滑动过程中导致内容错乱
			mFileHolder.mFileItemLayout.setBackgroundColor( Color.RED );
		}
	}

&emsp;&emsp;这个问题虽然很容易处理，但在开发的过程中经常遇到，比如根据判断条件为每一项设置不同的背景、头像或者文本时，稍有不注意或者偷懒都会导致意想不到的bug，特别是对于每一项需要同时修改多个属性的时候就要更加注意了，一定要保证每个条件分值下都为项的每个视图都设置属性了。

## setTag
&emsp;&emsp;在上面那段代码中增加监听项点击事件，由于用到了缓存，所以不能通过getId来获取具体点击了哪一项，通常我们都是通过给每一项的layout设置tag，在监听回调中getTag来获取具体点击了哪一项，比如下面这样：

	private void setFileInfo( FileInfo fileInfo, int position ){
		mFileHolder.mFilePathTxtView.setText( fileInfo.getFilePath( ) );
		mFileHolder.mVoiceTypeTxtView.setText( fileInfo.getVoiceType( ) );

		// 为每一项setTag，在onClick方法中通过v.getTag()方法获取点击了哪一项
		mFileHolder.mFileItemLayout.setTag(position);
		mFileHolder.mFileItemLayout.setOnClickListener(new OnClickListener( ) {
			@Override
			public void onClick(View v) {
				int position = ( Integer )v.getTag( );
				mSelectPosition = position;
				if( null != mOnFileListOnClickListener ){
					mOnFileListOnClickListener.onItemClick(mFileInfoList.get(position));
				}
				
				notifyDataSetChanged( );
			}
		});
		
		if(position == mSelectPosition){
			mFileHolder.mFileItemLayout.setBackgroundColor( Color.BLUE );
		}else{
			mFileHolder.mFileItemLayout.setBackgroundColor( Color.RED );
		}
	}

&emsp;&emsp;如果按上述这种方式编码，在程序运行后点击某一项你会发现程序会挂掉，会报下面的异常：

	02-01 15:57:49.340: E/AndroidRuntime(1949): java.lang.ClassCastException: java.lang.Integer cannot be cast to com.uperone.view.FileListAdapter$FileHolder
	02-01 15:57:49.340: E/AndroidRuntime(1949): at com.uperone.view.FileListAdapter.getView(FileListAdapter.java:72)

&emsp;&emsp;具体原因也是因为Holder缓存导致的，因为在getView方法中，我是通过为convertView设置tag来做到Holder重用的：
	
	@Override
	public View getView(int position, View convertView, ViewGroup parent) {
		if( null == convertView ){
			convertView = mLayoutInflater.inflate( R.layout.list_file_item_layout, null);
			
			mFileHolder = new FileHolder();
			mFileHolder.mFileItemLayout = ( LinearLayout )convertView.findViewById(R.id.fileItemLayoutId);
			mFileHolder.mFilePathTxtView = ( TextView )convertView.findViewById(R.id.filePathTxtId);
			mFileHolder.mVoiceTypeTxtView = ( TextView )convertView.findViewById(R.id.voiceTypeTxtId);
			mFileHolder.mFileItemLayout.setClickable(true);
			convertView.setTag( mFileHolder );
		}else{
			mFileHolder = ( FileHolder )convertView.getTag( );
		}
		
		
		if( !isDataEmpty() ){
			setFileInfo( mFileInfoList.get( position ), position );
		}
		
		return convertView;
	}

&emsp;&emsp;在点击ListView中的某一项时，由于设置每一项内容的setFileInfo方法是在convertView.getTag()后调用的，所以在系统调用getView()方法更新ListView时，convertView取到的tag是mHolder.mFileItemLayout设置的tag，使用了Holder，就不能在每项中有多个视图设置tag和取tag。

&emsp;&emsp;对于上述的异常，通常的解决方式是在ListView实例化的地方，通过setOnItemClickListener方法监听每一项的点击，避免tag冲突的问题。







