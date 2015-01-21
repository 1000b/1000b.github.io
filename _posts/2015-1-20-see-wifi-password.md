---
layout: post
title: Android通过ADB查看wifi密码
category: Android
tags: Android
keywords: ADB，WIFI密码，adb shell，/data/misc/wifi/*.conf
description: Android通过ADB查看wifi密码
---

&emsp;&emsp;在同一个生活环境，有时候wifi密码忘记了，但有时会有新的设备需要连接WIFI怎么办？

&emsp;&emsp;我之前做过一个WIFI密码分享工具专门针对这种需求。但是由于需要设备获取ROOT权限才能正常使用，涉及到个人隐私就没有发布，下面把原理记录下来，方便后面有需要的时候使用。

&emsp;&emsp;Android设备中wifi密码是保存在/data/misc/wifi/文件夹下的的conf文件中的，我们可以通过adb和DOS的cat命令来查看当前设备已经成功连接过的WIFI设备及密码。（下面假设设备已经和电脑相连、设备已经ROOT并且PC端ADB的环境变量已经配置OK）。

1. WIN + R，输入cmd回车；

2. adb devices查看连接到电脑上的设备，如果设备已经连接成功会被列出来，如果已经有列出的设备，请继续步骤3，否则请将android设备和PC成功连接；

3. 输入adb shell回车，然后输入：
	cat /data/misc/wifi/*.conf

4. 步骤3会将设备已经成功连接的wifi账号和密码显示出来（ssid后面跟的是账号，psk后面跟的是密码），这样你就可以将已经忘记了的密码分享给其它人了。

**PS：**如果你是普通用户，在已经取得ROOT权限的Android设备上安装好RE文件管理器，按照上面所描述的路径直接打开*.conf格式的文件即可查看。
