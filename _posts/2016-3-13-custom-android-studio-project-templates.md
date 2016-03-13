---
layout: post
title: 自定义Android Studio工程模板
category: Android
tags: Android
keywords: Android,Android Studio,project templates,template
description: 自定义Android Studio工程模板
---

&emsp;&emsp;每次创建新的Android Studio工程时，都需要手动修改一些工程的配置，比如删除不必要的依赖、删掉Activity中不必要的代码
配置私有maven库的地址、增加公用的依赖库、修改.gitingore、关闭lint的严格检查、配置APK的输出路径等等；项目比较少还好，如果项目比较多，并且还不断有新人加入时，就可以考虑修改Android Studio默认的project和module模板，避免做无用功和口口相传。

## 工程模板路径

&emsp;&emsp;Android Studio的工程模板在安装目录的“\plugins\android\lib\templates\gradle-projects”文件夹下，这里面包含了导入工程模板、新建工程模板、新建module模板等。

![gradle_project_template](http://ww2.sinaimg.cn/large/6d17e381gw1f1v5fphbfij20eh054abe.jpg)

## 提醒

&emsp;&emsp;修改前一定要备份，避免修改模板后导致新建project和module异常的情况（亲身经历，出现这样的情况你会很烦躁的）；

## 模板文件说明

#### NewAndroidProject模板

![NewAndroidProject](http://ww1.sinaimg.cn/large/6d17e381gw1f1v6fvcryqj20py0780y7.jpg)

#### NewAndroidModule模板

![NewAndroidModule](http://ww4.sinaimg.cn/large/6d17e381gw1f1v6rzdfzrj20py0adqac.jpg)

## 修改Android Studio工程模板

&emsp;&emsp;对工程模板中的每个文件和文件夹有一定的了解后，就可以按照自己的意愿修改模板中的内容了，如何修改得根据自己的需求而定，举个栗子，修改module的build.gradle模板（Android Studio\plugins\android\lib\templates\gradle-projects\NewAndroidModule\root\build.gradle.ftl），通常我们不需要依赖单元测试，去掉对单元测试的依赖，屏蔽lint的严格检查、并配置好APK的输出路径：

	<#if !(perModuleRepositories??) || perModuleRepositories>
	buildscript {
	    repositories {
	        jcenter()
	<#if mavenUrl != "mavenCentral">
	        maven {
	            url '${mavenUrl}'
	        }
	</#if>
	    }
	    dependencies {
	        classpath 'com.android.tools.build:gradle:${gradlePluginVersion}'
	    }
	}
	</#if>
	<#if isLibraryProject?? && isLibraryProject>
	apply plugin: 'com.android.library'
	<#else>
	apply plugin: 'com.android.application'
	</#if>
	<#if !(perModuleRepositories??) || perModuleRepositories>
	
	repositories {
	        jcenter()
	<#if mavenUrl != "mavenCentral">
	        maven {
	            url '${mavenUrl}'
	        }
	</#if>
	}
	</#if>
	
	android {
	    compileSdkVersion <#if buildApiString?matches("^\\d+$")>${buildApiString}<#else>'${buildApiString}'</#if>
	    buildToolsVersion "${buildToolsVersion}"
	
	    defaultConfig {
	    <#if isLibraryProject?? && isLibraryProject>
	    <#else>
	    applicationId "${packageName}"
	    </#if>
	        minSdkVersion <#if minApi?matches("^\\d+$")>${minApi}<#else>'${minApi}'</#if>
	        targetSdkVersion <#if targetApiString?matches("^\\d+$")>${targetApiString}<#else>'${targetApiString}'</#if>
	        versionCode 1
	        versionName "1.0"
	    }
	<#if javaVersion?? && (javaVersion != "1.6" && buildApi lt 21 || javaVersion != "1.7")>
	
	    compileOptions {
	        sourceCompatibility JavaVersion.VERSION_${javaVersion?replace('.','_','i')}
	        targetCompatibility JavaVersion.VERSION_${javaVersion?replace('.','_','i')}
	    }
	</#if>
	<#if enableProGuard>
	    buildTypes {
	        release {
	            minifyEnabled false
	            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
	        }
	    }
	</#if>
	
	    sourceSets {
	        main {
	            jniLibs.srcDirs = ['libs']
	        }
	    }
		
		// 屏蔽lint的严格检查
	    lintOptions {
	        abortOnError false
	    }
	}
	
	dependencies {
	    <#if dependencyList?? >
	    <#list dependencyList as dependency>
	    compile '${dependency}'
	    </#list>
	    </#if>
		// 在这里删掉对单元测试的依赖，如果需要依赖公用控件，直接在这里添加
	    compile fileTree(dir: 'libs', include: ['*.jar'])
	}
	
	repositories {
	    flatDir() {
	        dirs 'libs'
	    }
	}
	
	// 删掉没有签名的APK文件，同时在这里也可以配置APK文件的输出路径
	android.applicationVariants.all { variant ->
	  variant.assemble.doLast {
	    variant.outputs.each { output ->
	        println "aligned " + output.outputFile
	        println "unaligned " + output.packageApplication.outputFile
	
	        File unaligned = output.packageApplication.outputFile;
	        File aligned = output.outputFile
	        if (!unaligned.getName().equalsIgnoreCase(aligned.getName())) {
	            println "deleting " + unaligned.getName()
	            unaligned.delete()
	        }
	    }
	  }
	}
	


	






