---
layout: post
title: "在php5.5中使用pdo和mysql_escape_string的一个坑"
keywords: "php5.5, pdo, mysql_escape_string"
description: "php 5.5 环境下，在pdo扩展下使用在mysql_escape_string函数时遇到的一个坑"
date: 2013-11-16 18:04
comments: true
categories: [php]
tags: [php5.5, mysql]
---
最近在项目中使用了鸟哥的yar扩展，但是在php5.2.10环境中没有安装成功，所以将php升级到了5.5。
<!-- more -->
### 升级步骤 ###
* 安装php5.5.0
* 检测代码兼容性
* 从线上服务器日志中收集最热的一百条访问日志
* 下线一台服务器，启动php5.5环境，根据日志中最热的请求进行重放
* 检测服务器日志进行改进

升级过程很顺利，但是上线后在日志中发现如下信息：
    mysql_escape_string(): This function is deprecated; use mysql_real_escape_string() instead
于是顺手修复上线。鉴于最近没怎么写代码，手有点生，先发布到了测试环境测试没问题才上线。

就在去茶水间接了一杯水的当儿，运营反馈说线上页面显示异常。于是马上回滚代码。

问题代码如下：
    $db = getDb();
    $a = mysql_real_escape_string($keyword);
    $sql = 'select info from table where keyword = ' . $a;
    $res = $db->getRow($sql);

通过调试，发现：
    $keyword在mysql_real_escape_string处理之后，变成了false，所以在进行后面的查询时获取不到相应结果，页面就异常了。

再进一步调试发现：
    getDb使用的是pdo。
    $dbh = new PDO('mysql:host=xxx;port=xxx;dbname=xxx', 'xxx', 'xxx');

于是原因浮出水面
    1 在使用string mysql_real_escape_string时没有指定link_identifier
    2 所以会去找使用mysql_connect()打开的最后一个连接
    3 使用的是pdo，没有找到相关连接。于是尝试使用不带任何参数的mysql_connect()去建立一个连接来使用
    4 本机没有mysql server，自然建立连接失败。于是发生错误，并产生警告信息（E_WARNING）
    5 服务器上设置的错误报告等级已经屏蔽了E_WARNING，所以也没有监控到相关错误
