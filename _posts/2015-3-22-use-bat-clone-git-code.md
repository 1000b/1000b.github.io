---
layout: post
title: 通过批处理批量clone代码
category: Android
tags: Android
keywords: git,批处理，版本管理，clone
description: 通过批处理批量clone代码
---

&emsp;&emsp;开发的项目中，通常都是一个主工程依赖若干工程和jar包，我又习惯于将本地工作空间和本地版本管理用不同文件夹分开，比如在用Eclipse开发、git管理代码时，Eclipse的Workspace和git本地文件夹并不是同一个文件夹，只是在需要同步代码时我会通过BeyondCompare将Eclipse工作空间改动的代码同步到git本地文件夹，再add和push至服务器，这样做的好处在于即使Eclipse的工作空间发生意外（个人实际经历，Eclipse调皮的时候有很多，各种问题）也不会影响到git本地代码，我在代码管理的整个过程就是这样，下面说一说在代码管理过程中减少重复工作的方法。

&emsp;&emsp;由于我是将Eclipse和git分开，所以首先需要通过命令行或者Tortoise Git克隆服务器上的代码到本地，因为工程依赖关系太多，一个一个地clone、push太影响效率了，所以我会用批处理来实现过程，具体操作步骤是：

- 用批处理列出来Eclipse工作空间的工程列表，这个用批处理实现起来很简单，通过下面这一句话就可以列出一个文件夹下的所有文件和文件夹了：

		dir /b /on >list.txt

- 用批处理，clone上一步列出的所有工程：

		E:
	
		cd E:\git\project
		
		@echo downloading 工程1
		git clone -b 分支名称 git服务器地址
		
		@echo downloading 工程2
		git clone -b 分支名称 git服务器地址
	
		@echo downloading 工程3
		git clone -b 分支名称 git服务器地址

- Eclipse各工程有代码更新时，将更新通过BeyondCompare同步到git本地文件夹，然后push代码到服务器。

&emsp;&emsp;由于push的时候需要写日志，用脚本操作会相对麻烦些，如果想通过脚本将代码版本管理自动化，用Python会比批处理更好。

