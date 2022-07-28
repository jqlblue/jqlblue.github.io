---
layout: post
title: "在fedora上安装fcitx输入法和云拼音"
keywords: "fedora, fcitx, input method"
description: "如何在fedora操作系统上安装fcitx输入法并使用google云拼音"
date: 2013-10-26 12:30
comments: true
categories: [linux]
tags: [linux]
---
从fedora18开始，ibus感觉渐渐不如以前好用了，尤其是在emacs下使用的时候，经过死机。restart input method是家常便饭。

一次发现同事的ubuntu上在使用google输入法，让我眼前一亮。但是在64位的
fedora19上没有配置成功。于是尝试了下fcitx输入法，特此记录。
<!-- more -->
### 安装步骤 ###
{% codeblock fcitx install step lang:bash %}
yum install fcitx.x86_64
yum install fcitx-configtool.x86_64
yum install fcitx-gtk3.x86_64
yum install fcitx-cloudpinyin.x86_64
yum install fcitx-table-chinese.noarch
yum intall fcitx-qt4.x86_64
{% endcodeblock %}
### 配置 ###
编辑~/.bashrc，添加：

    export GTK_IM_MODULE=fcitx
    export QT_IM_MODULE=fcitx
    export XMODIFIERS="@im=fcitx"

重启系统或者logout，使之生效。

    如果将im_module设置为xim，系统重启时可能会造成应用程序卡死。
    此时可以通过键盘快捷键“CTRL+ALT+F2”，切换成tty2，
    通过console模式登录系统杀死fcitx进程，再切回X window，或者直接重启。

### 相关设置 ###
如下图所示：
    1. 在“input method”Tab，可以添加或者删除输入法
    2. “Global Config”Tab，主要用于设置相关快捷键
    3. “Appearance”列用于设置输入法弹出框的显示界面
    4. 点击“Addon”Tab，通过Cloud Pinyin可以设置云拼音。即可以将谷歌拼音，搜狗，百度，QQ输入法的内容合并进来。
       下图的设置，是当输入第二个词的时候，将云拼音的结果合并到第二个位置。
{% img /images/fcitx_config.png 'fcitx_config images' %}
{% img /images/fcitx_config_cloud_pinyin.png 'fcitx_config_cloud_pinyin images' %}

reference：

> [Fcitx](https://wiki.archlinux.org/index.php/Fcitx)
