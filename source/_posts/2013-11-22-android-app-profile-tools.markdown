---
layout: post
title: "初探android应用性能分析"
keywords: "android, app, profile, tools, performance"
description: "通过通过服务端开发人员的视角，对安卓(android)手机上应用(app)的渲染性能进行分析，找到性能瓶颈"
date: 2013-11-22 18:03
comments: true
categories: [mobile]
tags: [mobile, android, profile]
---
如果一个android应用打开时比较慢，或者使用起来比较卡。这个可能是客户端代码有待优化，也可能是服务端性能比较挫。对一个客户端开发者而言，在客户端代码中增加相关debug日志，即可比较准确地定位问题。但这活要落到一个服务端开发人员手里，要怎么办？

本文将在没有apk源码的情况下，以服务端开发人员的视角进行客户端app性能的分析。
<!-- more -->
在分析之前，我们先补充点android基础知识。
### android基础知识 ###
我们所说的android应用，一般都是通过将一个以apk结尾的文件安装在手机等移动设备上才能运行起来的。所以我们先从apk说起。

##### 什么是apk #####

我们先从网上下载一个apk
    $ wget http://shouji.360tpcdn.com/131106/0124832c4cf8c35a762cfece3bac52b1/com.sina.weibo_650.apk

然后查看这个文件的类型
    $ file com.sina.weibo_650.apk
    com.sina.weibo_650.apk: Zip archive data, at least v2.0 to extract

会发现`com.sina.weibo_650.apk`是一个zip压缩文件。解压缩后的文件，主要包括*一些资源文件*，*一些配置文件*，*一些类库*，还有*一个class.dex*。目录结构如下
    AndroidManifest.xml
    assets
    classes.dex
    lib
    META-INF
    org
    res
    resources.arsc

粗略一看，发现 `class.dex` 这个文件有5.9M，这应该就是主角。继续执行如下命令
    $ file classes.dex
    classes.dex: Dalvik dex file version 035

因为没有开发过android应用，不明白用java开发的app和这个Dalvik dex file之间有什么关系？所以我们先跳出apk的视角。

##### android平台架构 #####

{% img /images/mobile/android_architecture.png 'android architecture images' %}

如上图，android基于linux操作系统，使用linux内核与设备的硬件进行交互。在内核之上，又抽象出了一层，包括Dalvik虚拟机等。

因为`dex`是`Dalvik VM` Executes的全称，即android `Dalvik`虚拟机执行程序。

那一个apk的生产和执行过程将是：
`*.java -> *.class -> classes.dex（classes.dex将由Dalvik VM转换成机器码，由linux内核交给cpu去执行）`

这样的话，在linux系统上使用profile软件的经验，也将派上用场。

android相关基础知识先介绍到此，感兴趣的请进一步查阅本文后面的参看资料。
### android应用性能分析 ###

##### apk启动速度 #####

在分析之前，我们先看看android程序的执行流程
{% img /images/mobile/android_application_execute_flow.png 'android application execute flow images' %}

如上图，只要获取到启动ActivityManager所需要的时间，即是apk的启动时间。

    adb logcat | grep ActivityManager
其中"Displayed"对应的时间，即是launch Activity对应的时间，也就是apk启动时间，也可以使用如下命令：

    adb logcat -c && adb logcat -s ActivityManager | grep  "Displayed"
* 要使用 `adb`，需要先用usb线连接电脑和手机，并在手机的`设置`->`开发者选项`中开启`USB调试`
* `adb`这个工具，可以通过在android sdk的platform-tools目录中找到。后面介绍的`systrace`也在这个目录。

##### 页面渲染性能 #####

android应用中的页面，是由android系统一帧，一帧地绘制的，其中每一帧的处理如下图：
{% img /images/mobile/android_view_execute_flow.png 'android view execute flow images' %}

即：
`计算视图大小（measure） -> 安置视图的位置（layout） -> 绘制（draw）视图`

通过收集每帧的处理时间，即可以了解页面的渲染性能。

>当fps（每秒处理帧数，页面刷新率）为60时，页面的渲染看起来会比较平滑，这就需要每帧的处理时间不能大于16ms（1000/60）

要检测一个应用在渲染页面时的每帧处理时间，通过如下命令，即可获得每帧的处理时间
    adb shell dumpsys gfxinfo com.sina.weibo

在输出日志的`Profile`数据段，包含了三列`Draw`，`Process`，`Execute`分别对应的处理时间，单位是ms。这三列的总和，就是渲染每帧时的处理时间。如
    Draw	Process	Execute
    0.95	0.93	0.72
    0.84	1.16	0.56
    0.83	0.89	0.69
    1.32	2.15	1.14
    1.29	1.37	1.01
>在进行分析之前，需要在`设置`->`开发者选项`中点击`GPU呈现模式分析`，选择`在adb shell dumpsys gfxinfo中`。

收集步骤：
    1.重新启动app
    2.在界面完全加载完之后，在界面上慢慢上下滑动几个像素
    3.在终端执行adb shell dumpsys gfxinfo com.sina.weibo
这时将在终端输出页面渲染时的最后128帧中每帧所花费的时间，将相关数据贴到excel表格中，点击其中的`insert`->`chart`，即可生成相关图表
{% img /images/mobile/frame_render_time.png 'frame render time images' %}

>其中`com.sina.weibo`就是app的包名

获取包名的方法:
    adb shell pm list packages

##### 使用systrace进一步分析  #####
通过收集该apk的启动速度和每帧的渲染时间，并与竟品进行对比发现。该app启动时间的确比较慢，也偶尔有丢帧的现象发生。如何近一步分析呢？这时就需要`systrace`了。

示例使用方法如下：
    cd android-sdk-linux/platform-tools/systrace
    python systrace.py --app=com.qihoo.appstore gfx view

上面这条命令将会在`android-sdk-linux/platform-tools/systrace`目录下生成`trace.html`。其中收集了包名为`com.qihoo.appstore`的应用在android系统上针对`gfx`和`view` category的执行数据。

`trace.html`在浏览器中打开如下图：
{% img /images/mobile/android_systrace_output.png 'android systrace output images' %}

可以使用如下方法，对`trace.html`进行进一步分析：
* 通过鼠标点击左侧的`+`，`-`可以展开或者收缩相关显示数据
* 通过键盘上的`a`，`d`可以使显示的内容沿着顶部的时间轴向左或者向右移动
* 通过键盘上的`w`，`s`可以对显示的内容进行放大或者缩小
* 使用鼠标点击内容页面的某一个块，在下方会显示详情
* 使用鼠标选择一块内容页面，在下方会显示汇总信息


将光标定位到最后一行，使用`w`进行放大，使用`d`向左移动到2260ms左右，如下图：
{% img /images/mobile/android_systrace_output_zoom.png 'android systrace output detail images' %}

发现对于那些`performTraversals`处理超过16ms的帧，其中`eglSwapBuffers`处理的时间都比较长，这应该就是问题所在。

使用usb线连接上手机，在命令行下运行：
    python systrace.py -h

可以查看相关使用方法。
> systrace是在在android4.1上新增的工具，在4.1,4.2和4.3上使用的方法不同

reference：

[^1] http://www.curious-creature.org/docs/android-performance-case-study-1.html

[^2] http://www.curious-creature.org/docs/android-performance-case-study-1.html

[^3] http://www.vogella.com/articles/AndroidTools/article.html

[^4] http://blog.csdn.net/aaa2832/article/details/7849400

[^5] http://www.cnblogs.com/taobox/articles/3405931.html

[^6] http://bigflake.com/systrace/

[^7] http://developer.android.com/tools/debugging/systrace.html

[^8] http://kitoslab-eng.blogspot.com/2013/01/aprof-android-profiler-profiling-tool.html

[^9] http://udinic.wordpress.com/tag/rendering/
