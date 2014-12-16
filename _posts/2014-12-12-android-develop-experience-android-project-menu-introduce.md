---
layout: post
title: Android开发经验谈-Android工程目录介绍
category: Android
tags: Android
keywords: Android开发经验，工程目录, 源码，Resources， assets
description: Android开发经验谈-Android工程目录介绍
---

&emsp;&emsp;

## Android工程目录介绍
一个完整的Android工程目录大致如下图所示：

![Android工程目录](http://img.my.csdn.net/uploads/201412/15/1418642748_9321.jpg)

&emsp;&emsp;其中：**gen**、**Android 4.0.3**、**Android Dependencies**、**bin**、**obj**为程序编译时生成的目录，下面一一对android工程下各文件夹进行介绍：

- **src:**存放程序源代码，用于存放后缀为*.java、*.aidl的文件；

- **gen:**这也是程序编译生成的目录，包含“BuildConfig.java”和“R.java”两个自动生成的文件，里面的内容不可以修改，关于BuildConfig.java的运用可参见[Android BuildConfig.DEBUG的妙用](http://stormzhang.com/android/2013/08/28/android-use-build-config/),R.java中存放了各种资源文件的ID，如果你的工程依赖其它的libproject，你会发现你的gen文件夹下有多个R.java文件（一个包名对应一个R.java文件），这些资源ID的值都是有规则的，你会看到它们都是以"0x7f"打头的，具体的命名规则自行谷歌。

- **Android 4.0.3:**这个是工程依赖android.jar包的版本，在Eclipse中，可以通过右键工程 -> Properties -> Android -> 勾选对应的android sdk -> OK,来切换工程依赖的android.jar包版本，当你阅读github上的项目时，多数情况都会是作者通过.gitignore将编译生成的目录过滤掉了，这是你可以通过查看“project.properties”文件中的“target”属性以了解该项目编译时采用的android.jar包版本。

- **Android Dependencies:**这个文件夹下存放了工程依赖的所有jar包，它是编译后自动生成的文件夹，libs文件夹下的所有jar包在编译后都会拷贝到这个文件夹下。

- **assets:**这里可以存放一个程序的基础数据文件，比如你工程里面需要播放的flash文件，需要加载的本地网页、字库文件等等。

- **bin:**编译后生成的dex、apk、资源文件都存放在这里面，这个文件夹相当于apk解压后的文件夹；

- **jni:**如果你的工程中的某些功能需要通过库将其封装起来，可以将具体实现放在jni中，jni文件夹下至少需要有"Android.mk"、"Application.mk"、"*.c"、".h"这几种文件，其中"Android.mk"文件是jni源码的编译规则文件，它是一个makefile，"Application.mk"文件用于指定编译库所支持的平台，常见的有arm和mips，其中arm又分为armeabi、armeabi-v7a、armeabi-v9a等等，jni编译后生成的库存放在libs文件夹下，具体存放的路径根据Application.mk文件中指定的编译库平台而定。

- **libs:**用于存放so和jar包，其中so为工程依赖的库文件，jar包为程序依赖的第三方工程。

- **obj:**为程序编译生成的目录，生成的文件后缀为*.o，如果想具体了解，可以看一下编译原理。

- **res:**存放程序的资源文件，这里面可以存放图片、动画、selector文件、音频、属性信息等。
	
	**res/anim：**用于存放工程中的动画文件(*.xml,animationdrawable的动画文件也存放在该文件夹下）,定义格式如下(具体内容根据动画效果而定)：

		<?xml version="1.0" encoding="utf-8"?>
		<set xmlns:android="http://schemas.android.com/apk/res/android"
		    android:interpolator="@android:anim/decelerate_interpolator" >
		    <alpha
		        android:duration="3000"
		        android:fromAlpha="0.0"
		        android:toAlpha="1.0" />
		</set>
	
	**res/menu**:存放菜单项的布局；

	**res/drawable：**用户存放各组件的selector文件，比如按钮的正反选，定义格式如下：
		
		<?xml version="1.0" encoding="utf-8"?>
		<selector xmlns:android="http://schemas.android.com/apk/res/android" >
		    <item android:drawable="@drawable/app_update_confirm_normal" android:state_pressed="false"/>
		    <item android:drawable="@drawable/app_update_confirm_pressed" android:state_pressed="true"/>
		</selector>

	**res/drawable-hdpi、res/drawable-mdpi、res/drawable-ldpi、res/drawable-xhdpi：**用于存放工程中用到的图片文件，不同文件夹对应图片的精度不一样；

	**res/layout：**每个activity、fragment、自定义view、popupwindow、dialog、adapter的布局文件，如果你的程序需要多分辨率适配，你可以新建多个layout文件夹，文件夹后面跟上分辨率即可。

	**res/raw：**用于存放音频文件，比如app的背景音、各种音效等等，后缀通常为*.ogg、*.mp3、*.wav等等；

	**res/values/attrs.xml：**用于存放自定义view的自定义属性信息，定义格式如下：
		
		<?xml version="1.0" encoding="utf-8"?>
		<resources>
		    <attr name="focused_icon" format="reference" />
		    <attr name="indicator_icon" format="reference" />
		    <attr name="indicator_startY" format="integer" />
		    <attr name="indicator_icon_width" format="integer" />
		    <attr name="background_array" format="reference" />
		
		    <declare-styleable name="WelcomeView">
		        <attr name="focused_icon" />
		        <attr name="indicator_icon" />
		        <attr name="indicator_startY" />
		        <attr name="indicator_icon_width" />
		        <attr name="background_array" />
		    </declare-styleable>
		    
		    <declare-styleable name="SpellToneView">  
		        <attr name="view_width" format="integer" />  
		        <attr name="view_height" format="integer" />  
		    </declare-styleable>
		</resources>

	**res/values/styles.xml：**存放activity、application等自定义主题，定义格式如下：

		<style name="AppTheme" parent="android:Theme.Light.NoTitleBar">
	        <item name="android:windowEnableSplitTouch">false</item>
	        <item name="android:splitMotionEvents">false</item>
	    </style>

	**res/values/arrays.xml：**存放数组文件，支持定义整数、字符串、资源文件数组等等，定义格式如下：
	
		<?xml version="1.0" encoding="utf-8"?>
		<resources>
		    <array name="backgrounds">  
		        <item>@drawable/learn_spell_splash_1</item>
		        <item>@drawable/learn_spell_splash_2</item>    
			</array>
		</resources>

	**res/values/colors.xml：**用于定义颜色值，定义格式如下：

		<?xml version="1.0" encoding="utf-8"?>
		<resources>
		    <!-- 白色 -->
		    <color name="ivory">#FFFFF0</color>
		    <!-- 象牙色 -->
		    <color name="lightyellow">#FFFFE0</color>
		    <!-- 亮黄色 -->
		    <color name="yellow">#FFFF00</color>
		    <!-- 黄色 -->
		    <color name="snow">#FFFAFA</color>
		    <!-- 雪白色 -->
		    <color name="floralwhite">#FFFAF0</color>
		    <!-- 花白色 -->
		    <color name="lemonchiffon">#FFFACD</color>
		    <!-- 柠檬绸色 -->
		    <color name="cornsilk">#FFF8DC</color>
		    <!-- 米绸色 -->
		    <color name="seashell">#FFF5EE</color>
		    <!-- 海贝色 -->
		    <color name="lavenderblush">#FFF0F5</color>
		    <!-- 淡紫红 -->
		    <color name="papayawhip">#FFEFD5</color>
		    <!-- 番木色 -->
		    <color name="blanchedalmond">#FFEBCD</color>
		    <!-- 白杏色 -->
		    <color name="mistyrose">#FFE4E1</color>
		    <!-- 浅玫瑰色 -->
		    <color name="bisque">#FFE4C4</color>
		    <!-- 桔黄色 -->
		    <color name="moccasin">#FFE4B5</color>
		    <!-- 鹿皮色 -->
		    <color name="navajowhite">#FFDEAD</color>
		    <!-- 纳瓦白 -->
		    <color name="peachpuff">#FFDAB9</color>
		    <!-- 桃色 -->
		    <color name="gold">#FFD700</color>
		    <!-- 金色 -->
		    <color name="pink">#FFC0CB</color>
		    <!-- 粉红色 -->
		    <color name="lightpink">#FFB6C1</color>
		    <!-- 亮粉红色 -->
		    <color name="orange">#FFA500</color>
		    <!-- 橙色 -->
		    <color name="lightsalmon">#FFA07A</color>
		    <!-- 亮肉色 -->
		    <color name="darkorange">#FF8C00</color>
		    <!-- 暗桔黄色 -->
		    <color name="coral">#FF7F50</color>
		    <!-- 珊瑚色 -->
		    <color name="hotpink">#FF69B4</color>
		    <!-- 热粉红色 -->
		    <color name="tomato">#FF6347</color>
		    <!-- 西红柿色 -->
		    <color name="orangered">#FF4500</color>
		    <!-- 红橙色 -->
		    <color name="deeppink">#FF1493</color>
		    <!-- 深粉红色 -->
		    <color name="fuchsia">#FF00FF</color>
		    <!-- 紫红色 -->
		    <color name="magenta">#FF00FF</color>
		    <!-- 红紫色 -->
		    <color name="red">#FF0000</color>
		    <!-- 红色 -->
		    <color name="oldlace">#FDF5E6</color>
		    <!-- 老花色 -->
		    <color name="lightgoldenrodyellow">#FAFAD2</color>
		    <!-- 亮金黄色 -->
		    <color name="linen">#FAF0E6</color>
		    <!-- 亚麻色 -->
		    <color name="antiquewhite">#FAEBD7</color>
		    <!-- 古董白 -->
		    <color name="salmon">#FA8072</color>
		    <!-- 鲜肉色 -->
		    <color name="ghostwhite">#F8F8FF</color>
		    <!-- 幽灵白 -->
		    <color name="mintcream">#F5FFFA</color>
		    <!-- 薄荷色 -->
		    <color name="whitesmoke">#F5F5F5</color>
		    <!-- 烟白色 -->
		    <color name="beige">#F5F5DC</color>
		    <!-- 米色 -->
		    <color name="wheat">#F5DEB3</color>
		    <!-- 浅黄色 -->
		    <color name="sandybrown">#F4A460</color>
		    <!-- 沙褐色 -->
		    <color name="azure">#F0FFFF</color>
		    <!-- 天蓝色 -->
		    <color name="honeydew">#F0FFF0</color>
		    <!-- 蜜色 -->
		    <color name="aliceblue">#F0F8FF</color>
		    <!-- 艾利斯兰 -->
		    <color name="khaki">#F0E68C</color>
		    <!-- 黄褐色 -->
		    <color name="lightcoral">#F08080</color>
		    <!-- 亮珊瑚色 -->
		    <color name="palegoldenrod">#EEE8AA</color>
		    <!-- 苍麒麟色 -->
		    <color name="violet">#EE82EE</color>
		    <!-- 紫罗兰色 -->
		    <color name="darksalmon">#E9967A</color>
		    <!-- 暗肉色 -->
		    <color name="lavender">#E6E6FA</color>
		    <!-- 淡紫色 -->
		    <color name="lightcyan">#E0FFFF</color>
		    <!-- 亮青色 -->
		    <color name="burlywood">#DEB887</color>
		    <!-- 实木色 -->
		    <color name="plum">#DDA0DD</color>
		    <!-- 洋李色 -->
		    <color name="gainsboro">#DCDCDC</color>
		    <!-- 淡灰色 -->
		    <color name="crimson">#DC143C</color>
		    <!-- 暗深红色 -->
		    <color name="palevioletred">#DB7093</color>
		    <!-- 苍紫罗兰色 -->
		    <color name="goldenrod">#DAA520</color>
		    <!-- 金麒麟色 -->
		    <color name="orchid">#DA70D6</color>
		    <!-- 淡紫色 -->
		    <color name="thistle">#D8BFD8</color>
		    <!-- 蓟色 -->
		    <color name="lightgray">#D3D3D3</color>
		    <!-- 亮灰色 -->
		    <color name="lightgrey">#D3D3D3</color>
		    <!-- 亮灰色 -->
		    <color name="tan">#D2B48C</color>
		    <!-- 茶色 -->
		    <color name="chocolate">#D2691E</color>
		    <!-- 巧可力色 -->
		    <color name="peru">#CD853F</color>
		    <!-- 秘鲁色 -->
		    <color name="indianred">#CD5C5C</color>
		    <!-- 印第安红 -->
		    <color name="mediumvioletred">#C71585</color>
		    <!-- 中紫罗兰色 -->
		    <color name="silver">#C0C0C0</color>
		    <!-- 银色 -->
		    <color name="darkkhaki">#BDB76B</color>
		    <!-- 暗黄褐色 -->
		    <color name="rosybrown">#BC8F8F</color>
		    <!-- 褐玫瑰红 -->
		    <color name="mediumorchid">#BA55D3</color>
		    <!-- 中粉紫色 -->
		    <color name="darkgoldenrod">#B8860B</color>
		    <!-- 暗金黄色 -->
		    <color name="firebrick">#B22222</color>
		    <!-- 火砖色 -->
		    <color name="powderblue">#B0E0E6</color>
		    <!-- 粉蓝色 -->
		    <color name="lightsteelblue">#B0C4DE</color>
		    <!-- 亮钢兰色 -->
		    <color name="paleturquoise">#AFEEEE</color>
		    <!-- 苍宝石绿 -->
		    <color name="greenyellow">#ADFF2F</color>
		    <!-- 黄绿色 -->
		    <color name="lightblue">#ADD8E6</color>
		    <!-- 亮蓝色 -->
		    <color name="darkgray">#A9A9A9</color>
		    <!-- 暗灰色 -->
		    <color name="darkgrey">#A9A9A9</color>
		    <!-- 暗灰色 -->
		    <color name="brown">#A52A2A</color>
		    <!-- 褐色 -->
		    <color name="sienna">#A0522D</color>
		    <!-- 赭色 -->
		    <color name="darkorchid">#9932CC</color>
		    <!-- 暗紫色 -->
		    <color name="palegreen">#98FB98</color>
		    <!-- 苍绿色 -->
		    <color name="darkviolet">#9400D3</color>
		    <!-- 暗紫罗兰色 -->
		    <color name="mediumpurple">#9370DB</color>
		    <!-- 中紫色 -->
		    <color name="lightgreen">#90EE90</color>
		    <!-- 亮绿色 -->
		    <color name="darkseagreen">#8FBC8F</color>
		    <!-- 暗海兰色 -->
		    <color name="saddlebrown">#8B4513</color>
		    <!-- 重褐色 -->
		    <color name="darkmagenta">#8B008B</color>
		    <!-- 暗洋红 -->
		    <color name="darkred">#8B0000</color>
		    <!-- 暗红色 -->
		    <color name="blueviolet">#8A2BE2</color>
		    <!-- 紫罗兰蓝色 -->
		    <color name="lightskyblue">#87CEFA</color>
		    <!-- 亮天蓝色 -->
		    <color name="skyblue">#87CEEB</color>
		    <!-- 天蓝色 -->
		    <color name="gray">#808080</color>
		    <!-- 灰色 -->
		    <color name="olive">#808000</color>
		    <!-- 橄榄色 -->
		    <color name="purple">#800080</color>
		    <!-- 紫色 -->
		    <color name="maroon">#800000</color>
		    <!-- 粟色 -->
		    <color name="aquamarine">#7FFFD4</color>
		    <!-- 碧绿色 -->
		    <color name="chartreuse">#7FFF00</color>
		    <!-- 黄绿色 -->
		    <color name="lawngreen">#7CFC00</color>
		    <!-- 草绿色 -->
		    <color name="mediumslateblue">#7B68EE</color>
		    <!-- 中暗蓝色 -->
		    <color name="lightslategray">#778899</color>
		    <!-- 亮蓝灰 -->
		    <color name="lightslategrey">#778899</color>
		    <!-- 亮蓝灰 -->
		    <color name="slategray">#708090</color>
		    <!-- 灰石色 -->
		    <color name="slategrey">#708090</color>
		    <!-- 灰石色 -->
		    <color name="olivedrab">#6B8E23</color>
		    <!-- 深绿褐色 -->
		    <color name="slateblue">#6A5ACD</color>
		    <!-- 石蓝色 -->
		    <color name="dimgray">#696969</color>
		    <!-- 暗灰色 -->
		    <color name="dimgrey">#696969</color>
		    <!-- 暗灰色 -->
		    <color name="mediumaquamarine">#66CDAA</color>
		    <!-- 中绿色 -->
		    <color name="cornflowerblue">#6495ED</color>
		    <!-- 菊兰色 -->
		    <color name="cadetblue">#5F9EA0</color>
		    <!-- 军兰色 -->
		    <color name="darkolivegreen">#556B2F</color>
		    <!-- 暗橄榄绿 -->
		    <color name="indigo">#4B0082</color>
		    <!-- 靛青色 -->
		    <color name="mediumturquoise">#48D1CC</color>
		    <!-- 中绿宝石 -->
		    <color name="darkslateblue">#483D8B</color>
		    <!-- 暗灰蓝色 -->
		    <color name="steelblue">#4682B4</color>
		    <!-- 钢兰色 -->
		    <color name="royalblue">#4169E1</color>
		    <!-- 皇家蓝 -->
		    <color name="turquoise">#40E0D0</color>
		    <!-- 青绿色 -->
		    <color name="mediumseagreen">#3CB371</color>
		    <!-- 中海蓝 -->
		    <color name="limegreen">#32CD32</color>
		    <!-- 橙绿色 -->
		    <color name="darkslategray">#2F4F4F</color>
		    <!-- 暗瓦灰色 -->
		    <color name="darkslategrey">#2F4F4F</color>
		    <!-- 暗瓦灰色 -->
		    <color name="seagreen">#2E8B57</color>
		    <!-- 海绿色 -->
		    <color name="forestgreen">#228B22</color>
		    <!-- 森林绿 -->
		    <color name="lightseagreen">#20B2AA</color>
		    <!-- 亮海蓝色 -->
		    <color name="dodgerblue">#1E90FF</color>
		    <!-- 闪兰色 -->
		    <color name="midnightblue">#191970</color>
		    <!-- 中灰兰色 -->
		    <color name="aqua">#00FFFF</color>
		    <!-- 浅绿色 -->
		    <color name="cyan">#00FFFF</color>
		    <!-- 青色 -->
		    <color name="springgreen">#00FF7F</color>
		    <!-- 春绿色 -->
		    <color name="lime">#00FF00</color>
		    <!-- 酸橙色 -->
		    <color name="mediumspringgreen">#00FA9A</color>
		    <!-- 中春绿色 -->
		    <color name="darkturquoise">#00CED1</color>
		    <!-- 暗宝石绿 -->
		    <color name="deepskyblue">#00BFFF</color>
		    <!-- 深天蓝色 -->
		    <color name="darkcyan">#008B8B</color>
		    <!-- 暗青色 -->
		    <color name="teal">#008080</color>
		    <!-- 水鸭色 -->
		    <color name="green">#008000</color>
		    <!-- 绿色 -->
		    <color name="darkgreen">#006400</color>
		    <!-- 暗绿色 -->
		    <color name="blue">#0000FF</color>
		    <!-- 蓝色 -->
		    <color name="mediumblue">#0000CD</color>
		    <!-- 中兰色 -->
		    <color name="darkblue">#00008B</color>
		    <!-- 暗蓝色 -->
		    <color name="navy">#000080</color>
		    <!-- 海军色 -->
		    <color name="black">#000000</color>
		    <!-- 黑色 -->
		
		
		    <!-- 自定义颜色 -->
		    <color name="cl_sidebar">#ffffff</color>
		    <color name="cl_page_bg">#f3f3f3</color>
		    <!-- 已使用 -->
		    <color name="hasuse">#ffff7D3D</color>
		    <color name="networkdata_down_book_text_color">#ff217f00</color>
		
		    <!-- 个人中心地区、学校筛选 -->
		    <color name="save_cancel_color">#07722A</color>
		
		</resources>

	**res/values/dimens.xml:**用于定义各组件的尺寸大小、字体大小等信息，定义格式如下：

		<resources>
		    <dimen name="update_app_log_title_textsize">22sp</dimen>
		    <color name="update_app_log_tilte_textcolor">#BF2E27</color>
		    <dimen name="update_app_log_info_textsize">20sp</dimen>
		    <color name="update_app_log_info_textcolor">#6E478C</color>
		    <color name="update_dialog_bg_color">#00FFFFFF</color>
		</resources>

	**res/values/strings.xml：**用于定义字符串信息，如果你的程序支持多语种，可以新建多个strings-*.xml，每个文件存放对应语种的字符串信息，各语种的简码参见[语种简码](http://www.loc.gov/standards/iso639-2/php/code_list.php)，strings.xml文件定义格式如下：

	<?xml version="1.0" encoding="utf-8"?>
	<resources>
	    <string name="cancel">取消</string>
	    <string name="update">更新</string>
	    <string name="app_id">=546db563</string>
	    <string name="loading">jiā&#160;zǎi&#160;zhōng\n加&#160;&#160;载&#160;&#160;中</string>
	</resources>

	**res/xml：**用于存放app作为桌面插件的配置文件。

- **AndroidMainfest.xml:**工程的配置文件，记录应用中所使用的各种组件。这个文件列出了应用程序所提供的功能，在这个文件中，你可以指定应用程序使用到的服务(如电话服务、互联网服务、短信服务、GPS服务等等)。另外当你新添加一个Activity的时候，也需要在这个文件中进行相应配置，只有配置好后，才能调用此Activity。AndroidManifest.xml将包含如下设置：application permissions、Activities、intent filters等。AndroidManifest.xml文件中的每一种标签属性的意义可参见[App Manifest](http://developer.android.com/guide/topics/manifest/manifest-intro.html)

- **project.properties:**里面包含程序编译目标版本信息、依赖的jar包路径。

**注：**关于工程下每一个目录的更详细解释可到Android Developer官网查看[Providing Resources](http://developer.android.com/guide/topics/resources/providing-resources.html)

## project引用libproject和lib
&emsp;&emsp;Android中有三种类型的工程，分别是：project、libproject、lib（准确来说是两种，一种是project，另一种是library），其中只有project才能运行，project可以依赖libproject和lib，通常lib工程会被打包成jar包的形式存放在project的libs文件夹下，project打包成jar包的方法参见：[Android将Activity打成jar包供第三方调用（解决资源文件不能打包的问题）](http://blog.csdn.net/xiaanming/article/details/9257853)。

**经验：**

- 如果你工程的libs文件夹下有*.so文件，建议不要将其打包成jar包，因为将资源文件和so打包在jar包里面是一件非常麻烦的事情，个人建议这种情况将其打包成libproject，一是方便更新，而是没有必要将时间浪费在这上面；

- 你工程引用libproject的方法：右键你的工程 -> Properties -> Android -> Library -> Add -> 选择你需要添加的libproject，点击ok即可导入；

- 在你的工程中引用jar包，这和eclipse的配置环境有关，我的eclipse直接将jar包拷贝到工程的libs文件夹下即可，有的IDE需要手动添加；

- 将你的工程打包成libproject的方法：右键你的工程 -> Properties -> Android -> Library -> 勾选 Is Library,点击OK。

## 几种工程框架搭建方式介绍
&emsp;&emsp;一个项目工程的可读性完全取决于你的程序框架和编码规范，关于成员变量、方法、包等等这些的命名，每个公司都有自己的编码规范，如果没有可以参见谷歌的Java编码规范[英文版](http://google-styleguide.googlecode.com/svn/trunk/javaguide.html)、[中文版](http://www.hawstein.com/posts/google-java-style.html)。

&emsp;&emsp;关于程序框架的搭建，基本上没有人会告诉你该如何做，这里有一篇比较好的文章，介绍了几种值得参考的搭建框架的方式[App工程结构搭建：几种常见Android代码架构分析](http://www.cnblogs.com/qianxudetianxia/archive/2011/06/26/2088503.html)，我将它引用了过来：

&emsp;&emsp;关于Android架构，因为手机的限制，目前我觉得也确实没什么大谈特谈的，但是从开发的角度，看到整齐的代码，优美的分层总是一种舒服的享受的。

&emsp;&emsp;从艺术的角度看，其实我们是在追求一种美。

&emsp;&emsp;本文先分析几个当今比较流行的android软件包，最后我们汲取其中觉得优秀的部分，搭建我们自己的通用android工程模板。

&emsp;&emsp;**1. 微盘**

&emsp;&emsp;微盘的架构比较简单，我把最基本，最主干的画了出来：

![微盘](http://img.my.csdn.net/uploads/201412/15/1418648070_4411.png)

&emsp;&emsp;第一层：com.sina.VDisk：com.sina(公司域名)+app(应用程序名称) 。

&emsp;&emsp;第二层：各模块名称（主模块VDiskClient和实体模块entities）

&emsp;&emsp;第三层：各模块下具体子包，实现类。

&emsp;&emsp;从图中我们能得出上述分析中一个最简单最经典的结构，一般在应用程序包下放一些全局的包或者类，如果有多个大的模块，可以分成多个包，其中包括一个主模块。

&emsp;&emsp;在主模块中定义基类，比如BaseActivity等，如果主模块下还有子模块，可以在主模块下建立子模块相应的包。说明一点，有的时候如果只有一个主模块，我们完全可以省略掉模块这一层，就是BaseActivity.java及其子模块直接提至第二层。

&emsp;&emsp;在实体模块中，本应该定义且只定义相应的实体类，供全局调用(然而实际情况可能不是这样，后面会说到)。在微盘应用中，几乎所有的实体类是以 xxx+info命名的，这种命名也是我赞成的一种命名，从语义上我觉得xxxModel.java这种命名更生动更真实，xxxModel给我一种太机 械太死板的感觉，这点完全是个人观点，具体操作中以个人习惯为主。还有一点，在具体的xxxInfo,java中有很多实体类中是没有get/set的方 法，而是直接使用public的字段名。这一点，我是推荐这种方式的，特别是在移动开发中，get/set方法很多时候是完全没有必要的，而且是有性能消 耗的。当然如果需要对字段设置一定的控制，get/set方法也是可以酌情使用的。

&emsp;&emsp;**2. 久忆日记**

&emsp;&emsp;相比于微盘的工程结构，久忆日记的结构稍微复杂了一些。如下图：

![9e](http://img.my.csdn.net/uploads/201412/15/1418648070_6235.png)

&emsp;&emsp;1).第一层和前面微盘一样的.

&emsp;&emsp;2).第二层则没有模块分类,直接把需要的具体实现类都放在下面，主要日记的一些日记相关的Activity。

&emsp;&emsp;3).第二层的实体包命令为model包，里面不仅存放了实体类xxx.java，而且存放了更高一级的实体类的相关类，比如xxxManager.java,xxxFactory.java.关于这一点，我们可以参考一下android.jar中结构，我们发 现，Activity.java和ActivityManager.java，View.java和 ViewManager.java，Bitmap.java和BitmapFactory.java等等N多类似的一对类都在同一个包下，我个人觉得实体 包下存放实体类相应的Manager和Factoty类也是正确的，是我们应该采纳的一种结构。这里就打破了前面微盘中说的实体包下存放且只存放实体类的 说法了。在现实中，从灵活和合理的角度，久忆日记的这种实体包中存放对象内容更加实用。

&emsp;&emsp;4).第二层中增加了前面微盘中没有涉及到的config，constant和common包。第一，其中config中存储一些系统配置，比如名称，应 用参数等系统级的常量或者静态变量，当然，如果你有其他大模块的配置，比如如果拥有复杂的用户管理模块的话，完全可以增加一个 UserConfig.java中存储用户的一些配置信息等等。第二，constant包，此包下存放的都是public static final常量，定义状态，类型等等。出于性能考虑，android开发中我不推荐使用枚举。common包中定义一个公用库，这里因为应用单一，无法很 好的说明common包内容结构。

&emsp;&emsp;5).common包要涉及后面多个软件比较后我们再得出结论。

&emsp;&emsp;通过久忆日记的分析，借鉴到了不少的东西，使我们的架构更丰满更强大了。

&emsp;&emsp;**3. 网易新闻**

&emsp;&emsp;网易新闻确实做的不错，从应用的角度看，是我最欣赏的应用之一。它的工程结构是怎么样的呢？

![netease](http://img.my.csdn.net/uploads/201412/15/1418648071_4587.png)

&emsp;&emsp;网易新闻的工程结构和前面2各app又有很多的不同，它并没有按照模块来分，而是主要按照组件的类型来分的，然后把此类型所有的类全部放在其下。那么这种把所有activity全部放在activity包下的分法的确在android开发中比较普遍。

&emsp;&emsp;1).第一层被分成了两层，可以看出来，这里肯定是采用了公用包jar，如此说来，我们开发公用包的时候也应该按照"公司域名+公用模块名称"组合方式来命名比较好。

&emsp;&emsp;2).第三层(绿色层)中activity和service包下都是存放所有的activity组件和service组件，其实这里面包含了一种代码习 惯。往往activity相关的类如监听器，线程，适配器等非常多的类，这些不好直接丢在activity包下，而是直接写在相应的activity中以 匿名或者内部类形式定义，否则activity包和service包看上去会比较杂乱。

&emsp;&emsp;因为android的app很可能不是很大，activity或者service包也不会杂乱，所以网易新闻的这种方式也是很有参考借鉴价值的。

&emsp;&emsp;**4. 小米应用**

&emsp;&emsp;小米应用包括3个应用，小米分享，小米阅读，小米标签，从实际代码开发来看，我感觉不是同一个团队，或者同一组人开发的。 这种情况下，他们的架构又使如何？

![miyue](http://img.my.csdn.net/uploads/201412/15/1418648072_7686.png)


![miqian](http://img.my.csdn.net/uploads/201412/15/1418648073_4046.png)

&emsp;&emsp;上面的结构以及结构内部的细节其实很多地方我都是不大苟同的，但是能做出来好东西就是值得大家学习的，所以我只把其中我认为最值得学习的一点拿出来说。

&emsp;&emsp;首先，widget，provider这些特殊模块分类建立单独的模块包即可，这里久不多说什么。

&emsp;&emsp;第二，通过观察，我们发现小米分享中每个应用都有common包，不仅有应用程序级别的common包，而且有应用程序内级别的common包。我想说的 是，android开发中随着项目开发的积累，确能提取到很多公用的方法、类、功能模块。各个项目之间如此，各个项目内部也是如此，所以针对项目类被各个 模块调用的方法，类也可以提取出相应的公用库。

&emsp;&emsp;那么这里有个问题，公用common包的内部包可能涉及到很多的内容，是否要分包分级呢，又如何分包分级？我觉得，这个因情况而已，一般来说移动开发，为 了减少包的大小，我们会控制common包的膨胀，往往common包仅仅包括一些最简洁最经典的东西，东西又很少的话就无需分包，但是如果贵公司开发成 百上千，每个项目都用到行为分析，意见反馈等公用模块，分一下包会更清楚一点。总而言之，分不分包无关紧要，尽量让你的代码结构清晰，思路了然就好。

&emsp;&emsp;**5. 聚各家之长，集大家之成**

&emsp;&emsp;上面粗略的分析之后，我们应该对android程序的架构有一个感觉，清晰而杂乱。我也没有去了看更多其他应用的结构，暂时就总结一下，得出一个我们自己的通用的工程结构。

&emsp;&emsp;假如公司名为tianxia，目前公司准备开发读书应用，交友应用，生活服务应用。

&emsp;&emsp;第一时间我们应该得出下面这种整个的架构(具体的app开发当然要分开)：

![tianxia](http://img.my.csdn.net/uploads/201412/15/1418648086_4717.png)

&emsp;&emsp;公司下开发3个应用reader，friend，life，其中common包为这三个应用共用，config，oauth为可选，view存放一些最通 用的自定义view，比如对话框，定制的列表等，如果你觉得有些view可能不会通用，最好把它放在应用程序类的common包下。

&emsp;&emsp;如果各位看过Android学习系列(6)--App模块化及工程扩展的话，对于这种多应用模式，应该存在android库共用情况，来解决资源替换，工程复用的问题。所以我又修改如下：

![tianxia1](http://img.my.csdn.net/uploads/201412/15/1418648086_4324.png)

&emsp;&emsp;其中BaseApplication做一些所有app都会用到的基础初始化或者配置。之后其他应用的application应该都继承此BaseApplication。

&emsp;&emsp;base是一个android库，也是一个完整的android工程，而common只是一个jar文件，当然你也可以根据需要作为android库来开发。其他主工程reader，friend，life应该引用base工程。

&emsp;&emsp;ad包存放公司自定义的一些软广告。

&emsp;&emsp;feedback包下存储一些用户反馈等通用功能模块。

&emsp;&emsp;其实，很多情况下，upgrade模块也可以添加到base工程下，制定统一的软件升级机制。

&emsp;&emsp;接下来我们以reader为例子，来详细完成它的工程结构的设计。

![tianxia2](http://img.my.csdn.net/uploads/201412/15/1418648086_9208.png)

&emsp;&emsp;其中，config包下的AppConfig.java存放应用程序的根配置，比如版本，目录配置等等。

&emsp;&emsp;module包下分为各个模块，blog为博客模块，bbs为论坛模块，person为整站个人信息模块，widget表示一种特殊功能模块。

&emsp;&emsp;common包下存放一些工具类，本应用程序的一些自定义View等等。

&emsp;&emsp;再结合之前所讲的内容，我们把整个串起来，完善一个reader的最后的架构如下(两外两个freind和life亦是类似如此)：

![tianxia3](http://img.my.csdn.net/uploads/201412/15/1418648087_7220.png)

**注意：**

&emsp;&emsp;1).功能模块和类型模块均可以划分，如果没有需要的话，模块的划分都可以省略。

&emsp;&emsp;2).activity和service这类组件划分，如果没有需要的话，组件的划分都可以省略。

&emsp;&emsp;3).所有的划分，如果没有需要的话，所有的划分都可以省略。

&emsp;&emsp;但是，但是，这种分类，我个人还是觉得层次清晰，架构明朗，值得参考的，当然其中很多细节我也没有仔考虑，如有不妥，还请发现者指出。


## 如何获取各个资源文件夹下的内容
&emsp;&emsp;对于res文件夹下的资源文件，可以通过如下两种方式来获取：

- 在xml文件中通过资源ID获取，获取格式一般为"@资源类型/资源ID名称"；

- 在代码中通过Resources类来获取，获取格式一般为（假定你是在activity中获取）getResources().getXXX()。

&emsp;&emsp;在xml文件中引用或者在代码中读取资源文件的方式如下表格所示：

<table>
	<tbody>
		<tr><td>类型</td><td>名称</td><td>xml文件中引用范例</td><td>代码中使用范例</td></tr>
		<tr><td>图片、selector</td><td>drawable</td><td>@drawable/main_background</td><td>getResources().getDrawable(id)</td></tr>
		<tr><td>字符串</td><td>string</td><td>@string/app_name</td><td>getResources( ).getString( id )</td></tr>
		<tr><td>颜色值</td><td>color</td><td>@color/blue</td><td>getResources( ).getColor( id )</td></tr>
		<tr><td>大小</td><td>dimen</td><td>@dimen/text_size</td><td>getResources( ).getDimension( id )</td></tr>
		<tr><td>数组</td><td>array</td><td>@array/splash_drawable_array</td><td>getResources( ).getIntArray(id)
        getResources().getStringArray(id)
		getResources().getTextArray(id)
        getResources().obtainTypedArray(id)</td></tr>
		<tr><td>主题</td><td>style</td><td>@style/transparent_style</td><td></td></tr>
		<tr><td>布局</td><td>layout</td><td>@layout/list_header_layout</td><td>getResources( ).getLayout(id)</td></tr>
		<tr><td>动画</td><td>anim</td><td>@anim/loading_anim</td><td>getResources( ).getAnimation(id)</td></tr>
		<tr><td>属性</td><td>attr</td><td>@attr/text_attribute</td><td></td></tr>
		<tr><td>菜单</td><td>menu</td><td>@menu/action_menu</td><td></td></tr>
		<tr><td>raw</td><td>raw</td><td>@raw/background_voice</td><td></td></tr>
		<tr><td>xml</td><td>xml</td><td>@array/setting_preferences_xml</td><td>getResources( ).getXml(id)</td></tr>
		<tr><td>自定义属性</td><td>declare-styleable</td><td>@declare-styleable/paint_view_styleable</td><td></td></tr>
	</tbody>
</table>

&emsp;&emsp;读取/加载assets文件夹下的文件：

- 读取assets文件夹下的文件：
	
		public static String getFromAssets( Context context ,String fileName){
	    	if( null == context || TextUtils.isEmpty( fileName ) ){
	    		return null;
	    	}
	    	
	    	StringBuilder result = new StringBuilder( );
	        try{ 
	            InputStreamReader inputReader = new InputStreamReader( context.getResources().getAssets().open(fileName) ); 
	            BufferedReader bufReader = new BufferedReader(inputReader);
	            String line="";
	            while((line = bufReader.readLine()) != null){
	            	result.append( line );
	            }
	        }catch (Exception e){ 
	            e.printStackTrace(); 
	        }
	        
	        return result.toString( );
	    }

- 读取assets文件夹下的自定义字体文件（建议将字体文件放在assets/fonts/文件夹下）：
		
		Typeface mTypeface = Typeface.createFromAsset( contex.getAssets( ), "fonts/pinyinok.ttf" );// "fonts/pinyinok.ttf"为assets文件夹下字体文件的路径

- 通过webview加载assets文件夹下的本地flash、网页：
		
		// load html
		mWebView.loadUrl("file:///android_asset/swfFile.html");
		// load swf
		mWebView.loadUrl("file:///android_asset/qualibus.swf");


## 如何阅读Android项目代码
&emsp;&emsp;当你参与一个已经在运行的项目或者是拿到一个开源项目时，通常都需要通过去阅读原来理解其中的逻辑和框架（个人觉得类图、说明文档都不如阅读源代码好使），通常我们只需要找到程序的入口，然后按照入口一步一步地去阅读源代码，所以阅读项目代码的首要工作是找到项目的入口。如上所讲，Android中的四大组件都需要在配置文件中定义（recevier也可以在代码中动态注册），所以阅读android项目代码可以从AndroidManifest.xml入手，我在阅读android项目代码时通常是按照如下步骤进行的：

1. 在AndroidManifest.xml文件中找到程序主入口，主activity的intent-filter标签会包含如下内容：

		<intent-filter>
	        <action android:name="android.intent.action.MAIN" />
	        <category android:name="android.intent.category.LAUNCHER" />
	    </intent-filter>

2. 如果项目自定义了application，会在AndroidMainifest.xml的application标签中注册该application，比如Eoe的android客户端源码的application标签如下，包名+.MyApplication就是application这个类的具体位置了，通常项目的初始化工作和监听每个activity的生命周期、系统回调都在自定义application里面完成，比如初始化广告、后台统计、语音、社会化分享等等：

		<application
	        android:name=".MyApplication"
	        android:allowBackup="true"
	        android:icon="@drawable/ic_launcher"
	        android:label="@string/app_name">
		</application>

3. 在AndroidMainfest.xml文件中注册的每一个组件的具体路径是package+name，但有的项目会将name写成完整的项目路径，根据路径去找会比较麻烦，可以通过按住CTRL+鼠标点击该组件名称即可进入到对应的类中进行查看；

4. 找到主Activity后，在onCreate方法中找到布局文件，按Ctrl+点击鼠标查看该布局文件，了解整个界面的布局，用到了哪些组件、每个组件的ID等等，从而有助于理解代码，其次是阅读每一个事件触发方法体，程序是动态的，只有用户有操作才会去响应相应的方法，Android中的事件触发有两种：

	- 一种是系统回调：比如生命周期中的每个回调方法、物理按键（开关键、声音键）点击响应、虚拟按键（返回、Home、Menu、键盘按键点击）点击响应、onDraw方法等等；

	- 一种是用户触发事件：比如按钮点击、刷新脏区域、启动其它组件等等。

5. 如果有GridView、ViewPager、ListView，通过其SetAdapter方法找到对应组件的自定义Adapter，进入Adapter查看每一项的布局；

6. 阅读其它组件(Receiver、Broadcast、ContentProvider)的方法类似；

7. 查看是否有aidl文件、是否引用了libproject、项目的libs文件夹下是否引用了第三方jar包等等。

8. 如果程序有桌面插件，根据AndroidManifest.xml文件中定义的桌面插件receiver进行查看，可参考的搜索标签有：“android.appwidget.provider”、“android.appwidget.action.APPWIDGET_UPDATE”

8. 对于不能清晰理解的地方，查看注释、说明文档和类图；

&emsp;&emsp;在阅读Android项目代码时，只要找到项目入口，其它一切就好办了。关于如何阅读Android项目代码和Android系统源码可参见这两个答案：

- [拿到 Android 项目源码后，如何才能以最高效的速度看懂？](http://www.zhihu.com/question/20181899)

- [大牛们是怎么阅读 Android 系统源码的？](http://www.zhihu.com/question/19759722)