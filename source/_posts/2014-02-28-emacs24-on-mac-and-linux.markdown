---
layout: post
title: "在mac系统上使用emacs24打造web开发环境"
keywords: "emacs24.3, mac, web, develop environment"
description: "如何在mac系统上使用emacs 24 配置php，python的web开发环境"
date: 2014-02-28 14:16
comments: true
categories: [emacs]
tags: [linux, mac, emacs]
---
Emacs 是一个强大的、可扩展的文本编辑器。不同于vim，它是一个流行的无模式文本编辑器。尤其是当emacs24内置包管理elpa后，使用通过`prelude`，`goblin`等，轻松使用emacs打造一个顺手的diy的web开发环境。
<!-- more -->

### 安装Emacs24.3 ###

##### 安装Homebrew #####

`Homebrew`是mac系统上的包管理软件，是用`Ruby`语言编写的。我们可以使用它在终端安装系统没有自带的`Unix`相关工具。

*安装步骤*

    ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"
    cd /usr/local/Library && git stash && git clean -d -f


##### 通过编译源代码安装Emacs #####
在安装`Homebrew`时，会同时安装`gcc`和`autoconf`，所以我们可以直接下载源代码进行编译安装。通过如下地址可以下载到最新的emacs安装文件。
    http://www.gnu.org/software/emacs/

如果没有`wget`等工具，可以通过`brew`进行安装，如：
    brew install wget

*安装步骤*
    cd /somepath/
    wget http://mirror.bjtu.edu.cn/gnu/emacs/emacs-24.3.tar.gz
    tar zxvf emacs-24.3.tar.gz
    cd emacs-24.3
    ./autogen.sh
    ./configure --with-ns
    make install
    sudo ln -s /somepath/emacs-24.3/nextstep/Emacs.app /Applications/Emacs24.3.app

##### 通过Homebrew安装Emacs #####

`Homebrew`本身也是下载源代码进行编译安装，但是它可以帮我们简化这一过程。这就是技术的魅力 -- make live easier。

*安装步骤*
    brew install emacs --cocoa
    brew linkapps

顺利的话，最新版的emacs就安装在mac了。如果中途遇到问题，按照提示解决下就好。

有可能下载地址被墙，这时通过通过修改源代码的下载地址解决，方法如下：

    1. brew edit softname，如 brew edit emacs
    2. 修改其中的url，保存退出
如：

{% codeblock lang:ruby %}
require 'formula'

class Emacs < Formula
    homepage 'http://www.gnu.org/software/emacs/'
    #url 'http://ftpmirror.gnu.org/emacs/emacs-24.3.tar.gz'
    url 'http://mirror.bjtu.edu.cn/gnu/emacs/emacs-24.3.tar.gz'
{% endcodeblock %}

安装完成后可以在`应用程序`，或者`Launchpad`中启动emacs，它默认长这样：

{% img /images/emacs/startup.png 'emacs start up %}

### 配置Emacs ###

由于emacs24已经自带了包管理系统。只需几个简单的步骤，即可通过[Emacs Prelude](https://github.com/bbatsov/prelude)或者[Goblin Emacs](https://github.com/jqlblue/goblin-emacs)体验emacs的魅力。步骤如下：

    cd /somepath/
    git clone https://github.com/jqlblue/goblin-emacs
    ln -s /somepath/goblin-emacs ~/.emacs.d

启动emacs后，会自动下载需要的扩展，完成后即可体验。

{% img /images/emacs/goblin-startup.png 'goblin emacs start up %}

完成`jedi`，python自动完成的配置
    cd ~/.emacs.d/elpa/jedi*
    sudo pip install -r requirements.txt

或者指定pypi源
    sudo pip install -i http://pypi.douban.com/simple -r requirements.txt

### 补充说明 ###

* Goblin-emacs简介

goblin-emacs在prelude的基础上，对`PHP`，`Python`等`mode`进行了增强，并尽量保持原生的快捷键。相关功能介绍：

    flymake语法检测
    php-mode
    php基于字典的自动完成
    python基于jedi的自动完成
    org-mode
    doxymacs 生成文档注释
    slime－mode
    版本控制工具的集成

当使用emacs编辑`ruby`或者`lua`源码时，会自动下载并安装相关`mode`，相关映射在`core/goblin-packages.el`中进行配置。

* 交换`Control`键和`Caps-Lock`键

因为emacs上的很多快捷键默认都是以`Control`开始。操作久了小拇指会比较难受，将`Control`和`Caps-Lock`进行交换，可以解放要经常蜷缩的小拇指。
{% img /images/emacs/swap-control-capslock.png 'swap control caps-lock %}

* 某些汉字显示为方块

由于某些字体不支持斜体的中文汉字等，这是就会在emacs中出现方块。解决方法如下：

    M-x customize-face RET font-lock-comment-face
    修改其中的"slant"为"normal"

goblin－emace通过添加了如下设置解决：
    (set-fontset-font "fontset-default"
        'gb18030 '("Microsoft YaHei" . "unicode-bmp"))
    )

* 其他技巧

一些常用的技巧记录如下
    通过`C-h t`可以查看emacs自带的教程
    通过M-x describe-mode可以查看当前支持的mode和相关快捷键


reference：

[^1] http://earthwithsun.com/questions/631306/emacs-24-loading-a-package-installed-via-elpa

[^2] http://toumorokoshi.github.io/emacs-from-scratch-part-2-package-management.html

[^3] http://blog.yam.com/hn12303158/article/35207136

[^4] http://blog.chinaunix.net/uid-26354188-id-3195392.html
