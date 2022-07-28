---
layout: post
title: mac终端穿墙技术汇总
date: 2016-11-17 10:32:39
description:
keywords: "mac terminal 翻墙 穿墙 ss shadowsocks socks5 vpn gfw Proxifier proxy Surge"
comments: true
categories: [devops]
tags: [devops, gfw]
---
作为一个互联网从业人员，要想在天朝愉快地工作，生活，目前必须正视墙（gfw）的存在。
本文涉及的翻墙方法，主要针对mac系统。但大部分内容，同样适用于window和linux。也可以自行寻找相关替代品。
<!--more -->

# 土豪的方法 #
如果你是一个使用mac系统的土豪，那么，访问这个网站 http://nssurge.com/ 就够了。

# others #
实际上，这才是本文的重点。

## 准备工作 ##
我们需要先搭建一个ss（Shadowsocks）服务器，或者买个账号（https://shadowsocks.com/）。

### 搭建ss server###
* 买个海外的云主机，各大云的香港或者海外节点，应该都能满足需求。
* 安装 ss server
在云主机的命令行下，执行如下命令
```
    pip install shadowsocks
```
或者
```
    git clone https://github.com/shadowsocks/shadowsocks.git
    cd shadowsocks
    python setup.py
```
* 配置
创建配置文件，如`/etc/shadowsocks.json`，示例内容如下
```
{
    "server":"my_server_ip",
    "server_port":8388,
    "local_port":1080,
    "password":"barfoo!",
    "timeout":600,
    "method":"table",
    "auth": true
}
```
> 配置文件是json格式，注意最后一行没有`,`

* 启动ss server
`ssserver -c /etc/shadowsocks.json -d start`
* 停止ss server
`
ssserver -c /etc/shadowsocks.json -d stop
`

### 安装ss 客户端 ###
推荐 [ShadowsocksX-NG](https://github.com/shadowsocks/ShadowsocksX-NG)，因为 [ShadowsocksX](https://github.com/shadowsocks/shadowsocks-iOS/releases)已无法正常更新pac文件。

至此，使用浏览器的话，就可以自由地在互联网上遨游了。当然，你会发现更好用的翻墙技术。

## 终端(terminal)翻墙##

### 亲测可用的方案 ###
下载软件 [proxifier](https://www.proxifier.com/download.htm)，仅支持windows和mac，收费软件。

如果是学生的话，可以给我留言，我共享个注册码给你。其他人建立购买，在这物价横飞的时代，几百块，分分钟就花没了。

shadowsocks代理属于socks5代理，通俗的理解，socks5只是局部代理。使用Proxifier把shadowsocks代理转全局代理，类vpn。所以，该方案实际上不局限于终端翻墙。
### 其他方案 ###

[proxychains-ng](https://eliyar.biz/proxy-for-mac-terminal/)
[tsocks](https://mba811.gitbooks.io/web-study/content/fq/fq3.html)


# reference #
[^1] https://shadowsocks.org/

[^2] https://github.com/shadowsocks

[^3] https://shadowsocks.com/

[^4] https://www.dou-bi.co/ss-jc7/

[^5] http://www.voidcn.com/blog/shenshouer/article/p-6254512.html
