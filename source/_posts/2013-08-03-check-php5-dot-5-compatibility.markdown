---
layout: post
title: "check php 5.5 compatibility"
keywords: "php compatibility codesniffer"
description: "how to check php syntactic compatibility"
date: 2013-08-03 18:16
comments: true
categories: [php]
tags: [php5.5, codesniffer]
---
github上有个项目可以检测php5.3，5.4的兼容性，如下：
[https://github.com/wimg/PHPCompatibility](https://github.com/wimg/PHPCompatibility)

如果最近你想把php升级到5.5，尝试下generators和coroutines，这个应该对你
有帮助。也可以参见博文：

[http://techblog.wimgodden.be/2012/03/04/php-5-4-compatibility-checks-using-php_codesniffer/](http://techblog.wimgodden.be/2012/03/04/php-5-4-compatibility-checks-using-php_codesniffer/)
<!-- more -->
第一次尝试时，可能因为php配置的问题（date.timezone），所以没有检测出任
何东西。

在php.ini中设置了

> date.timezone = Asia/Shanghai

之后，发现效果不错，只是显示的行号有问题，而且检测的速度不尽人意。

所以我在安装完php5.5后，写了个shell脚本，用php -l来检测。内容如下：

{% codeblock lang:bash %}
#!/bin/sh
SOURCE_ROOT='/path/php-code'
PHP_BIN='/usr/local/php-5.5/bin/php'
for file in $(find ${SOURCE_ROOT} -type f -iname '*.php'); do
    check_syntax=$(${PHP_BIN} -l $file | grep -v 'No syntax errors detected')
done
{% endcodeblock %}

使用方法：
> sh /path/check-php-compatibility.sh > check-result.txt
