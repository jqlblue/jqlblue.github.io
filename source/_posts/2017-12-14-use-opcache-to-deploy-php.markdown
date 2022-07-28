---
layout: post
title: "基于Opcache发布php项目"
description: "如何基于php的Opcache发布php代码"
keywords: "deploy php project use opcache，代码发布，Persistent secondary file-based cache for OPCache"
date: 2017-12-14 11:17
comments: true
categories: "php"
tags: [devops, php]
---
php的`Opcache`已经release好多年了，现在基本都是php的标配。最近看到了php创始人`Rasmus Lerdorf`的一篇[talk](http://talks.php.net/confoo16#/)，于是有了使用php的opcache发布代码的想法。
<!-- more -->

# Opcache

`Opcache`也叫`Zend OPcache`，其前身是Zend公司开发的闭源PHP优化加速组件`Optimizer+`。于2013年3月中旬改名为`Opcache`，并开源。

在2013年6月发布的php5.5.0版本中，整合了`Opcache`。

[pecl](https://pecl.php.net/package/ZendOpcache)上的`Opcache`扩展，于2015-01-12转正。


`OPcache`通过将 PHP 脚本预编译的字节码存储到共享内存(或者文件)中来提升 PHP 的性能， 存储预编译字节码的好处就是 省去了每次加载和解析 PHP 脚本的开销。不过`OPcache` 没有象 APC 那样的 user cache 功能。

{% img /images/opcache/php_lifecycle_opcache.png PHP script lifecycle %}


# Opcache使用

php5.5及以上版本已经整合了`Opcache`，只需要在编译php时添加

    --enable-opcache --enable-opcache-file

即可为php添加opcache扩展，然后修改`php.ini`，主动开启`Opcache`

    zend_extension="/path/opcache.so"
    [opcache]
    opcache.enable=1
    opcache.enable_cli=1


之后运行php程序时，会生成`Opcache`，下次运行时可以直接加载，免去了词法分析，语法分析，以及解析php脚本花费的时间。

`Opcache`默认存储在php进程的共享内存中，不过也可以存储到本地文件中。性能的差异大致如下：

{% img /images/opcache/secondary_file-based_cache.png Persistent secondary file-based cache for OPCache %}

看完这张图，突然有种脑洞大开的感觉。

发布php时，预先生成opcache-file，只发布opcache，是否可行呢？

# 使用Opcache发布php

开启`Opcache`后，可以使用`opcache_compile_file`生成php的opcache-file。涉及`php.ini`中的如下配置参数

    opcache.max_accelerated_files=20000
    opcache.file_cache=/data/php/opcache
    opcache.file_cache_only=0
    opcache.validate_timestamps=0
    opcache.save_comments=0

{% codeblock lang:php %}

function opcache_compile_files($dir) {
    foreach(new RecursiveIteratorIterator(new RecursiveDirectoryIterator($dir)) as $v) {
        if(!$v->isDir() && preg_match('%\.php$%', $v->getRealPath())) {
            $file = $v->getRealPath();
            opcache_compile_file($file);
            file_put_contents($file, '');
            echo $file . "\n";
        }
    }
}

opcache_compile_files('/path/project/foo');
{% endcodeblock %}

在命令行下执行上述脚本，即可遍历整个项目，生成对应php的opcache-file，同时会清空原php文件的内容。

如：

    ls /data/php/opcache/4d400aec8fadf667fabe41a87f30f7cc

    /data/php/opcache/4d400aec8fadf667fabe41a87f30f7cc/path/project/app/Utility.php.bin

```
    1. 生成opcache-file时，需要设置opcache.file_cache_only=0
    2. 需要保持原有项目的目录结构和文件 -- 内容可以清空，使opcache-file生效
```
