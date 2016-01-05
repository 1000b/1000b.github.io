---
layout: post
title: 使用Nexus Repository搭建属于自己公司的私有maven服务器
category: Android
tags: Android
keywords: Android,Nexus Repository,maven,私服
description: 使用Nexus Repository搭建属于自己公司的私有maven服务器
---

## 前言

  在Android应用开发过程中，不同IDE对工程的依赖方式不一样：


  使用Eclipse开发时，项目之间的依赖关系是这样的：一个主工程(project)可以依赖多个libproject、so、jar包，对jar包和so的依赖是直接将jar和so放在工程的libs文件夹下(老版本的ADT需要手动配置Build Path)，对libproject的依赖呈现在工程的“project.properties”文件中。

![](http://upload-images.jianshu.io/upload_images/44804-d3263edcd81eebb3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  使用Android Studio开发时，除了可以依赖module（对应Eclipse中的libproject）、jar和so，还可以依赖aar（aar和jar包不同之处在于可以将so和资源文件一起打包），as的依赖关系全部（jar、so、aar、libproject）在build.gradle文件中的android标签中管理。

![](http://upload-images.jianshu.io/upload_images/44804-c8ce1edae0f4894f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


  使用AS开发应用时，除了可以依赖本地的库之外，还可以依赖网上（公有maven服务器、私有maven服务器、jcenter等），如果是依赖本地的，必须要将依赖的module和主工程放在一个project里面，这就导致了每个project都需要配置这些依赖关系，如果是公司内多个工程依赖同一个公司内部的控件，控件有更新时，同步非常麻烦，但公司内部的控件不可能部署到公有maven服务器上，所以有必要搭建一个局域网内的maven服务器，方便管理公司内部的公共库。

## maven私有服务器搭建

  搭建maven私服使用得比较多的是Nexus，Nexus是基于maven仓库管理的社区项目，主要的使用场景就是可以在局域网搭建一个maven私服,用来部署第三方公共构件或者作为远程仓库在该局域网的一个代理。

  关于Nexus的介绍和配置很简单，具体可以查看这里：[Android 项目部署之Nexus私服搭建和应用](http://blog.csdn.net/l2show/article/details/48653949)。

## 上传库到私服

  上传库到私服有两种方式，一种是库中配置，配置完成后执行upload这个task，另外一种方式是直接上传。下面分别对这两种方式做介绍：

  在库中配置，步骤如下：

  1. 在project下的gradle.properties文件中定义通用属性，方便如果有多个库需要部署时，不需要修改每一个库中的配置：

        #本地库
        URLMAVEN_URL= http://172.28.1.*:8081/nexus/content/repositories/thirdparty/
        MAVEN_SNAPSHOT_URL = http://172.28.1.*:8081/nexus/content/repositories/thirdparty-snapshot/
        #对应maven的groupId值
        GROUP=common
        #登录nexus oss的用户名
        NEXUS_USERNAME=admin
        #登录nexus oss的密码
        NEXUS_PASSWORD=admin123
        # groupid
        GROUP_ID = common
        # type
        TYPE = aar
        # description
        DESCRIPTION = dependences lib


  2. 修改module对应的build.gradle文件，配置以谁的名义上传这个库，上传到什么地方，这个库叫什么名字，属于哪个group，ID和version、description、packageing等信息

          apply plugin: 'com.android.library'
          apply plugin: 'maven'

          android {
                    compileSdkVersion 17 
                    buildToolsVersion "23.0.2"
                    defaultConfig {
                              minSdkVersion 15
                              targetSdkVersion 17
                              versionCode 1
                              versionName "1.0"
                    }    

                    buildTypes {        
                              release {            
                                        minifyEnabled false            
                                        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'        
                                        }    
                    }   
 
                    sourceSets {        
                              main {            
                                        jniLibs.srcDirs = ['libs']        
                              }    
                    }  
  
                    lintOptions {       
                               abortOnError false    
                    }    
          }

          dependencies {    
                    compile fileTree(dir: 'libs', include: ['*.jar']) 
          }

          uploadArchives {    
                    configuration = configurations.archives    
                    repositories {        
                              mavenDeployer {            
                                        snapshotRepository(url: MAVEN_SNAPSHOT_URL) {                
                                                  authentication(userName: NEXUS_USERNAME, password: NEXUS_PASSWORD)            
                                        }            
                                        repository(url: MAVEN_URL) {                
                                                  authentication(userName: NEXUS_USERNAME, password: NEXUS_PASSWORD)            
                                        }            
                                        pom.project {                
                                                  version '1.0.0'                
                                                  artifactId 'TestLibrary'                
                                                  groupId GROUP_ID                
                                                  packaging TYPE                
                                                  description DESCRIPTION            
                                        }        
                              }    
                    }
          }

          artifacts {    
          archives file('TestLibrary.aar')
}


  3. 打开Gradle projects（在AS的右边栏），找到对应的module，展开，找到Tasks下面的upload标签并双击，在Gradle Console标签可以查看是否上传成功。

![](http://upload-images.jianshu.io/upload_images/44804-fcd7a59e8df08df0.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


  直接上传：直接上传很简单，直接按照下图的箭头操作即可，如果上传aar还没有研究过，有兴趣的可以自己研究一下：

![](http://upload-images.jianshu.io/upload_images/44804-baf52d72407bffa2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 在项目中使用私有仓库


  在项目中使用私有仓库的步骤如下：


  1. 在project的build.gradle文件中指定私有仓库的地址，like this：

![](http://upload-images.jianshu.io/upload_images/44804-04243a7e8bf65436.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


  2. 在需要依赖私有仓库的build.gradle文件中设置依赖关系：

![](http://upload-images.jianshu.io/upload_images/44804-44c656639adffa6b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)