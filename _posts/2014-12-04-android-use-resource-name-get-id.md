---
layout: post
title: Android通过资源文件名获取资源ID
category: Android
tags: Android
keywords: Android，Resource，raw，drawable
description: Android通过资源文件名获取资源ID
---
## 引言
&emsp;&emsp;当我们需要读取资源文件中一组命名有规则的资源文件时，通常我们有两种方式：

1. 在res/values/arrays.xml定义一个资源数组，然后在代码中通过**TypedArray**去获取数组里面的资源ID；
2. 在代码中申请一个资源id数据，然后将资源ID一一列出来。

&emsp;&emsp;上述两种方式中，第一种方式在数组比较大时，加载的时候会很耗时；第二种方式使得代码看起来很乱。本文要介绍的是另外一种方式，在代码中通过资源文件名来获取资源的ID。

## 实现
&emsp;&emsp;Resource.java提供了一个方法**getIdentifier**，通过这个方法可以实现资源文件名和资源ID之间的转换，该方法的详细介绍如下：

	/**
     * Return a resource identifier for the given resource name.  A fully
     * qualified resource name is of the form "package:type/entry".  The first
     * two components (package and type) are optional if defType and
     * defPackage, respectively, are specified here.
     * 
     * <p>Note: use of this function is discouraged.  It is much more
     * efficient to retrieve resources by identifier than by name.
     * 
     * @param name The name of the desired resource.
     * @param defType Optional default resource type to find, if "type/" is
     *                not included in the name.  Can be null to require an
     *                explicit type.
     * @param defPackage Optional default package to find, if "package:" is
     *                   not included in the name.  Can be null to require an
     *                   explicit package.
     * 
     * @return int The associated resource identifier.  Returns 0 if no such
     *         resource was found.  (0 is not a valid resource ID.)
     */
    public int getIdentifier(String name, String defType, String defPackage) {
        try {
            return Integer.parseInt(name);
        } catch (Exception e) {
            // Ignore
        }
        return mAssets.getResourceIdentifier(name, defType, defPackage);
    }

&emsp;&emsp;实际上它是引用的Asserts.java类中的**getResourceIdentifier**方法，这是一个native方法，具体实现被打包在库中了。

&emsp;&emsp;通过下面这个封装的类即可实现资源文件名到资源ID的转换，使用的时候按照注释说明传入相应地参数就可以了：

	package com.uperone.resources;
	
	import android.content.Context;
	import android.text.TextUtils;
	
	public class ResourcesUtils {
		private ResourcesUtils( ){
			
		}
		
		/**
		 * use resouce name to get resource id
		 * @param context
		 * @param resourceName the resouce name that you want to get it's id
		 * @param type resource type,contains:
		 * @return -1 illegal input params, !=-1 resouce id
		 * 
		 * */
		public static int getResourceId( Context context, String resourcesName, String type ){
			if( null == context || TextUtils.isEmpty( resourcesName ) || TextUtils.isEmpty( type ) ){
				return -1;
			}
			
			return context.getResources( ).getIdentifier(resourcesName, type, context.getPackageName( ) );
		}
		
		public static final String TYPE_DRAWABLE = "drawable";
		public static final String TYPE_RAW = "raw";
	}

## 类似文章
[**Android根据资源名获取资源ID**](http://droidyue.com/blog/2014/09/12/get-resource-id-by-name-in-android/)



