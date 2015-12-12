---
layout: post
title: Eclipse转Android Studio的过程中有必要弄明白的一些问题
category: Android
tags: Android
keywords: Android,Android Studio,Eclipse
description: Eclipse转Android Studio的过程中有必要弄明白的一些问题
---

&emsp;&emsp;AS出来一年多了，最近才从Eclipse转到AS，但我并不觉得使用Eclipse有多落后，它们都只是一个工具而已，哪个顺手就用哪个，用得好都能提高生产力，不会合理利用，再好的工具也是惘然。很多使用Eclipse的Android程序员不知道代码重构的快捷键、如何在运行时调试、一个Workspace一大堆工程......，我想即使转到Android Studio也并不见得比Eclipse顺手。

&emsp;&emsp;下面将自己在Eclipse转AS过程中遇到的一些问题以及对各个问题的理解列出来，方便后续查阅。

### 1、问：Eclipse的工程如何导入到AS？

**答：**我的处理方式是在AS中新建工程，然后将Eclipse中对应工程的文件拷贝过来；当然也可以通过Eclipse将project导成gradle版本的，然后在AS中导入该工程。

### 2、问：对于Native层的代码，是如何处理的，在AS上如何编译JNI的代码？

**答：**AS上同样可以开发JNI，只不过配置脚本的过程比较麻烦，各个gradle版本，配置的方式有些不一样；我的处理方式是AS上只做java开发，JNI还是在Eclipse中开发，方便编译和调试；

### 3、问：在AS上开发会和Eclipse一样，卡吗？

**答：**会，卡不卡和你整个工作空间的复杂度有关，如果Eclipse的一个工作空间工程比较少，是不会卡的；AS也一样，如果AS的一个工作空间有太多工程，同样会很卡，特别是编译的时候；

### 4、问：AS存在启动慢的问题吗，有没有Eclipse那种初始化进度一直在0%的状态？

**答：**目前为止我还没有遇到过，即使一个工作空间有上十个工程。

### 5、问：AS编译比Eclipse或者ant编译快吗？

**答：**不一定，这也跟你项目的复杂度有关，如果你的工程依赖关系简单，用gradle编译会很快，当然用Eclipse和ant编译也一样；如果你的工程依赖关系复杂，用gradle编译比用Eclipse、ant还慢。我的建议是：主工程不要依赖太多的libproject，否则会编译很慢，可以把这些libproject打包成aar，这样同样复杂的项目用gradle编译会比用Eclipse和ant编译快不少；

### 6、gradle与ant相比，有什么优点？

**答：**优点比较多，主要的优点是配置简单，特别是在持续集成的时候，如果是gradle，一条命令就行了，如果是ant，还得自己写编译脚本。

### 7、问：AS中如何配置工程的依赖关系？

**答：**在Eclipse中，会存在几种依赖：

- 一种是jar包，直接放在libs文件夹即可（早先的Eclipse版本需要设置buildpath依赖关系才算配置OK）；

- 另外一种是libproject，这需要右键主工程—properties—Android—点击Add添加依赖项，配置完成后依赖关系会更新到工程根目录下的“project.properties”文件。

&emsp;&emsp;在AS中会很简单，右键主工程—Open Module Setting — 选中某一个工程，点击右边的Dependencies选项，点击“+”，分别添加Library/File/Module dependency，Library dependency和File dependency主要是添加jar包（File dependency的jar包是放在工程的libs文件夹下），Module dependency是添加libproject，so放在工程的"libs/架构文件夹"下，不需要配置依赖关系。AS的依赖关系配置完成后，可以在工程的"build.gradle"文件中查看。依赖关系配置完成后，记得在build.gralde文件的android标签下增加下面这句话，依赖关系才生效：

	sourceSets{    
		main {
	        jniLibs.srcDirs = ['libs']
	    }
	}

### 8、问：Eclipse和AS中主工程对其它工程的依赖有什么异同？

**答：**

- 相同点：Eclipse和AS都可以依赖so、jar包和libproject；组织结构也一样，so和jar包放在libs文件夹，libproject是一个独立的工程，需要手动配置依赖关系。

- 不同点：AS还可以依赖aar，并且AS除了可以依赖本地的库，还可以依赖服务器上的库，但Eclipse只能依赖本地库。

### 9、问：jar包和aar有啥区别？

**答：**jar包不能将so和资源文件打包进去，但aar可以，看得到的就是这点区别。

### 10、问：AS使用过程遇到过哪些问题，是怎么解决的？

**答：**

- assets文件的存放目录在”src/main/”目录下，和java、res文件夹平级；

- aidl文件需要单独在”src/main/”目录下新建一个文件夹，然后创建对应的包名，将aidl文件放在包名对应的包下；

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

- Logcat的console中，显示”no debuggable applications”的问题：Tools→Android→Enable ADB Integration；

- 如果依赖工程和主工程中有同名同类型的资源文件，需要修改依赖工程中的资源名称编译时才不会报错，如果依赖工程中的这个资源文件是整个工程都不需要用到的，可以直接删掉；

- Android Studio中一个主工程依赖多个library的模式编译时很慢（clean和rebuild时，之前Eclipse中是这种模式），因为这种工程框架是主工程和每个依赖工程中都有一个build.gradle，编译起来会消耗比较长的时间，可以将没有资源文件和so的依赖工程打包成jar包，有资源文件和so的打包成aar文件，然后在主工程中引用，这样编译会很快；

- Android Studio对九图的要求很严格，如果文件以”.9.png”结尾但是图片不是9图，编译的时候会报错，解决方案是直接在AS中打开这张图片，通过9图编辑工具编辑成9图即可；

- 修改Module之间的依赖关系有两种方式：

（1）直接修改每个module的build.gradle文件中的dependencies；

（2）右键project→Open Module Settings→在弹出面板的左侧Modules一栏中选中要修改依赖关系的Module，点击右侧的Depencencies标签修改即可；

- Android Studio自动导包：File→Settings→Editor→General→Auto Import→Java→切换“Insert imports on paste”为“All”→勾选“Add unambigious imports on the fly”；

- 代码格式化快捷键：CTRL+ALT+L；

- 重命名文件夹或者文件的快捷键：ALT+SHIFT+R；

- 鼠标悬浮在某个方法上时，显示该方法的信息：Preferences→Editor→Show doc on mouse move；

- 删除一个Module，直接在IDE中选中Module后按Delete是删不掉的，需要先右键project→Open Module Settings→在弹出面板的左侧Modules一栏中选中要删除的Module→点击面板左上角的“-”符号→点击OK后回到IDE，然后选中要删掉的Module，按Delte快捷键删掉即可；

- Android Studio中执行Lint等工具对代码的检测，Analyze→Inspect Code；

- 导入aar：将aar拷贝到libs文件夹，在module的build.gradle文件增加下面这段话：

		repositories {
			flatDir() {
			    dirs 'libs'
			}
		}

然后在build.gradle的dependencies标签中按照如下格式引用aar文件即可：

		compile(name:'aar包名不带扩展名', ext:'aar')

### 12、问：AS相比于Eclipse，有哪些新的工具或者更方便的功能？

**答：**

- 查看APP的内存占用、内存变化情况的工具；

- 查看APP运行过程中网络使用情况的工具；

- 查看CPU、GPU使用情况的工具；

- 代码清理（Analyze—Code cleanup....）、代码静态检查工具（增强的ling检查工具，Analyze—Inspect code....）；

- 可以直接使用DOS窗口；

- 给打码加书签的功能（Eclipse也有，只是之前没用过）；

- IDE类9图编辑功能；快捷键......很多很多一些小的功能，用熟了特别方便。

### 13、问：在使用AS的过程中，有什么忠告？

**答：**就像在使用Eclipse的时候不要轻易更新ADT一样，在使用AS的过程中不要轻易更新gradle和AS，每个版本会有一些差别，会有很多坑，还是等新版本出来一段时间，比较稳定后再用，毕竟IDE是提高生产力的工具，如果需要花大把时间去学习如何使用和解决使用过程中的问题就太没意思了。

### 14、问：Eclipse的工程转成AS的版本后，在同一个机器中安装会报”INSTALL_FAILED_VERSION_DOWNGRADE“这个错误，是什么原因导致的？

**答：**这是因为机器中存在一个版本号比安装apk版本号更高的版本，出现这个问题的原因是因为as除了可以在Manifest.xml文件中设置apk的版本名和版本号，还可以在build.gradle文件中设置apk的版本名和版本号，记得修改build.gralde中的版本名和版本号到最新就可以了。

### 15、导入Google官方的Sample不成功，如何处理？

**答：**Android Studio 导入 Google Sample 时，需要联网获取，由于某些原因，连接不了服务器。参考网上的方案，给出一种更为简便、快速的方法：在 host 文件中加入一句： 

		64.233.162.84 gsamplesindex.appspot.com 

如果 ip 失效了，找个有用的替换即可（win7的host文件在：C:\Windows\system32\drivers\etc 目录下）。

### 16、问：如何在构建时删掉build\output目录下的unsigned-align.apk？

**答：**在build.gradle文件中加上下面这一段话：

	// delete unaligned files
	android.applicationVariants.all { variant ->
	  variant.assemble.doLast {
	    variant.outputs.each { output ->
	        println "aligned " + output.outputFile
	        println "unaligned " + output.packageApplication.outputFile
	
	        File unaligned = output.packageApplication.outputFile;
	        File aligned = output.outputFile
	        if (!unaligned.getName().equalsIgnoreCase(aligned.getName())){
	            println "deleting " + unaligned.getName()
	            unaligned.delete()
	        }
	    }
	}




	















