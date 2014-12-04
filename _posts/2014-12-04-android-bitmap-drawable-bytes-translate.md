---
layout: post
title: Android中Bitmap、Drawable、bytes数组之间相互转换
category: Android
tags: Android
keywords: Android，bitmap，drawable，bitmapdrawable，byte数组
description: Android中Bitmap、Drawable、bytes数组之间相互转换
---

	package com.uperone.resources;
	
	import java.io.ByteArrayOutputStream;
	
	import android.content.Context;
	import android.graphics.Bitmap;
	import android.graphics.BitmapFactory;
	import android.graphics.Canvas;
	import android.graphics.PixelFormat;
	import android.graphics.drawable.BitmapDrawable;
	import android.graphics.drawable.Drawable;
	
	public class BitmapUtils {
		private BitmapUtils( ){
			
		}
		
		/**
		 * bitmap -> bytes
		 * 
		 * */
		public static byte[] bitmap2Bytes( Bitmap bitmap ){
			if( null == bitmap ){
				return null;
			}
			
			ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream( );    
			bitmap.compress( Bitmap.CompressFormat.PNG, 100, byteArrayOutputStream );
		    
		    return byteArrayOutputStream.toByteArray( );
		}
		
		/**
		 * bytes -> bitmap
		 * 
		 * */
		public static Bitmap bytes2Bitmap( byte[] bytes ){
			if( null == bytes || bytes.length == 0 ){
				return null;
			}
			
			return BitmapFactory.decodeByteArray( bytes, 0, bytes.length ); 
		}
		
		/**
		 * bitmap -> drawable
		 * 
		 * */
		public static Drawable bitmap2Drawable( Context context, Bitmap bitmap ){
			if( null == context || null == bitmap ){
				return null;
			}
			
			return new BitmapDrawable( context.getResources( ), bitmap );  
		}
		
		/**
		 * drawable -> bitmap
		 * 
		 * */
		public static Bitmap drawable2Bitmap( Drawable drawable ){
			if( null == drawable ){
				return null;
			}
			
			Bitmap bitmap = Bitmap.createBitmap(
					drawable.getIntrinsicWidth(),
					drawable.getIntrinsicHeight(),
					drawable.getOpacity() != PixelFormat.OPAQUE ? Bitmap.Config.ARGB_8888 : Bitmap.Config.RGB_565);  
			Canvas canvas = new Canvas( bitmap );  
			drawable.setBounds( 0, 0, drawable.getIntrinsicWidth( ), drawable.getIntrinsicHeight( ) );  
			drawable.draw( canvas );  
			
			return bitmap;  
		}
	}



