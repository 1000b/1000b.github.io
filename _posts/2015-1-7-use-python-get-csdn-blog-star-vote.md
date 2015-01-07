---
layout: post
title: 使用Python脚本拉取2014 CSDN博客之星投票情况
category: Python
tags: Python
keywords: Python,2014,CSDN,博客之星
description: 使用Python脚本拉取2014 CSDN博客之星投票情况
---

## 前言
&emsp;&emsp;最近在自学Python，正好2014 CSDN博客之星投票搞得如火如荼，拿来练练手。

- 环境：Win7 64位 Python 2.7；

- 用到了正则表达式、函数、写文件、urllib2；

- 没有用到线程；

- 程序也不怎么规范，但终归是能够达到目的了，哈哈。

## 源码

	# -*- coding: utf-8 -*-

	import urllib2;
	import re;
	import os;
	import thread;
	
	
	def loadBlogSort(url):
	    pageCount = getPageCount(url);
	    print 'pageCount == ',pageCount;
	    baseUrl = 'http://vote.blog.csdn.net/Blogstar2014/Selection?PageIndex=';
	    urlSuffix = '#content';
	
	    filepath = 'csdn_blog_star_vote.txt';
	    if os.path.exists(filepath):
	        os.remove(filepath);
	    f = open(filepath,'w+');
	    for pageIndex in range(1,int(pageCount)+1):
	        contentUrl = baseUrl + str(pageIndex) + urlSuffix;
	        print 'pageIndex == ',pageIndex, ' contentUrl == ',contentUrl;
	        user_agent = 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.95 Safari/537.36'
	        headers = { 'User-Agent' : user_agent }
	        request = urllib2.Request(contentUrl, headers = headers)
	        response = urllib2.urlopen(request);
	        result = response.read();
	        # unicodeResult = result.decode("utf-8");
	        # 名称
	        # <div\sclass=\"star-con\"><span\sclass=\"star-name\"><a\shref=(.+?)\starget=\"_blank\"\stitle=(.+?)>(.+?)</a></span>
	        names = re.findall('<div\sclass=\"star-con\"><span\sclass=\"star-name\"><a\shref=(.+?)\starget=\"_blank\"\stitle=(.+?)>(.+?)</a></span>',result,re.S);
	        nameList = [];
	        for name in names:
	            # print '昵称：',name[2];
	            nameList.append(name[ 2 ]);
	
	        # 博客地址
	        # <dt><a\shref=\"(.+?)\"\s\starget="_blank"><img\ssrc=(.+?)></a></dt>
	        blogUrlList = [];
	        detailUrls = re.findall('<dt><a\shref=\"(.+?)\"\s\starget="_blank"><img\ssrc=(.+?)></a></dt>',result,re.S);
	        for detailUrl in detailUrls:
	            blogUrlList.append(getBlogUrl(detailUrl[0]));
	
	        # 得票
	        # <p><b>得票：</b><span\sid=(.+?)>(.+?)</span></p>
	        votes = re.findall('<p><b>(.+?)</b><span\sid=(.+?)>(.+?)</span></p>',result,re.S);
	        voteList = [];
	        for vote in votes:
	            # print ' 得票：',str(vote[2]);
	            voteList.append(vote[ 2 ]);
	        # 博文浏览量、博文数、评论数
	        # <div\sclass="star-post1"><span>(.+?)</span><span>(.+?)</span><span>(.+?)</span></div>
	        infos = re.findall('<div\sclass="star-post1"><span>(.+?)</span><span>(.+?)</span><span>(.+?)</span></div>',result,re.S);
	        infoIndex = 0;
	        blankSize = 20;
	        for info in infos:
	            user = '昵称：'+nameList[infoIndex] + ( blankSize - len(nameList[infoIndex]) )*' '+'得票：'+voteList[infoIndex] + ( blankSize - len(voteList[infoIndex]) )*' '+'博文浏览量: '+str(info[0]) + ( blankSize - len(str(info[0])) )*' '+'博文数：'+str(info[1]) + ( blankSize - len(str(info[1])) )*' '+'评论数：'+str(info[2])+ + ( blankSize - len(str(info[2])) )*' '+'博客地址：' + blogUrlList[infoIndex]+ '\n'
	            # print user;
	            f.write(user);
	            infoIndex += 1;
	
	    f.close();
	    print '写文件完毕！';
	
	# 得到博客链接
	def getBlogUrl(detailUrl):
	    url = 'http://vote.blog.csdn.net/' + detailUrl;
	    user_agent = 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.95 Safari/537.36'
	    headers = { 'User-Agent' : user_agent }
	    request = urllib2.Request(url, headers = headers)
	    response = urllib2.urlopen(request);
	    result = response.read();
	    blogUrls = re.findall('<p>(.+?)<a\shref=\"(.+?)\"\s\starget="_blank">(.+?)</a></p>',result,re.S);
	    print 'blogUrl == ',url + '\n' + str(blogUrls[0][1]);
	    return str(blogUrls[0][1]);
	
	# 得到总页码数
	def getPageCount(url):
	    user_agent = 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.95 Safari/537.36'
	    headers = { 'User-Agent' : user_agent }
	    request = urllib2.Request(url, headers = headers)
	    response = urllib2.urlopen(request);
	    result = response.read();
	    pageCount = re.findall('<div\sid=\"PageCount\"\sstyle=\"\sdisplay:none\">(.+?)</div>',result,re.S);
	    return pageCount[0];
	
	url = 'http://vote.blog.csdn.net/Blogstar2014/Selection?PageIndex=1#content';
	loadBlogSort(url);

## 效果

![2014_csdn_blog_star_vote](http://ww1.sinaimg.cn/large/6d17e381gw1eo11bmy3wfj21840ngh2p.jpg)