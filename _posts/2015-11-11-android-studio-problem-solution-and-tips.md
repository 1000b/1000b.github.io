---
layout: post
title: Android Studio使用过程中遇到的一些问题及解决方案
category: Android
tags: Android
keywords: Android,Android Studio,Eclipse
description: Android Studio使用过程中遇到的一些问题及解决方案
---

&emsp;&emsp;由于之前的项目太复杂，主要是考虑到JNI在AS上编译不方便，还要考虑到项目进度，最近才从Eclipse转到AS，主要方案是AS中只引用jar包和so，JNI的编译还是在Eclipse中进行。这过程中遇到过很多问题，记录下来方便后续查阅，本文中遇到的所有问题都是在Windows系统下。

- assets文件的存放目录在"src/main/"目录下，和java、res文件夹平级；

- 引用libs文件夹中的so，需要在对应module下的build.gradle文件的android标签下加上如下属性：

		android {
		    sourceSets {
		        main {
		            jniLibs.srcDirs = ['libs']
		        }
		    }
		}

- android studio的编译时屏蔽掉lint检查，可以避免由于编译条件太过严格而编译不过的问题：

		lintOptions {
		    abortOnError false
		}

- 如果遇到多个jar包中的某个文件冲突，可以在对应module下的build.gradle文件的android标签下加上如下属性：

		packagingOptions {
		    exclude 'META-INF/NOTICE.txt'// 这里是具体的冲突文件全路径
		    exclude 'META-INF/LICENSE.txt'
		}

- 调整logcat文件显示的颜色：File→Setting→Editor→Colors&Fonts→Android Logcat→在界面的右侧调节logcat每个级别日志的颜色；

- 显示行号：File→Setting→Editor→General→Appearance→勾选“Show line numbers”；

- Logcat的console中，显示"no debuggable applications"的问题：Tools→Android→Enable ADB Integration；

- 如果依赖工程和主工程中有同名同类型的资源文件，需要修改依赖工程中的资源名称编译时才不会报错，如果依赖工程中的这个资源文件是整个工程都不需要用到的，可以直接删掉；

- Android Studio中一个主工程依赖多个library的模式编译时很慢（clean和rebuild时，之前Eclipse中是这种模式），因为这种工程框架是主工程和每个依赖工程中都有一个build.gradle，编译起来会消耗比较长的时间，可以将没有资源文件和so的依赖工程打包成jar包，有资源文件和so的打包成aar文件，然后在主工程中引用，这样编译会很快；

- Android Studio对九图的要求很严格，如果文件以".9.png"结尾但是图片不是9图，编译的时候会报错，解决方案是直接在AS中打开这张图片，通过9图编辑工具编辑成9图即可；

- 修改Module之间的依赖关系有两种方式：（1）直接修改每个module的build.gradle文件中的dependencies；（2）右键project→Open Module Settings→在弹出面板的左侧Modules一栏中选中要修改依赖关系的Module，点击右侧的Depencencies标签修改即可；

- Android Studio自动导包：File→Settings→Editor→General→Auto Import→Java→切换“Insert imports on paste”为“All”→勾选“Add unambigious imports on the fly”；

- 代码格式化快捷键：CTRL+ALT+L；

- 重命名文件夹或者文件的快捷键：ALT+SHIFT+R；

- 鼠标悬浮在某个方法上时，显示该方法的信息：Preferences→Editor→Show doc on mouse move；

- 删除一个Module，直接在IDE中选中Module后按Delete是删不掉的，需要先右键project→Open Module Settings→在弹出面板的左侧Modules一栏中选中要删除的Module→点击面板左上角的“-”符号→点击OK后回到IDE，然后选中要删掉的Module，按Delte快捷键删掉即可；

- Android Studio中执行Lint等工具对代码的检测，Analyze→Inspect Code。






	















