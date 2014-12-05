---
layout: post
title: 修改ViewPager调用setCurrentItem时，滑屏的速度
category: Android
tags: Android
keywords: Android，setCurrentItem，滑动速度
description: 修改ViewPager调用setCurrentItem时，滑屏的速度
---

&emsp;&emsp;在使用ViewPager的过程中，有需要直接跳转到某一个页面的情况，这个时候就需要用到ViewPager的setCurrentItem方法了，它的意思是跳转到ViewPager的指定页面，但在使用这个方法的时候有个问题，跳转的时候有滑动效果，当需要从当前页面跳转到其它页面时，跳转页面跨度过大、或者ViewPager每个页面的视觉效果相差较大时，通过这种方式实现ViewPager跳转显得很不美观，怎么办呢，我们可以去掉在使用ViewPager的setCurrentItem方法时的滑屏速度，具体实现如下：

## 自定义一个Scroll类，用于控制ViewPager滑动速度：

	import android.content.Context;
	import android.view.animation.Interpolator;
	import android.widget.Scroller;
	
	public class FixedSpeedScroller extends Scroller {
		private int mDuration = 0;
	
	    public FixedSpeedScroller(Context context) {
	        super(context);
	    }
	
	    public FixedSpeedScroller(Context context, Interpolator interpolator) {
	        super(context, interpolator);
	    }
	
	    public FixedSpeedScroller(Context context, Interpolator interpolator, boolean flywheel) {
	        super(context, interpolator, flywheel);
	    }
	
	
	    @Override
	    public void startScroll(int startX, int startY, int dx, int dy, int duration) {
	        super.startScroll(startX, startY, dx, dy, mDuration);
	    }
	
	    @Override
	    public void startScroll(int startX, int startY, int dx, int dy) {
	        super.startScroll(startX, startY, dx, dy, mDuration);
	    }
	}

## 在初始化ViewPager时，对ViewPager作如下设置：
		/**
	     * 设置ViewPager的滑动速度
	     * 
	     * */
	    private void setViewPagerScrollSpeed( ){
	    	try {
	            Field mScroller = null;
	            mScroller = ViewPager.class.getDeclaredField("mScroller");
	            mScroller.setAccessible(true); 
	            FixedSpeedScroller scroller = new FixedSpeedScroller( mViewPager.getContext( ) );
	            mScroller.set( mViewPager, scroller);
	        }catch(NoSuchFieldException e){
	        	
	        }catch (IllegalArgumentException e){
	        	
	        }catch (IllegalAccessException e){
	        	
	        }
	    }

&emsp;&emsp;运行代码后你就发现，它是直接跳转，没有滑屏效果了。