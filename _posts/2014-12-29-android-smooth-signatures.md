---
layout: post
title: Android手写优化-平滑的签名效果实现
category: Android
tags: Android
keywords: Android，手写，signatures
description: Android手写优化-平滑的签名效果实现
---

## 前言

&emsp;&emsp;这是一篇翻译至[http://corner.squareup.com/](http://corner.squareup.com/2010/07/smooth-signatures.html)的文章，这是[原文](http://corner.squareup.com/2010/07/smooth-signatures.html)，之前有人在TIEYE上翻译过这篇文章，但现在链接已经失效，手写效率问题是一直是Android平台上一个比较棘手的问题，所以有必要将这篇文章带给Android开发者，这篇文章在ITEYE那篇译文的基础上有所改动，如果英语还可以，请尽量阅读原文。

## 正文

&emsp;&emsp;在信用卡支付流程中，使用手写签名能够提高支付的安全性，并有效降低过程成本。使用Square在手机上进行支付，用户可以用手指在屏幕上签名，无需拿出笔来在收据上签字。

![after](http://img.my.csdn.net/uploads/201412/29/1419827547_3486.png)

&emsp;&emsp;**提示：**该界面中提供了手机摇一摇清屏的功能

&emsp;&emsp;用户在该界面提供的签名，将签署在电子邮件收据中，以帮助Square监测和防止消费欺诈。

&emsp;&emsp;下面我们尝试在Android客户端上实现该界面，先尝试从最简单可行的方式开始：生成一个自定义View，能够监听触屏事件，并根据触摸路径画出点。

	public class SignatureView extends View {
	  private Paint paint = new Paint();
	  private Path path = new Path();
	
	  public SignatureView(Context context, AttributeSet attrs) {
	    super(context, attrs);
	
	    paint.setAntiAlias(true);
	    paint.setColor(Color.BLACK);
	    paint.setStyle(Paint.Style.STROKE);
	    paint.setStrokeJoin(Paint.Join.ROUND);
	    paint.setStrokeWidth(5f);
	  }
	
	  @Override
	  protected void onDraw(Canvas canvas) {
	    canvas.drawPath(path, paint);
	  }
	
	  @Override
	  public boolean onTouchEvent(MotionEvent event) {
	    float eventX = event.getX();
	    float eventY = event.getY();
	
	    switch (event.getAction()) {
	      case MotionEvent.ACTION_DOWN:
	        path.moveTo(eventX, eventY);
	        return true;
	      case MotionEvent.ACTION_MOVE:
	      case MotionEvent.ACTION_UP:
	        path.lineTo(eventX, eventY);
	        break;
	      default:
	        return false;
	    }
	
	    // Schedules a repaint.
	    invalidate();
	    return true;
	  }
	}

&emsp;&emsp;可以看到实现出来的效果还是与预期有一定差距的，签名的笔画有锯齿，并且明显反应迟钝，笔迹跟不上手指。

![before](http://img.my.csdn.net/uploads/201412/29/1419827546_1843.png)

&emsp;&emsp;下面我们尝试从两个不同的途径来解决上面的问题。

&emsp;&emsp;**触屏事件丢失**

&emsp;&emsp;笔迹跟不上手指这个问题，可能的原因是：

- Android对触屏事件的采样率过低；

- 绘制事件阻塞了触屏事件的采样；

&emsp;&emsp;幸运的是，经过实验考证，问题并不是这两个原因导致的。同时，我们发现Android对触屏事件进行批量处理。传递给onTouchEvent()的每一个MotionEvent都包含上至前一个onTouchEvent()调用之间捕获的若干个坐标点。如果将这些点都加入到绘制中，可使签名效果更加平滑。

&emsp;&emsp;隐藏的坐标数组可以通过MotionEvent类的下列方法获取：

- ·getHistorySize()

- ·getHistoricalX(int)

- ·getHistoricalY(int)

&emsp;&emsp;下面我们利用这些方法，将中间点包含进SignatureView的绘制：

	public class SignatureView extends View {
	  public boolean onTouchEvent(MotionEvent event) {
	    ...
	    switch (event.getAction()) {
	      case MotionEvent.ACTION_MOVE:
	      case MotionEvent.ACTION_UP:
	
	        // When the hardware tracks events faster than they are delivered,
	        // the event will contain a history of those skipped points.
	        int historySize = event.getHistorySize();
	        for (int i = 0; i < historySize; i++) {
	          float historicalX = event.getHistoricalX(i);
	          float historicalY = event.getHistoricalY(i);
	          path.lineTo(historicalX, historicalY);
	        }
	
	        // After replaying history, connect the line to the touch point.
	        path.lineTo(eventX, eventY);
	        break;
	    ...
	  }
	}

&emsp;&emsp;这个简单的改进，使签名效果外观有了显著的提升。但该View对用户触屏的响应仍然迟钝。

&emsp;&emsp;**局部刷新**

&emsp;&emsp;我们的SignatureView在每一次调用onTouchEvent()时，会在触屏坐标之间画线，并进行全屏刷新，但即使只是很小的像素级变动，也需要全屏重绘。

&emsp;&emsp;显然，全屏重绘效率低下且没有必要。我们可以使用 View.invalidate(Rect) 方法，选择性地对新添画线的矩形区域进行局部刷新，可以显著提高绘制性能。

&emsp;&emsp;采用的算法思路如下：

- 创建一个代表脏区域的矩形；

- 获得 ACTION_DOWN 事件的 X 与 Y 坐标，用来设置矩形的顶点；

- 获得 ACTION_MOVE 和 ACTION_UP 事件的新坐标点，加入到矩形中使之拓展开来（别忘了上文说过的历史坐标点）；

- 刷新脏区域。

&emsp;&emsp;采用该算法后，我们能够明显感觉到触屏响应性能的大幅提升。

&emsp;&emsp;以上我们对SignatureView进行了两方面的改造提升：

- 将触屏事件的中间点加入绘制，使笔画更加流畅逼真；

- 以局部刷新取代全屏刷新，提高绘图性能，使触屏响应更加迅速。

&emsp;&emsp;最终的效果出炉：

![after](http://img.my.csdn.net/uploads/201412/29/1419827547_3486.png)

下面是SignatureView的最终完成代码，我们去掉了一些无关的方法（如摇动检测）

	public class SignatureView extends View {
	
	  private static final float STROKE_WIDTH = 5f;
	
	  /** Need to track this so the dirty region can accommodate the stroke. **/
	  private static final float HALF_STROKE_WIDTH = STROKE_WIDTH / 2;
	
	  private Paint paint = new Paint();
	  private Path path = new Path();
	
	  /**
	   * Optimizes painting by invalidating the smallest possible area.
	   */
	  private float lastTouchX;
	  private float lastTouchY;
	  private final RectF dirtyRect = new RectF();
	
	  public SignatureView(Context context, AttributeSet attrs) {
	    super(context, attrs);
	
	    paint.setAntiAlias(true);
	    paint.setColor(Color.BLACK);
	    paint.setStyle(Paint.Style.STROKE);
	    paint.setStrokeJoin(Paint.Join.ROUND);
	    paint.setStrokeWidth(STROKE_WIDTH);
	  }
	
	  /**
	   * Erases the signature.
	   */
	  public void clear() {
	    path.reset();
	
	    // Repaints the entire view.
	    invalidate();
	  }
	
	  @Override
	  protected void onDraw(Canvas canvas) {
	    canvas.drawPath(path, paint);
	  }
	
	  @Override
	  public boolean onTouchEvent(MotionEvent event) {
	    float eventX = event.getX();
	    float eventY = event.getY();
	
	    switch (event.getAction()) {
	      case MotionEvent.ACTION_DOWN:
	        path.moveTo(eventX, eventY);
	        lastTouchX = eventX;
	        lastTouchY = eventY;
	        // There is no end point yet, so don't waste cycles invalidating.
	        return true;
	
	      case MotionEvent.ACTION_MOVE:
	      case MotionEvent.ACTION_UP:
	        // Start tracking the dirty region.
	        resetDirtyRect(eventX, eventY);
	
	        // When the hardware tracks events faster than they are delivered, the
	        // event will contain a history of those skipped points.
	        int historySize = event.getHistorySize();
	        for (int i = 0; i < historySize; i++) {
	          float historicalX = event.getHistoricalX(i);
	          float historicalY = event.getHistoricalY(i);
	          expandDirtyRect(historicalX, historicalY);
	          path.lineTo(historicalX, historicalY);
	        }
	
	        // After replaying history, connect the line to the touch point.
	        path.lineTo(eventX, eventY);
	        break;
	
	      default:
	        debug("Ignored touch event: " + event.toString());
	        return false;
	    }
	
	    // Include half the stroke width to avoid clipping.
	    invalidate(
	        (int) (dirtyRect.left - HALF_STROKE_WIDTH),
	        (int) (dirtyRect.top - HALF_STROKE_WIDTH),
	        (int) (dirtyRect.right + HALF_STROKE_WIDTH),
	        (int) (dirtyRect.bottom + HALF_STROKE_WIDTH));
	
	    lastTouchX = eventX;
	    lastTouchY = eventY;
	
	    return true;
	  }
	
	  /**
	   * Called when replaying history to ensure the dirty region includes all
	   * points.
	   */
	  private void expandDirtyRect(float historicalX, float historicalY) {
	    if (historicalX < dirtyRect.left) {
	      dirtyRect.left = historicalX;
	    } else if (historicalX > dirtyRect.right) {
	      dirtyRect.right = historicalX;
	    }
	    if (historicalY < dirtyRect.top) {
	      dirtyRect.top = historicalY;
	    } else if (historicalY > dirtyRect.bottom) {
	      dirtyRect.bottom = historicalY;
	    }
	  }
	
	  /**
	   * Resets the dirty region when the motion event occurs.
	   */
	  private void resetDirtyRect(float eventX, float eventY) {
	
	    // The lastTouchX and lastTouchY were set when the ACTION_DOWN
	    // motion event occurred.
	    dirtyRect.left = Math.min(lastTouchX, eventX);
	    dirtyRect.right = Math.max(lastTouchX, eventX);
	    dirtyRect.top = Math.min(lastTouchY, eventY);
	    dirtyRect.bottom = Math.max(lastTouchY, eventY);
	  }
	}




	