---
layout: post
title: JXL自动换行的实现
category: Java
tags: Java
keywords: Java,JXL，Excel
description:  JXL自动换行的实现
---

&emsp;&emsp;Java语言中，操作Excel文件比较知名的库有：POI和JXL，我一直使用JXL，通过JXL写文件时，对于同一个单元格内容需要换行显示时直接在字符串后面加上"\n"是不能达到效果的，必须通过WritableCellFormat来完成该功能，具体实现如下：
	
	// 打开文件
	WritableWorkbook workBook = Workbook.createWorkbook( new File( filePath ) );
	// 创建sheet
	WritableSheet sheet0 = workBook.createSheet("详细评测信息", 0 );
	// 设置单元格格式
	WritableFont writableFont = new WritableFont(WritableFont.createFont("宋体"),11, WritableFont.NO_BOLD, false);
	WritableCellFormat writableCellFormat = new WritableCellFormat(writableFont);
	writableCellFormat.setWrap(true);
	// 操作单元格
	String content = "hello" + "\n" + "world" + "\n" + "!";
	sheet0.addCell( new Label(0, 0, content, writableCellFormat) );
	// 写文件
	workBook.write( );
	// 关闭文件
	workBook.close( );

&emsp;&emsp;在使用JXL的过程中，我总结了如下几点需要注意的地方：

- JXL只支持office2003及以前的版本，建议在读写文件的时候以后缀为xls文件格式的方式操作；

- JXL对单个excel文件的限制是在5M，对于超过5M的文件处理会比较麻烦，极有可能OOM；

- JXL中获取sheet表中的行数是返回有内容的总行数，有些单元格看上去没有内容但里面可能存在空格，所以对于需要操作的excel表格应该将多余的空行删掉；

- JXL对excel文件中的内容全部是按照U16-LE编码方式处理的，一定要注意excel文件的编码格式，否则有可能出现读取、写文件时出现乱码的现象，我的经验是使用WPS而不是office操作excel文件；

**参考资料：**

- [jxl的execl导出的相关设置(合并单元格,自动换行等)](http://blog.csdn.net/wzy126126/article/details/7228385)

- [Java Excel API](http://jexcelapi.sourceforge.net/)
