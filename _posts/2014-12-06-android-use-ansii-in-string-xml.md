---
layout: post
title: 在Android的string.xml中使用转义字符实现想要的显示效果
category: Android
tags: Android
keywords: Android，转义字符，string.xml，空格
description: 在Android的string.xml中使用转义字符实现想要的显示效果
---

&emsp;&emsp;今天在做一个拼音、汉字双行显示的功能，需要下方的汉字和上方的拼音对应，于是我在string.xml文件中通过空格来实现对齐效果，结果发现无论汉字之间敲多少个空格显示出来的始终只空了一格，研究发现可以通过在string.xml中使用转义字符实现我要的效果。

&emsp;&emsp;我们在string.xml中用得最多的转义字符就是"\n"换行符了，其实还有很多可以在string.xml中使用的转义字符，下面是ANSII码表：

![ANSII码表](http://img.my.csdn.net/uploads/201412/06/1417841399_7128.jpg)


&emsp;&emsp;其中下列字符是可以使用在string.xml文件中的(没有亲自测试的字符就不列出来了，使用的时候可以自己尝试，markdown也支持转义字符，只能截图了。。。)：

![转义字符](http://img.my.csdn.net/uploads/201412/06/1417842137_3894.jpg)


&emsp;&emsp;比如我在string.xml文件中使用了空格转义字符：

![代码](http://img.my.csdn.net/uploads/201412/06/1417842396_8911.jpg)

&emsp;&emsp;显示效果如下：

![转义字符显示效果](http://img.my.csdn.net/uploads/201412/06/1417842396_7108.jpg)
