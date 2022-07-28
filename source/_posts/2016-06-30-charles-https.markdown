---
layout: post
title: "使用charles在移动设备上捕获https数据包"
keywords: "charles, https, ios"
description: "通过使用charles设置代理，捕获移动设备上捕获的https数据包"
date: 2016-06-30 16:38:16
comments: true
categories: [devops]
tags: [devops, ios, charles, https]
---
对于互联网从业人员而言，掌握抓包，是必备技能。
<!--more-->

`Charles`是一个http代理，工作模式如下图：

{% img /images/mobile/http_proxy.png '抓包' %}

但是默认只能抓http协议的数据包，要捕获https的数据包，需要进行相关配置。

下文以`IOS`移动设备为例，讲述配置步骤（`android`设备类似）：

* 在移动设备安装ssl证书

Charles ssl证书的下载地址如下：

[http://www.charlesproxy.com/getssl](http://www.charlesproxy.com/getssl)

在移动设备的浏览器中打开上述`Url`，即可进行安装。

* 安装http代理`Charles`

软件下载地址如下：
[http://www.charlesproxy.com/latest-release/download.do](http://www.charlesproxy.com/latest-release/download.do)

* 启用http代理

打开`Charles`软件，默认会启动一个监听本地8888端口的http代理， 也可以在`Charles`的设置中修改相关端口。

* 配置`Charles`支持https

在`Charles`中打开：

    Proxy -> SSL Proxying Settings

勾选

    Enable SSL Proxying

然后在下方的`Locations`中点击

    Add

添加需要抓https接口的域名。

例：

```
Host:www.baidu.com
Port:443
```

* 在移动设备上修改代理

{% img /images/mobile/ios_http_proxy.png 'ios设置代理' %}

其中，`服务器`是安装了`Charles`软件的电脑的`IP`，端口是`Charles` http代理开启的端口。
