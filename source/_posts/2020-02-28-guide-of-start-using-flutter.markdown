---
layout: post
title: "flutter入坑指南"
description: "通过介绍flutter的技术架构和安装过程，带你领略flutter的魅力所在"
keywords: "flutter web android ios flutter 按照指南，flutter 入门"
date: 2020-02-28 03:20
comments: true
categories: "mobile"
tags: [mobile, flutter]
published: true
---
flutter是google推出的一个开源的ui框架(UI software development kit)，开发者可以使用flutter，通过dart语言来开发跨平台，高保真，高性能的`App`，目前已经支持的平台有： `Android`, `IOS`, `Windows`, `Mac`, `Linux`, `Google Fuchsia[5]`和 `Web`。
<!-- more -->
flutter的第一个版本(Alpha (v0.0.6))是在2017年5月份发布的。2018年12月4号，google发布了flutter1.0，支持android和ios平台。

ui框架，是一个比较抽象的概念。但是说到移动端跨平台的ui框架，我们可能会联想到`Hybrid`,`React-Native`等。所以我们以移动端为例，先尝试解释下ui框架。

如果将移动端的系统架构进行分层，大概如下。
{% img /images/flutter/mobile_struct.png mobile os architecture android os ios os struct %}

其中，对ios系统的每层架构进行细分，如下图
{% img /images/flutter/mibile_ios_struct_1.png mobile os architecture ios os layer architecture %}

对android系统的每层架构进行细分，如下图
{% img /images/flutter/mobile_android_struct.png mobile os architecture android os layer architecture %}

我们在android系统上开发出的apk，或者在ios系统上开发的ipa，主要涉及和平台特有的SDK，以及本地服务（蓝牙，摄像头，传感地，地理位置），与系统的交互如下图：
{% img /images/flutter/mobile_native_struct.png mobile os architecture android ios native oem sdk %}

如果要开发一款`App`，分别支持android平台和ios平台的。这样的话，就需要分别基于android和ios平台上特有的SDK进行开发。一款产品，需要维护两套代码。由于两个平台存在着不同的特性，很难在两个平台上对齐相同的交互体验。

所以后来出现了`Hybrid`，即混合开发的方案。可以基于IOS平台上的UIWebView和android平台上的WebView，使用HTML，CSS和Javascript进行开发，借助第三方的`Hybrid`框架，基本可以做到一套代码，多端运行。由于要使用本地的浏览器（UIWebView或WebView）进行渲染，还要使用第三方`Hybrid`框架提供的BRIDGE，与本地服务进行通信，所以性能会比较差。
{% img /images/flutter/mobile_hybrid_struct.png mobile os architecture android ios Hybrid %}

在后来就出现了`React-Native`等跨平台的方案，可以通过js-bridge与OEM组件，以及本地服务进行通信，较`Hybrid`的方案，可以带来更好的用户体验。
{% img /images/flutter/mobile_cross_platform_struct.png mobile os architecture android ios Cross-Platform REACT NATIVE %}

到这里，我们再次回归到本文的重点，flutter的方案，如下图：
{% img /images/flutter/mobile_flutter_struct.png mobile os architecture android ios flutter %}

flutter并不会直接编译成ios或者android应用程序。基于flutter开发的应用程序，包括两部分：

    Dart业务代码
    Flutter引擎代码

业务代码会经过frontend_server，gen_snapshot，xcrun，ninja编译工具，转换为具体相应系统架构（arm/arm64等）的二进制指令。相关编译产物如下：
{% img /images/flutter/mobile_flutter_struct_4.png flutter编译产物 %}

以android平台为例，基于flutter开发的应用程序的启动流程如下：
{% img /images/flutter/mobile_flutter_struct_5.png flutter应用程序启动流程 %}

    Flutter Application会通过onCreate完成初始化配置，加载引擎libflutter.so，注册JNI方法

    然后调用Flutter Activity的onCreate，通过FlutterJNI的AttachJNI()方法来初始化引擎Engine，Dart虚拟机，Isolate，taskRunner等对象。

    再经过层层处理最终调用main.dart中main()方法，执行runApp(Widget app)来处理整个Dart业务代码。

由于flutter使用自渲染引擎，不使用WebView和平台的原生控件，所以不仅保证了多个平台上ui的一致性，也保证高性能。flutter的技术架构如下
{% img /images/flutter/mobile_flutter_struct_2.png flutter应用程序技术架构 %}

通过Dart Framework层来统一Flutter C++引擎和Web引擎，最终就可以运行在Android，iOS，Browser上。

要开始使用flutter的话，非常方便。在flutter官网，可以选择不同平台，根据官网的指南进行安装，非常easy。

    https://flutter.dev/docs/get-started/install

reference：

[^1] https://book.flutterchina.club/chapter1/flutter_intro.html

[^2] https://flutter.dev/

[^3] http://gityuan.com/flutter/

[^4] https://www.intellectsoft.net/blog/mobile-app-architecture/

[^5] https://tech.meituan.com/2018/08/09/waimai-flutter-practice.html
