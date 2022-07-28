---
layout: post
title: "手动刷入android 4.4.3 OTA 更新包"
description: "下载安卓4.4.3 ota更新包，通过adb手动更新到最新的安卓系统"
keywords: "andriod 4.4.3 KitKat nexus4 ota update"
date: 2014-06-08 06:35
comments: true
categories: [mobile]
tags: [mobile]
toc: true
---
今天上午，nexus4终于收到了google android 4.4.3 的ota更新包，但是从上午到晚上，愣是没有下载下来。这就像因为长智齿而牙龈肿痛的你被人请吃麻辣香锅那样难受。
<!-- more -->
作为一个吃货，怎么能受得了这份煎熬？就算用半边牙齿，也不能虚此行。

当然，作为码农。就算被铜墙铁壁包围，也要想办法越过长城，对世界说出那句“hello world”。

# 诊断 #
更新包为什么下不下来，这肯定是有原因的。对手机的网络请求进行抓包，应该可以查明原因。

如何对手机进行抓包，可以参见之前的博文“[移动应用无线抓包指南](http://jqlblue.github.io/2013/08/04/guide-of-packet-mobile-capture/)”。
如果手机使用的是家里的wifi网络，那对手机进行抓包会非常easy。两步即可：
    1. 在电脑上对fiddle进行设置
    2. 修改手机上的网络设置，设置代理，其中代理服务器的ip就是电脑的ip
> 如何设置可参见博文[移动应用无线抓包指南](http://jqlblue.github.io/2013/08/04/guide-of-packet-mobile-capture/)”

设置完成后，再请求时发现更新包的无法下载。
{% img /images/mobile/android-4.4.3-update.png android-4.4.3-update %}

这时有两种方案：
    1. 通过代理等途径，获取更新包域名的对应的ip，绑定host。
    2. 因为已经抓包获取到了更新包的下载地址，可以通过代理等途径，下载更新包并手动刷入。
> 因为在手机上设置的代理服务器是电脑的ip，所以只要在电脑上绑定host，手机上也会生效。

由于更新包下载地址的域名是动态的，所以没法绑定host。于是只有选择下载更新包，手动刷入。
# 下载4.4.3 OTA 更新号 #
为了方便，已下载针对nexus4的android4.4.3的ota更新包。需要的，可直接通过如下地址下载[android-4.4.3-ota](http://pan.baidu.com/s/1mgjxxLA#dir/path=%2Fsoft%2Fandroid-4.4.3-update%2Fkitkat-4.4.3-update)。
# 使用adb手动刷入OTA更新包 #
## 手动刷入的准备工作 ##
在手动刷入更新包时，除了下载更新包，还需要做如下准备工作：

* 在手机的`开发者选项`中，开启`USB调试`。

> 在`设置`，`关于手机`中，狂点`版本号`，可开启`开发者选项`。

* 在电脑上使用usb线连接手机

手机上应该会出现如下画面。选择`允许`

{% img /images/mobile/android-usb-debug.png 安卓usb调试 %}

使用usb连接手机后，电脑上可能会自动安装相关驱动程序，请耐心等待完成。

* 下载adb


`adb`包含在android的sdk中，但是我们只需要`adb.exe`, `AdbWinApi.dll`, `AdbWinUsbApi.dll`。

如果不想去下载android的sdk，可以通过如下地址下载[刷机adb](http://pan.baidu.com/s/1mgjxxLA#dir/path=%2Fsoft%2Fandroid-4.4.3-update%2Fadb)。

下载完成后，解压到某个目录，如`D:\soft\nexus4\Tools`，在命令行执行：
    cd D:\soft\nexus4\Tools
    d:
    adb.exe devices

如果看到下图，说明准备工作告一段落。如果没有，可能是相关驱动安装地有问题，可自行查阅解决。

{% img /images/mobile/android-adb-devices.png 安卓adb devices %}

## 开刷 ##

* 关机，然后按住`音量下键`和`电源键`，进入fastboot模式：


{% img /images/mobile/android-fastboot.png 安卓fastboot %}


* 通过按`音量上下键`进行切换，切换到`Recovery Mode`模式，按`电源键`选择进入：


{% img /images/mobile/android-recovery-mode.png 安卓recovery-mode %}


此时，你可以看到一个倒地的机器人：


{% img /images/mobile/android-recovery-mode-2.png 安卓recovery-mode %}


* 按`电源键`，然后再迅速按`音量上键`

> 这一步比较艰难，需要多尝试几次

直到看到如下界面：

{% img /images/mobile/android-apply-update.png 安卓adb update %}

再按`音量上下键`进行切换，切换到`apply update from ADB`，按`电源键`选择进入：

{% img /images/mobile/android-sideload.png 安卓sideload %}

* 通过USB再次连接电脑和手机

在命令行执行：
    adb.exe sideload kitkat-4.4.3.zip

{% img /images/mobile/android-adb-sideload.png 安卓 adb sideload %}

手机上将会出现如下界面：

{% img /images/mobile/android-update-ota-1.png 安卓 ota update %}

耐心等待，等ota更新包安装完成时，会出现如下界面，按`电源键`选择重启即可。

{% img /images/mobile/android-update-ota-2.png 安卓 ota update %}

重启后，会对已安装的应用进行优化。通过`设置`，`关于手机`查看系统版本，发现已经是`4.4.3`。

{% img /images/mobile/android-4.4.3-update-end.jpg  安卓 ota 更新完成 %}
