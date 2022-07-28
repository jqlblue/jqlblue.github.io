---
layout: post
title: "那个套模版的，放开你的F5"
description: "使用livereload浏览器扩展，提升web前端开发工程师的开发效率，减少修改代码后需要重新刷新页面的工序"
keywords: "模版工程师，套模版，livereload，释放F5，python"
date: 2015-02-28 18:29
comments: true
categories: [collect]
tags: [collect, php, python]
---
老江说过：“科学技术是第一生产力”。技术的魅力在于通过改善相关流程或者提供相关工具，对人们的生活进行改善，make live esaier。


<!-- more -->

*对于自喻为模版工程师的同行们，套模版的流程大抵是：*

    写代码，保存

    打开浏览器，按F5刷新页面，检查相关前端效果


我记得[轩脉刃](http://weibo.com/yjf10)曾经写过一个统计鼠标按键的小工具。如果对模版工程师工作时键盘的按键进行统计，那么F5的使用率肯定不容忽视。

倘若能在代码保存后就自动刷新浏览器，那不仅能解放模版工程师的F5按键，也能提升他们的开发效率。突然感觉非常美妙。

我记得有人说过，这个世界上不缺乏原创的idea，缺的只是一双能发现它的眼睛。


正如`livereload`所说的－“The Web Developer Wonderland”。

使用`livereload`，*通过如下几个步骤*，就可以做到当我们保存代码后，自动刷新浏览器中相关页面内容。

> 安装livereload浏览器扩展

相关浏览器扩展的下载地址如下：

[browser extensions](http://feedback.livereload.com/knowledgebase/articles/86242-how-do-i-install-and-use-the-browser-extensions)


> 安装livereload server端

安装python环境，然后在终端执行


    pip install livereload

或者

    easy_install livereload

> 启动livereload server端

假设我的代码目录在`/home/galendy/code/demo`，在终端执行

    livereload /home/galendy/code/demo

> 点击浏览器扩展

`livereload`的基本原理是：

    livereload server端会启动本地的socket服务（默认开放本地的35729端口），当监听的目录下的文件内容有变化时，向该socket写入数据

    livereload浏览器扩展会连接本地的35729端口，当有新消息到来时，会在浏览器中插入一段js代码，刷新当前页面

实际上，前端工程师还会使用`livereload`完成css，js等文件的合并和压缩。想要了解更多，请参考：

[livereload](http://livereload.com/)

[python livereload](http://livereload.readthedocs.org/en/latest/)
