---
layout: post
title: Android各个Support Library介绍
category: Android
tags: Android
keywords: Android,Support Library,v4,v7,v8,v13,v14,v21
description: Android Support Library介绍
---


## 主工程、依赖包、jar包、android.jar、Android Support Library的关系

&emsp;&emsp;一个Android工程通常包括主工程和依赖包，依赖包又有两种形式：

- 一种是单独的工程：在主工程中的配置文件指明主工程和依赖包的依赖关系之后，就可以在主工程中正常使用依赖包的类和接口了，这种适合于依赖包中有图片资源、so等不方便打包到jar包中的情况，比如[Nine Old Androids](https://github.com/JakeWharton/NineOldAndroids)、[PullToRefresh](https://github.com/chrisbanes/Android-PullToRefresh)、[FancyCoverFlow](https://github.com/davidschreiber/FancyCoverFlow)等；

- 另一种是jar包：放在主工程的libs文件夹下，这种通常是依赖包中只有代码和可以打包到jar包中的文件，比如[Fastjson.jar](https://github.com/alibaba/fastjson)、[Volley.jar](https://android.googlesource.com/platform/frameworks/volley)、[Gson.jar](https://github.com/google/gson)等。

&emsp;&emsp;为了程序能够编译通过和在设备中正常运行，主工程除了依赖第三方的工程和jar包之外，还需要依赖安卓系统本身的代码，也就是我们在sdk的每个版本中看到的android.jar，这里面集成了android的所有API，随着android sdk的升级，高版本的sdk中会增加很多新的API，比如ActionBar、Fragment、RecyclerView等，如果在低版本的sdk中需要使用高版本新增的API怎么办？不可能去更新移动设备中的android.jar吧，因为硬件设备集成的sdk版本是固定的，android.jar也是固定的，设备中的一些参数、硬件选型也是根据当前sdk版本来定的，所以最好的方式是将新增的API以依赖包的形式集成到需要使用高版本API的应用程序中。

&emsp;&emsp;谷歌早已经考虑到了这个问题，所以推出了一系列脱离于android.jar的依赖包，比如常见的android-support-v4.jar、appcompat-v7等。这些依赖包可以直接集成到应用程序中，依赖包有的是jar包，有的是独立的工程。命名的如下：

**jar包：**

android-support-v[API Level Value].jar，比如android-support-v4.jar、android-support-v13.jar。

**依赖工程：**

[support包功能]_v[API Level Value]，比如appcompat_v7、gridlayout_v7。

&emsp;&emsp;各个依赖包可以在“<sdk>/extras/android/support/”文件夹下查看。

## 各个版本的Android Support Library介绍

### V4 Support Library

&emsp;&emsp;这个包的名字是：“android-support-v4.jar”，是为Android 1.6(API版本为4)及以上的版本设计的，它包含大部分高版本中有而低版本中没有的API，包括application components、user interface features、accessibility、data handling、network connectivity、and programming utilities，下面是对V4中的一些关键API的介绍：

**App Components：**

- **Fragment:**一个专为解决Android碎片化的类，通过它可以让同一个程序适配不同的屏幕。

- **NotificationCompat:**支持更丰富的通知形式；

- **LocalBroadcastManager:**适合于应用内的消息传递。

**User Interface：**

- **ViewPager:**一个可以管理子view的viewgroup，用户可以在各个view之间自由切换，这个在很多应用中都有使用到；

- **PagerTitleStrip:**一个关于当前页面、上一个页面和下一个页面的一个非交互的指示器。它经常作为ViewPager控件的一个子控件被被添加在XML布局文件中。

- **PagerTabStrip:**一个关于当前页面、上一个页面和下一个页面的一个可交互的指示器。它经常作为ViewPager控件的一个子控件被被添加在XML布局文件中。

- **DrawerLayout:**抽屉

- **SlidingPaneLayout:**用于实现两列面板的切换，在UI最上层的使用提供了一个水平的，多个面板的布局。左边的面板可以看作是一个内容列表或者是浏览，右边的面板的任务是显示详细的内容。

**Accessibility：**

- **ExploreByTouchHelper：**帮助自定义View实现accessibility的帮助类；

- **AccessibilityEventCompat、AccessibilityNodeInfoCompat、AccessibilityNodeProviderCompat、AccessibilityDelegateCompat：**Accessibility的适配类

**Content：**

- **Loader：**异步加载数据；

- **FileProvider：**应用间的私有文件共享。

&emsp;&emsp;关于V4的更多API介绍可以参见：[android-support-v4.jar API References](https://developer.android.com/intl/zh-cn/reference/android/support/v4/app/package-summary.html)

### Multidex Support Library

&emsp;&emsp;该support包用于使用多dex技术编译APP，当一个应用的方法数超过65536个时需要使用multidex配置，关于multidex的更多信息，可以参见[如何编译超过65K方法数的应用](https://developer.android.com/intl/zh-cn/tools/building/multidex.html)

### V7 SupportLibraries

&emsp;&emsp；针对Android 2.1(API Level 7)及以上的版本谷歌提供了一系列的support包，这些support包各自对应着特定的功能，每一个都可以单独地被引用。

#### V7 appcompat library

&emsp;&emsp;这个包的主要作用是为了在低版本实现Android的Holo风格界面而引入的，主要包括ActionBar、AppCompat等类和主题，它是一个依赖工程而不是jar包。

**注意：**这个包需要依赖android-support-v4.jar，如果你使用的是Eclipse或者Ant编译你的APP，确保你在使用这个依赖包时集成了android-support-v4.jar这个jar包。

#### v7 cardview library

&emsp;&emsp;一个在Android 5.0才被引入的卡片布局support包。

#### v7 gridlayout library

&emsp;&emsp;一个支持网格布局的support包。

#### v7 mediarouter library

&emsp;&emsp;一个用于设备间音频、视频交换显示的support包。

#### v7 palette library

&emsp;&emsp;一个可以实现页面的颜色动态变换的support包，Palette是这个support包的核心类。

#### v7 recyclerview library

&emsp;&emsp;核心类是RecyclerView，用于替换ListView、GridView等需要依赖Adapter的View，具体可以查阅RecyclerView方面的资料。

#### v7 Preference Support Library

&emsp;&emsp;一个用于支持各种控件存储配置数据的support包。

### v8 renderscript library

&emsp;&emsp;一个用于渲染脚本的support包。

### v13 Support Library

&emsp;&emsp;这个包的作用主要是为Android3.2（API Level 13）及以上的系统提供更多地Framgnet特性支持，使用它的原因在于，android-support-v4.jar中虽然也对Fragment做了支持，由于要兼容低版本，导致他是自行实现的 Fragment 效果，在高版本的 Fragment 的一些特性丢失了，而对于 v13以上的 sdk 版本，我们可以使用更加有效，特性更多的代码。

### v17 Leanback Library

&emsp;&emsp;一个主要作用是用于支持电视设备的support包，为电视设备提供了很多组件，比如：BroweFragment、DetailsFragment、PlaybackOverlayFragment、SearchFragment等。

### Annotations Support Library

&emsp;&emsp;一个支持注解的support包。

### Design Support Library

&emsp;&emsp;一个用于支持Design Patterns的support包。

### Custom Tabs Support Library

&emsp;&emsp;一个提供了在应用中添加和管理custom tabs的support包。

### Percent Support Library

&emsp;&emsp;一个提供了百分比布局的support包，通过这个包可以实现百分比布局。

## 在主工程中查看support包的源码

&emsp;&emsp;对于本来就是工程的support包来说，在主工程中查阅该support包中的代码非常简单，但如果support包是jar包，则需要在主工程中手动配置才能在主工程中查看support包的源码，关于在IDE中如何查看support jar包的源码可以参见：[Android 如何在Eclipse中查看Android API源码以及support包源码](http://blog.csdn.net/xiaanming/article/details/9031141)。

## 参考资料

- [Support Library Features](https://developer.android.com/intl/zh-cn/tools/support-library/features.html#recommendation)

- [Android Support v4、v7、v13的区别和应用场景](http://my.oschina.net/chengliqun/blog/148451)

- [Android 如何在Eclipse中查看Android API源码以及support包源码](http://blog.csdn.net/xiaanming/article/details/9031141)















