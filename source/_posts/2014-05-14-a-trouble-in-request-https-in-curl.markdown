---
layout: post
title: "一个使用curl请求https资源的问题排查"
description: "排查在linux环境下，在某些服务器上，使用curl请求https资源发生证书验证失败的问题"
keywords: "curl https php https证书 ca-bundle.crt ldd"
date: 2014-05-14 14:50
comments: true
categories: [php]
tags: [php, linux]
toc: true
---
昨天临下班前，应客户端大牛的要求，开发了一个返回下载服务器ip列表的接口，用于在客户端指定host以解决用户下载时遭遇运营商dns劫持的问题。

开发时略微有少许忐忑，但测试时一切顺利，于是就轻松地回家了。
<!-- more -->

早上一上线代码，就收到了通过`sentry`发出的报警邮件。原以为是缓存没有及时更新的问题，立马手动进行更新。但还是没有通过接口获取到相关ip。随即回滚代码，重新上线。

# 排查过程 #

后来下线一台服务器进行调试时发现，在调用ops提供的接口获取ip列表时没有获取到返回数据，而相关接口是`https`的。

再跟踪请求资源的函数发现，相关函数没有对`https`请求做特殊处理。相关函数实现如下：

    public static function get($url, array $headers = array(), $timeout = 5)
    {
        $ch = curl_init();

        curl_setopt($ch, CURLOPT_URL, $url);

        if ($headers) {
            curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
        }

        curl_setopt($ch, CURLOPT_CONNECTTIMEOUT, $timeout);
        curl_setopt($ch, CURLOPT_TIMEOUT, $timeout);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

        $content = curl_exec($ch);
        $response = curl_getinfo($ch);

        curl_close($ch);

        if ($response['http_code'] == 200) {
            return $content;
        }

        return null;
    }

这或许就是昨天那少许忐忑的缘由。于是增加如下代码，测试通过后重新上线。

        if (substr($url, 0, 5) == 'https') {
            curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
            curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, false);
        }

# 进一步排查 #
线上的问题虽然暂时解决了，但是在问题解决之前，测试机上是正常的，这是为什么呢？

## 在命令行运行curl排查问题 ##

在命令行使用curl请求ops的接口，其中线上服务器的运行结果如下：

    $ curl 'https://x.x.x.x/get_ips'

    curl: (60) SSL certificate problem, verify that the CA cert is OK. Details:
    error:14090086:SSL routines:SSL3_GET_SERVER_CERTIFICATE:certificate verify failed
    More details here: http://curl.haxx.se/docs/sslcerts.html

测试机上可以正常获取到结果。

然后分别查看curl的版本和curl使用的动态连接库，都没有发现差异
    $ /usr/bin/curl -V
    $ type curl

    /usr/bin/curl
    $ ldd /usr/bin/curl

再查看上面的错误，发现可能是`https`证书的问题。于是添加`--verbose`参数，再次使用curl进行请求，以获取更多交互信息。

截取部分输出如下

    $ curl 'https://x.x.x.x/get_ips' --verbose

    * About to connect() to x.x.x.x port 80
    *   Trying x.x.x.x... connected
    * Connected to x.x.x.x (x.x.x.x) port 80
    * successfully set certificate verify locations:
    *   CAfile: /etc/pki/tls/certs/ca-bundle.crt
    CApath: none
    * SSLv2, Client hello (1):
    SSLv3, TLS handshake, Server hello (2):
    SSLv3, TLS handshake, CERT (11):
    SSLv3, TLS alert, Server hello (2):
    SSL certificate problem, verify that the CA cert is OK. Details:
    error:14090086:SSL routines:SSL3_GET_SERVER_CERTIFICATE:certificate verify failed
    * Closing connection #0
    curl: (60) SSL certificate problem, verify that the CA cert is OK. Details:
    error:14090086:SSL routines:SSL3_GET_SERVER_CERTIFICATE:certificate verify failed
    More details here: http://curl.haxx.se/docs/sslcerts.html

可见使用的证书的是`/etc/pki/tls/certs/ca-bundle.crt`。

使用测试机上的证书替换线上服务器的证书后，问题解决。

> 如果没有可用的证书，可以使用如下方法：

    $ curl http://curl.haxx.se/ca/cacert.pem -o /etc/pki/tls/certs/ca-bundle.crt

# 问题总结 #
在请求https的资源时，遇到证书不匹配的问题，一般的工具都有不进行https证书验证的选项，比如：

    $ wget 'https://x.x.x.x/get_ips' --no-check-certificate
    $ curl 'https://x.x.x.x/get_ips' -k

当然，也可以在请求时指定证书，或者对使用的https ca证书进行更新。

reference:
[^1] http://curl.haxx.se/docs/sslcerts.html
