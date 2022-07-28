---
layout: post
title: "php 5.5 upgrade guide"
keywords: "php 5.5 upgrade guide"
description: "how to upgrade or install php to version php 5.5"
date: 2013-08-10 09:41
comments: true
categories: [php]
tags: [php5.5, devops]
---
前一阵子经常收到应用服务器的报警。登录服务器查看日志，netstat，strace
看不出问题（道行不够）。然后restart之后，一切又都正常。

面对打着补丁的5.2，我们决定升级到php5.5
<!-- more -->
### The reason ###
左右事物发展变化的，除了自身的能力和品质，有时跟环境的变迁，也有着莫大
的关系。

作为一个在开发一线的互联网从业人员。优化，重构，这可能是我们要经常提及
的两个话题。

通过监控，度量，找出系统中那20%的瓶颈问题，一般就能让系统的性能有很大
的提升。这种优化算作自身能力的提升。

一个在线上运行的系统，实际上还依赖与服务器，带宽等硬件环境；操作系统，
程序编译器或者解释器等软件环境。这些环境，也在更新优化。

在自身能力不变的情况化，如果这些环境得以更新优化，我们的系统也会跑地更
快。

就像nike，同样的人，穿着nike会有飞一般的感觉。

### Step ###
先引用鸟哥在 [thinkinlamp](http://www.thinkinlamp.com/)演讲内容：
{% img /images/php-5.5-prefermance.png 'php-5.5-prefermance 'images' %}

理论上，从php5.2升级到php5.5，性能最少能提升30%。

当然，升级有风险，操作需谨慎。

下面是升级步骤：

##### 1. 安装php5.5 #####
{% codeblock php-5.5.0-install.sh lang:bash %}
PHP_VERSION=5.5.0
wget -c http://www.php.net/get/php-${PHP_VERSION}.tar.gz/from/this/mirror
tar zxvf php-${PHP_VERSION}.tar.gz
cd php-${PHP_VERSION}
PHP_PREFIX=/usr/local/php-${PHP_VERSION}
./configure \
--prefix=${PHP_PREFIX} \
--with-config-file-path=${PHP_PREFIX}/etc \
--disable-debug \
--enable-inline-optimization \
--disable-all \
--enable-fpm \
--enable-libxml \
--enable-session \
--enable-xml \
--enable-hash \
--enable-mbstring \
--with-layout=GNU \
--enable-filter \
--with-pcre-regex \
--with-zlib \
--enable-json \
--enable-mysqlnd \
--enable-pdo \
--with-mysql=mysqlnd \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--enable-simplexml \
--enable-dom \
--enable-phar \
--enable-tokenizer \
--enable-posix \
--enable-xmlwriter \
--enable-xmlreader \
--with-curl \
--with-iconv \
--with-mcrypt \
--enable-ctype \
--enable-opcache
make
make install
{% endcodeblock %}

完整脚本可参见：
[在centos5上安装php5.5.0](https://gist.github.com/jqlblue/6198630)

备注：
    1. 安装php时，要指定安装目录(prefix)，以备出现问题时可以及时回滚
    2. 安装扩展时同样需要注意，不要覆盖其他版本的扩展
       1) 使用绝对路径执行phpize
       2) 在configure时指定php5.5目录下的php-config，--with-php-config
       3) 编译扩展需要的lib库时，也要指定版本和安装目录
##### 2. php代码语法兼容性检测 #####
php每次大版本的升级，都会废弃一些函数，这些函数在新版本的php中使用的话，
就会报致命错误（Fatal error）。通过下面的方法，可以检测这些场景。
{% codeblock check-php5.5-compatibility.sh lang:bash %}
#!/bin/sh
#use sh ./check-php-compatibility.sh /path/code
if [[ $1 != '' ]];then
   SOURCE_ROOT=$1
else
   SOURCE_ROOT='/path/code/src'
fi
PHP_BIN='/usr/local/php-5.5.0/bin/php'
for file in $(find ${SOURCE_ROOT} -type f -iname '*.php'); do
    ${PHP_BIN} -d error_reporting=E_ALL -l $file | grep -v 'No syntax errors detected'
done
{% endcodeblock %}
更多的信息，请参见 [static-analysis-for-php](http://codeascraft.com/2012/08/10/static-analysis-for-php/)

##### 3. 从服务器上收集请求情况 #####
{% codeblock lang:bash %}
cat access.log | awk -F "[ ?]" '{urls[$7]++} END {for(key in urls)print urls[key],"\t",key}' | sort -nr
{% endcodeblock %}

##### 4. 通过收集的请求url进行测试 #####
对于get方式的url请求，直接wget或者curl就好。

对于post方式的curl请求，需要用curl模拟post请求进行提交。同时需要在
webserver日志中记录post请求参数。

##### 5. 查看webserver的错误日志，进行改进 #####

### Improve ###
上线后通过在nginx的access log中开启记录upstream_time和request_time进行
统计，php-5.5.0环境下接口的响应速度较5.2.5环境提升了(0.00487559-0.00366027)/0.00487559=24.9%

    php-5.5.0
    total_request_time:196416 total_upstream_time:1122.52 total_request:306678 avg_requst_time:0.640464 avg_upstream_time:0.00366027

    php-5.2.5
    total_request_time:200400 total_upstream_time:1498.65 total_request:307378 avg_requst_time:0.651966 avg_upstream_time:0.00487559

统计脚本

    cat access.log | grep '/api/getInfo' | awk -F ')"' '{print $2}' | grep -v ' -' | awk 'BEGIN{_request=0;_upstream=0;_i=0}{_request+=$2;_upstream+=$3;_i+=1}END{print "total_request_time:"_request" total_upstream_time:"_upstream" total_request:"_i" avg_requst_time:"_request/_i" avg_upstream_time:"_upstream/_i}'

另外，做了php5.3.10和php.5.5.0的压测比较

    php5.3.10
    Speed=30441 pages/min, 1224275 bytes/sec.
    Requests: 30441 susceed, 0 failed.

    php5.5.0
    Speed=42514 pages/min, 1727526 bytes/sec.
    Requests: 42514 susceed, 0 failed.
    PHP 5.5 vs 5.3 improve 39% (708-507)/507=0.3964497041
