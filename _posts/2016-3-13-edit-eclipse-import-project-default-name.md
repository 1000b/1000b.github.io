---
layout: post
title: 修改Eclipse导入项目的默认工程名
category: Android
tags: Android
keywords: Android,Eclipse,Import Project Name,import
description: 修改Eclipse导入项目的默认工程名
---

&emsp;&emsp;在Eclipse中导入工程时，有时候导入的工程名不是我们想要的工程名，还需要手动修改，如果同时导入多个工程，还有可能存在多个工程名重复的情况导致不能一次性导入工程，eg，这里的New Project Name显然不是我们想要在Eclipse中显示的工程名：

![Eclipse_Import_Project](http://ww1.sinaimg.cn/large/6d17e381gw1f1v825bmxfj20et0f540t.jpg)

&emsp;&emsp;解决这个问题最好的方式是，在需要导入的工程中新建一个".project"隐藏文件，并在这个文件中指定工程的名称，这样就能够避免导入工程时还需要手动修改名字的问题：

	<?xml version="1.0" encoding="UTF-8"?>
	<projectDescription>
		// 在这里指定工程的名次
		<name>LocalAppSearch</name>
		<comment></comment>
		<projects>
		</projects>
		<buildSpec>
			<buildCommand>
				<name>com.android.ide.eclipse.adt.ResourceManagerBuilder</name>
				<arguments>
				</arguments>
			</buildCommand>
			<buildCommand>
				<name>com.android.ide.eclipse.adt.PreCompilerBuilder</name>
				<arguments>
				</arguments>
			</buildCommand>
			<buildCommand>
				<name>org.eclipse.jdt.core.javabuilder</name>
				<arguments>
				</arguments>
			</buildCommand>
			<buildCommand>
				<name>com.android.ide.eclipse.adt.ApkBuilder</name>
				<arguments>
				</arguments>
			</buildCommand>
		</buildSpec>
		<natures>
			<nature>com.android.ide.eclipse.adt.AndroidNature</nature>
			<nature>org.eclipse.jdt.core.javanature</nature>
		</natures>
	</projectDescription>

&emsp;&emsp;上面这种是最完美的方式，如果是开源项目最应该这么干，因为别人导入你项目的时候会方便很多，不需要手动修改工程名称(github上的Eclipse项目，导入的时候很多都需要手动修改工程名称，太不人性化了)。当然还有两种在导入工程时修改工程名的方式。

- 在如上图的导入工程界面，选中某一项，双击"Project to Import"，右边的"New Project Name"就处于编辑状态了，可以直接在这里修改；

- 将工程导入到Eclipse中后修改，选中工程，直接按F2，修改即可。

	






