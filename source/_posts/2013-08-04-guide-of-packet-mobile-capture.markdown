---
layout: post
title: "移动应用无线抓包指南"
keywords: "安卓应用 手机抓包 无线抓包"
description: "如何通过无线网络在安卓手机上进行抓包"
date: 2013-08-04 11:41
comments: true
categories: [mobile, network]
tags: [mobile, wireshark]
---

### 引子  ###
如果将互联网比喻成纵横交错的铁道，那由c，h，o等元素组成的人和物则被打
包装进一节节车厢，然后组成一列火车在铁道上穿行。手机，笔记本等终端，可
以理解为车站，每天都有很多人进进出出。

有一天，你突然发现车站少了个东西，或者想了解下都有些什么人出站，什么人
入站。这时，我们就需要抓包（Capturing Packages）。

继续以铁路为例。一般情况下，我们只要在装载这人和物的火车入站和出站的时候设置关卡，
进行检查就可以了。
<!-- more -->
但突然有天发现某个车站是德国人给造的，我们要私自设置
关卡的话，就没有保修了。

实际上，我们私自设置关卡也无防。现在的东西更新换代太快，等坏的时候，直
接入手一个新的，或许比保修还经济呢。

等要动手的时候，想起这个德国制造，还是有些不忍。

在你正想点燃那支兰州的当儿，想起西边有你表哥建造的一个车站。所以可以代
理一把，让要进出的火车先去你表哥的车站。

### 抓包的三种方式 ###
至此，我们可以看到抓包的三种方式：

    1. 本地抓包
    2. 远程服务器抓包
    3. 代理抓包

### 本地抓包 ###
不管使用手机还是平板，进出的数据包，都会经过该设备的网卡。如果你的设备
已经root，可以使用tcpdump将抓包数据存成xxx.pcap，然后在电脑上就可以使
用wireshark进行查看。也可以使用webview
[cloudshark](http://www.cloudshark.org)。
更多信息请上google查询。

### 远程服务器抓包 ###
从设备上发出的请求，在网络通畅的情况下，最终都会达到某个服务器。所以我
们可以在远程服务器上抓包。可以使用tcpdump，但是我更推荐ngrep(network
grep)。

在centos上，直接
    yum install ngrep

下面是一些简单示例
    ngrep -t -d any port 25
    ngrep -q -W byline "(GET|POST).*"

更多用法请查看
    man ngrep

### 代理抓包 ###
开始废话说地有点多，这才是本文的重点。

设置代理，就是要在你的移动终端和某台电脑之间网络互通的情况下：
    1. 在电脑上设置代理
    2. 移动终端上网的时候连接这个代理
就可以在电脑上进行抓包了（*把一个陌生的概念转行成一个很熟悉的概念，
fiddle抓包嘛，码农应该都知道*）。

我们以移动终端与要进行代理抓包的电脑之间网络不通的情况为例进行说明（*如
果电脑和移动终端可以连接同一wifi，只要按照设置代理的部分进行操作就好*）

* 在电脑上创建无线网络

    我们使用360随身wifi在电脑上创建无线网络。要购买的话，现阶段需要时常关注
    [官网](http://wifi.360.cn/),因为不定期会在京东开启抢购。安装非常简
    单，插入usb接口，就自动创建好无线网络了（目前只支持windows系统）

    {% img /images/360wifi_1.png '360 wifi images' %}
    {% img /images/360wifi_2.png '360 wifi images' %}
    {% img /images/360wifi_3.png '360 wifi images' %}

* 在电脑上用fiddle设置代理

    {% img /images/360wifi_fiddle.png '360 wifiproxy fidlle images' %}

* 修改手机上的网络设置，并设置代理

    {% img /images/360wifi_proxy_1.png '360 wifi mobile network %}
    {% img /images/360wifi_proxy_2.png '360 wifi mobile network %}
    {% img /images/360wifi_proxy_3.png '360 wifi mobile network %}

    代理服务器的ip可以通过在电脑上查看网络连接获取，代理的端口就是在
    fiddle中设置的"listen on port"

    {% img /images/360wifi_proxy_ip.png '360 wifi mobile network %}
