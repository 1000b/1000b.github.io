---
layout: post
title: Android手写优化-更为平滑的签名效果实现
category: Android
tags: Android
keywords: Android，手写，signatures
description: Android手写优化-更为平滑的签名效果实现
---

## 前言

&emsp;&emsp;这是一篇翻译至[squareup](http://corner.squareup.com/)的文章，这是[原文](http://corner.squareup.com/2012/07/smoother-signatures.html)，之前有人在TIEYE上翻译过这篇文章，但现在链接已经失效，手写效率问题是一直是Android平台上一个比较棘手的问题，所以有必要将这篇文章带给Android开发者，这篇文章在ITEYE那篇译文的基础上有所改动，如果英语还可以，请尽量阅读原文。

## 正文

&emsp;&emsp;在[上一篇文章](http://zmywly8866.github.io/2014/12/29/android-smooth-signatures.html)中，我们讨论了Square如何在Android设备上把签名效果做的平滑。在最新发布的Android版Square Card Reader应用中，我们将签名效果更上一层楼，更平滑，更美观，响应更灵敏！

主要通过以下三个方面来改进用户体验效果：

- 使用改进的曲线算法;

- 笔划粗细变化;

- 以bitmap缓存提升响应能力。

&emsp;&emsp;*曲弧之美:*

&emsp;&emsp;当你在屏幕滑动手指进行签名时，Android将一序列的触屏事件传递给Square客户端，每个触屏事件都包含独立的 (x,y) 坐标。要创建出一个签名的图像，客户端需要重建这些采样触点之间的连接线段。计算连接一序列离散点之间连接线段的过程，称为[样条插值](http://en.wikipedia.org/wiki/Spline_interpolation)。

&emsp;&emsp;最简单的样条插值策略是直线连接每一对触点。这也是之前版本的Square客户端采用的策略。

![Splining0](http://img.my.csdn.net/uploads/201412/29/1419850574_5860.png)

&emsp;&emsp;可以看到，即使有足够多的触点去模拟签名的曲线，线性插值方法呈现的效果仍显得又硬又挫。仔细观察图中的签名曲线，可以发现连接线在触点处出现了硬角，原本应该是外圆弧状的地方呈现出难看的扁平状。

![Splining1](http://img.my.csdn.net/uploads/201412/29/1419850575_2894.png)

&emsp;&emsp;问题原因在于，用户签名时手指并不是直愣愣地作点到点直线划动，更多情况下是曲线式的移动。但我们的SignatureView只能捕捉到签名过程中的采样点，再通过猜测采样点间连线来模拟用户签名的完整轨迹。显然，直线连接并不是很好的模拟。

&emsp;&emsp;这里较为合适的一个插值方法是曲线拟合。我们发现[三次Bezier插值曲线](http://en.wikipedia.org/wiki/B%C3%A9zier_curve#Cubic_B.C3.A9zier_curves)是最理想的插值算法。我们能够利用Bezier控制点精确地确定曲线形状，更赞的是我们能够在网上轻松地找到很多高效的Bezier曲线绘制算法。

&emsp;&emsp;Bezier曲线绘制算法需要输入一组用于生成曲线的控制点，但我们目前得到的只有在曲线上的采样点本身，没有Bezier控制点。由此，我们的样条插值计算归结为，利用现有的采样触点，计算出一组用来作为Bezier绘制算法输入的控制点，画出目标曲线。

&emsp;&emsp;这里对平滑的三次方曲线绘制的相关数学知识不作详细讨论。有兴趣的朋友可以阅读[Kirby Baker的UCLA计算机课程讲义](http://www.math.ucla.edu/~baker/149.1.02w/handouts/dd_splines.pdf)。

&emsp;&emsp;完成了从线性插值到曲线插值，乍看差异很细微，但整体的圆滑效果提升却相当明显。

![Splining2](http://img.my.csdn.net/uploads/201412/29/1419850573_9635.png)

&emsp;&emsp;*笔划粗细变化:*

&emsp;&emsp;如果你仔细研究下写在纸上的手写签名，不难发现笔划的粗细并不是一成不变的。笔划的粗细是随着笔的速度和用力程度而改变的。尽管Android提供了一个跟踪触屏力度的API，但其效果并没有达到我们用于签名所需的灵敏度与连贯性。还好，跟踪笔划速度是可以实现的，我们仅需要将每个触点的采集时间作tag标记，然后就可以计算点到点之间的速度了。

	public class Point {
	  private final float x;
	  private final float y;
	  private final long timestamp;
	  // ...
	
	  public float velocityFrom(Point start) {
	    return distanceTo(start) / (this.time - start.time);
	  }
	}

&emsp;&emsp;由于我们的绘制了签名的每个Bezier曲线，笔划的粗细依据可为每段曲线的起止点间的速度。

	lastVelocity = initialVelocity;
	lastWidth = intialStrokeWidth;
	
	public void addPoint(Point newPoint) {
	  points.add(newPoint);
	  Point lastPoint = points.get(points.size() - 1);
	  Bezier bezier = new Bezier(lastPoint, newPoint);
	
	  float velocity = newPoint.velocityFrom(lastPoint);
	
	  // A simple lowpass filter to mitigate velocity aberrations.
	  velocity = VELOCITY_FILTER_WEIGHT * velocity
	      + (1 - VELOCITY_FILTER_WEIGHT) * lastVelocity;
	
	  // The new width is a function of the velocity. Higher velocities
	  // correspond to thinner strokes.
	  float newWidth = strokeWidth(velocity);
	
	  // The Bezier's width starts out as last curve's final width, and
	  // gradually changes to the stroke width just calculated. The new
	  // width calculation is based on the velocity between the Bezier's
	  // start and end points.
	  addBezier(bezier, lastWidth, newWidth);
	
	  lastVelocity = velocity;
	  lastWidth = strokeWidth;
	}

&emsp;&emsp;当我们动手实现的时候，却碰到了一个棘手的问题——Android的canvas API没有绘制曲线宽度可变的Bezier曲线的相关方法。这意味着我们必需以点成线，自己点画出目标曲线。

	/** Draws a variable-width Bezier curve. */
	public void draw(Canvas canvas, Paint paint, float startWidth, float endWidth) {
	  float originalWidth = paint.getStrokeWidth();
	  float widthDelta = endWidth - startWidth;
	
	  for (int i = 0; i < drawSteps; i++) {
	    // Calculate the Bezier (x, y) coordinate for this step.
	    float t = ((float) i) / drawSteps;
	    float tt = t * t;
	    float ttt = tt * t;
	    float u = 1 - t;
	    float uu = u * u;
	    float uuu = uu * u;
	
	    float x = uuu * startPoint.x;
	    x += 3 * uu * t * control1.x;
	    x += 3 * u * tt * control2.x;
	    x += ttt * endPoint.x;
	
	    float y = uuu * startPoint.y;
	    y += 3 * uu * t * control1.y;
	    y += 3 * u * tt * control2.y;
	    y += ttt * endPoint.y;
	
	    // Set the incremental stroke width and draw.
	    paint.setStrokeWidth(startWidth + ttt * widthDelta);
	    canvas.drawPoint(x, y, paint);
	  }
	
	  paint.setStrokeWidth(originalWidth);
	}

&emsp;&emsp;可以看到，笔划粗细变化的签名，更加接近真实的手写效果。

![Variable Stroke Width](http://img.my.csdn.net/uploads/201412/29/1419850573_9869.png)

&emsp;&emsp;*响应能力:*

&emsp;&emsp;影响一个签名过程愉悦程度的另外一个重要因素是对输入的响应能力。使用纸笔签名时，笔的移动与纸上笔划出现是没有任何延迟的。而在触摸屏设备上，出现响应延迟在所难免。我们要做的是尽可能地减少这种延迟感，缩短用户手指在屏幕上滑动与签名笔划出现之间的时间间隔。

&emsp;&emsp;一种简单渲染策略是将所有的Bezier曲线在我们signatureView的onDraw()方法中绘制。

	@Override
	protected void onDraw(Canvas canvas) {
	  for (Bezier curve : signature) {
	    curve.draw(canvas, paint, curve.startWidth(), curve.endWidth());
	  }
	}

&emsp;&emsp;之前提到，我们绘制Bezierq曲线的方法是多次调用canvas.drawPoint(...)方法来以点成线。每个曲线重绘，对于笔划简单的签名还算可行，但对笔划较为复杂的签名则明显感觉到很慢。即使采用指定区域刷新的方法，绘制重叠线段仍然会严重拖慢签名响应。

&emsp;&emsp;解决方法是当签名每增加一个曲线时，将相应的Bezier曲线绘制到一个内存中的Bitmap中。之后只需要在onDraw()方法中画出该bitmap，而不需要在整个签名过程中对每条曲线重复运行Bezier曲线绘制算法。

	Bitmap bitmap = null;
	Canvas bitmapCanvas = null;
	
	private void addBezier(Bezier curve, float startWidth, float endWidth) {
	  if (bitmap == null) {
	    bitmap = Bitmap.createBitmap(getWidth(), getHeight(),
	        Bitmap.Config.ARGB_8888);
	    bitmapCanvas = new Canvas(bitmap);
	  }
	  curve.draw(bitmapCanvas, paint, startWidth, endWidth);
	}
	
	@Override protected void onDraw(Canvas canvas) {
	  canvas.drawBitmap(bitmap, 0, 0, paint);
	}

&emsp;&emsp;使用该方法能保证签名的绘制响应，不受签名复杂度的影响。

&emsp;&emsp;*最终成品:*

&emsp;&emsp;综上所述，我们采用了三次样条插值来使签名效果更平滑，基于笔划速度的笔划粗细可变效果使签名更真实，bitmap缓存使得绘制响应得到优化。最终的成果是用户能够得到一个愉悦的签名体验和一个漂亮的签名。

![final](http://img.my.csdn.net/uploads/201412/29/1419850574_9632.png)

**注：**在github上可以下载到项目的源码：

[android-signaturepad](https://github.com/gcacace/android-signaturepad)
