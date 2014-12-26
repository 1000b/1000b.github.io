---
layout: post
title: 在Android library中不能使用switch-case语句访问资源ID的原因分析及解决方案
category: Android
tags: Android
keywords: Android，library，lib project，resource
description: 在Android library中不能使用switch-case语句访问资源ID的原因分析及解决方案
---

## 原因分析

&emsp;&emsp;当我们在Android依赖库中使用switch-case语句访问资源ID时会报如下图所示的错误，报的错误是case分支后面跟的参数必须是常数，换句话说出现这个问题的原因是Android library中生成的R.java中的资源ID不是常数：

![library error](http://img.my.csdn.net/uploads/201412/26/1419564510_7687.jpg)

&emsp;&emsp;打开library中的R.java，发现确实如此，每一个资源ID都没有被声明为final：

![library R.java](http://img.my.csdn.net/uploads/201412/26/1419564512_3649.jpg)

&emsp;&emsp;但是当你打开你的主工程，在onClick、onItemClick等各种回调方法中是可以通过switch-case语句来访问资源ID的，因为在主工程的R.java中资源ID都被声明为了final常量。

&emsp;&emsp;project中能够通过switch-case语句正常引用资源ID：

![project right](http://img.my.csdn.net/uploads/201412/26/1419564512_5840.jpg)

&emsp;&emsp;project中的R.java：

![project R.java](http://img.my.csdn.net/uploads/201412/26/1419564513_5678.jpg)

## 解决方案

&emsp;&emsp;既然是由于library的R.java中的资源ID不是常量引起的，我们可以在library中通过if-else-if条件语句来引用资源ID，这样就避免了这个错误：

![library use](http://img.my.csdn.net/uploads/201412/26/1419564512_5822.jpg)

## 参考资料：

&emsp;&emsp;为了进一步了解问题的具体原因，在万能的StackOverflow上还真搜到了这个问题：

> In a regular Android project, constants in the resource R class are declared like this:
> 
> 	public static final int main=0x7f030004;
> 	
> However, as of ADT 14, in a library project, they will be declared like this:
> 	
> 	public static int main=0x7f030004;
> 	
> In other words, the constants are not final in a library project. Therefore your code would no longer compile.
> 	
> The solution for this is simple: Convert the switch statement into an if-else statement.
> 	
> 	public void onClick(View src)
> 	{
> 	    int id = src.getId();
> 	    if (id == R.id.playbtn){
> 	        checkwificonnection();
> 	    } else if (id == R.id.stopbtn){
> 	        Log.d(TAG, "onClick: stopping srvice");
> 	        Playbutton.setImageResource(R.drawable.playbtn1);
> 	        Playbutton.setVisibility(0); //visible
> 	        Stopbutton.setVisibility(4); //invisible
> 	        stopService(new Intent(RakistaRadio.this,myservice.class));
> 	        clearstatusbar();
> 	        timer.cancel();
> 	        Title.setText(" ");
> 	        Artist.setText(" ");
> 	    } else if (id == R.id.btnmenu){
> 	        openOptionsMenu();
> 	    }
> 	}
> http://tools.android.com/tips/non-constant-fields
> 	
> Tip
> You can quickly convert a switch statement to an if-else statement using Eclipse's quick fix.
> 	
> Click on the switch keyword and press Ctrl + 1 then select
> 	
> Convert 'switch' to 'if-else'.

&emsp;&emsp;问题详见：[switch case statement error: case expressions must be constant expression](http://stackoverflow.com/questions/9092712/switch-case-statement-error-case-expressions-must-be-constant-expression)







	