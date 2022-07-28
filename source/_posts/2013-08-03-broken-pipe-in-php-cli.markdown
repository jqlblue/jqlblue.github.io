---
layout: post
title: "broken pipe in php cli"
keywords: "php cli broken pipe"
description: "how to fix broken pipe error when run php in cli"
date: 2013-08-03 20:08
comments: true
categories: [php]
tags: [php5.5, pcntl, broken pipe]
---
下周打算把服务器上的php升级到5.5，综合老大的建议，计划按这个步骤进行：

-------------------------------------------------------------------------------

* 下线一台服务器，另起目录安装php5.5

安装过程与php5.3差不多，要开启zend opcache的话，需要在
configure时--enable-opcache。在php.ini中配置时，需要以zend_extension的
形式加载。
<!-- more -->
* 从服务器访问日志中统计最近有请求的接口，按请求次数从大大小排序
{% codeblock lang:bash %}
cat access_log | awk '{print $7}' | awk -F "?" '{print $1}' | sort | uniq -c | sort -nr
{% endcodeblock %}
* 通过已统计的接口列表（通过第二步产生），从访问日志中查询相关请求地
   址（包括相关参数）
{% codeblock lang:bash %}
access_log | grep  '/api/test' | head -n 1
{% endcodeblock %}

* 绑定hosts，根据请求地址列表进行访问，观察web server和php的相关日志

-------------------------------------------------------------------------------
进行到第三步，就卡住了。：（

“cat access_log | grep  '/api/test' | head -n 1”这条命令在shell下执行
没有问题，但是如果用php的shell_exec运行，就会出现 “grep: writing
output: Broken pipe”。

一顿google之后，遇到这篇文章Python中的SIGPIPE信号。对作者的示例代码做
了下加工后，发现一切正常了，修改后的python代码如下
{% codeblock lang:python %}
import subprocess
import signal
def reset_sigpipe():
    signal.signal(signal.SIGPIPE, signal.SIG_DFL)

output = subprocess.Popen("cat access_log | grep  '/api/test' | head -n 1", shell=True, preexec_fn=reset_sigpipe, stdout=subprocess.PIPE)
print output.stdout.read()
{% endcodeblock %}

但是我是要用php来做这件事，下面是php相关代码
{% codeblock lang:php %}
<?php
$source = './source_url.txt';
$dest = './dest_url.txt';
$res = file($source);
pcntl_signal(SIGPIPE, SIG_DFL);
file_put_contents($dest, '');
foreach ($res as $row) {
    $row = trim($row);
    if (strpos($row, 'http://') !== false) {
        continue;
    }
    $command = "cat access_log | grep  '{$row}' | head -n 1 | awk '{print \$6\" \"\$7}' | awk -F '\"' '{print $2}' | awk '{print $1\" http://test.api.com\"$2}'";
    $check = system($command);
    file_put_contents($dest, $check . PHP_EOL, FILE_APPEND);
}
{% endcodeblock %}

和python中的signal.signal(signal.SIGPIPE, signal.SIG_DFL)一样，关键是
这句:
    pcntl_signal(SIGPIPE, SIG_DFL)
当php进程接收到SIGPIPE信号时，重置
为系统默认处理方式，即接收子进程的返回值。

**问题解决了，但是原因呢？**

我们再看看这行命令
    cat access_log | grep  '/api/test' | head -n 1

head命令在取得一行后立即退出（exit），此时pipe的读端就没了，但grep还会
继续往pipe写，此时pipe就会发送SIGPIPE信号，默认动作是终止程序。在shell
下执行时，grep收到SIGPIPE信号就退出了，所以运行没有问题。

但在通过php的shell_exec或者system运行为何就有问题。应该是php忽略了
SIGPIPE信号，所以grep会继续向broke pipe（读端关闭）写入，于是就出现了
    grep: writing output: Broken pipe

在天朝，有图也不一定是真相。所以应该是，也不一定是。废话少说，我们直接
上代码：
    $grep -r 'SIGPIPE' ./
    ./sapi/cli/php_cli.c:   signal(SIGPIPE, SIG_IGN); /* ignore SIGPIPE in standalone mode so
所以php以cli的形式运行时，会忽略SIGPIPE信号。

reference：

> [https://blogs.oracle.com/opal/entry/using_php_5_5_s](https://blogs.oracle.com/opal/entry/using_php_5_5_s)
