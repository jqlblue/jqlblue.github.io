---
layout: post
title: "如何静态编译php"
description: "php静态编译，如何在linix操作系统上，通过静态编译的方式，编译一个可以随意拷贝的php可执行文件。"
keywords: "linux php complie statically php 静态编译"
date: 2014-04-09 18:13
comments: true
categories: [php]
tags: [php, linux, devops]
styles: [data-table]
toc: true

---
有些时候，我们写了一个php脚本，但是对方的服务器上没有php环境。


这时，我们可以通过静态方式编译php，并将相关扩展一起打包进php可执行文件，然后在运行脚本时指定php binary。
<!-- more -->
安装步骤如下：

# 准备源文件 #

```
wget -c http://www.php.net/get/php-5.5.11.tar.gz/from/this/mirror
tar zxvf php-5.5.11.tar.gz
wget http://pecl.php.net/get/redis-2.2.5.tgz
tar xvf redis-2.2.5.tgz
mv redis-2.2.5 php-5.5.11/ext/redis
```

# 配置 #

## 重新生成configure ##

```
cd php-5.5.11
rm -f ./configure
./buildconf --force
```

## configure ##

```
./configure LDFLAGS=-static \
--prefix=/usr/local/php5-static \
--disable-all \
--enable-shared=no \
--enable-static=yes \
--enable-inline-optimization \
--enable-hash \
--enable-mbstring \
--with-layout=GNU \
--enable-filter \
--with-pcre-regex \
--with-zlib \
--enable-json \
--enable-ctype \
--disable-redis-session \
--enable-redis
```

## 修改Makefile ##

将
```
BUILD_CLI = $(LIBTOOL) --mode=link $(CC) -export-dynamic $(CFLAGS_CLEAN) $(EXTRA_CFLAGS) $(EXTRA_LDFLAGS_PROGRAM) $(LDFLAGS) $(PHP_RPATHS) $(PHP_GLOBAL_OBJS) $(PHP_BINARY_OBJS) $(PHP_CLI_OBJS) $(EXTRA_LIBS) $(ZEND_EXTRA_LIBS) -o $(SAPI_CLI_PATH)
BUILD_CGI = $(LIBTOOL) --mode=link $(CC) -export-dynamic $(CFLAGS_CLEAN) $(EXTRA_CFLAGS) $(EXTRA_LDFLAGS_PROGRAM) $(LDFLAGS) $(PHP_RPATHS) $(PHP_GLOBAL_OBJS) $(PHP_BINARY_OBJS) $(PHP_CGI_OBJS) $(EXTRA_LIBS) $(ZEND_EXTRA_LIBS) -o $(SAPI_CGI_PATH)
```
替换成
```
BUILD_CLI = $(LIBTOOL) --mode=link $(CC) $(CFLAGS_CLEAN) $(EXTRA_CFLAGS) $(EXTRA_LDFLAGS_PROGRAM) $(LDFLAGS) $(PHP_RPATHS) $(PHP_GLOBAL_OBJS) $(PHP_BINARY_OBJS) $(PHP_CLI_OBJS) $(EXTRA_LIBS) $(ZEND_EXTRA_LIBS) -all-static -o $(SAPI_CLI_PATH)
BUILD_CGI = $(LIBTOOL) --mode=link $(CC) $(CFLAGS_CLEAN) $(EXTRA_CFLAGS) $(EXTRA_LDFLAGS_PROGRAM) $(LDFLAGS) $(PHP_RPATHS) $(PHP_GLOBAL_OBJS) $(PHP_BINARY_OBJS) $(PHP_CGI_OBJS) $(EXTRA_LIBS) $(ZEND_EXTRA_LIBS) -all-static -o $(SAPI_CGI_PATH)
```
即：

在`BUILD_CLI`和`BUILD_CGI`对应的行中移除`-export-dynamic`，在`-o $(SAPI_CGI_PATH)`和`-o $(SAPI_CLI_PATH)`之前，添加`-all-static`

# 安装 #

```
make LDFLAGS=-ldl
sudo make install
```

# 检查 #

在命令行执行

    $ file /usr/local/php5-static/bin/php
    /usr/local/php5-static/bin/php: ELF 64-bit LSB executable, AMD x86-64, version 1 (SYSV), for GNU/Linux 2.6.9, statically linked, for GNU/Linux 2.6.9, not stripped

    $ /usr/local/php5-static/bin/php -m
    [PHP Modules]
    Core
    ctype
    date
    ereg
    filter
    hash
    json
    mbstring
    pcre
    redis
    Reflection
    SPL
    standard
    zlib

    [Zend Modules]

因为可执行文件中包含了调试信息，所以体积较大

    $ ll -h /usr/local/php5-static/bin/php
    -rwxr-xr-x 1 root root 18M 04-09 18:11 /usr/local/php5-static/bin/php

可以通过`strip`命令移除调试信息

    $ sudo strip /usr/local/php5-static/bin/php
    $ ll -h /usr/local/php5-static/bin/php
    -rwxr-xr-x 1 root root 6.1M 04-09 18:11 /usr/local/php5-static/bin/php

原始文件大小| 去除符号表后大小
:--------------:|:---------------:
`18M`      | **6.1M**

** reference :**

[^1] http://www.php.net/manual/zh/install.pecl.static.php

[^2] http://d.hatena.ne.jp/shimooka/comment/20110216/1297827454

[^3] http://www.gnu.org/software/libtool/manual/html_node/Link-mode.html

[^4] http://markmail.org/message/cpoenglavs4vwv32
