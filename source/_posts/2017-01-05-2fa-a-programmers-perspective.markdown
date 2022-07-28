---
layout: post
title: "深入理解双因子认证"
description: "深入理解双因子认证，使用google验证器搭建双因子认证服务"
keywords: "双因子认证 google验证器 2fa totp hotp Google Authenticator two-factor authentication"
date: 2017-01-05 12:03
comments: true
categories: "devops"
tags: "devops"
toc: true
---
去年年初，让ops在服务器上开启了基于google-authenticator的双因子认证。最近花了点时间进行深入了解，记录如下。
<!-- more -->

# 双因子认证的相关概念
双因子认证（Two-factor authentication，也叫2FA），是一种通过组合两种不同的验证方式进行用户身份验证的机制。Google在2011年3月份，宣布在线上使用双因子认证，MSN和Yahoo紧随其后。

双因子认证，除了需要验证用户名密码外，还要结合另外一种实物设备，如Rsa令牌，或者手机。

如果我们把传统的用户名密码验证称为单因子认证（1FA），那么对比双因子认证（2FA），他们的区别如下：

> 1FA - What you know (e.g. a password, a pin)

> 2FA - What you have (e.g. a phone, a hardware token)

> 3FA - What you are (e.g. your fingerprints, you retina)

双因子认证的产品大致可以分成两类：

* 可以产生token的硬件设备
* 智能手机的app

手机短信验证码，登录微信公众号时的扫码确认都可以称为双因子认证。

双因子认证，还会结合一个只有你有的硬件设备。只要这个专属的硬件设备不丢失（察觉这个设备丢失，比用户名密码泄露，会容易很多），就可以大大地提升账号的安全性。


# 双因子认证的实现

双因子认证的流程如下：
{% img /images/2fa/flow.png Two-factor authentication flow %}

认证过程中涉及的token，一般会使用一次性密码([One-time password](https://en.wikipedia.org/wiki/One-time_password))，相关实现有：

* HOTP: 基于次数的一次性密码（[HMAC-Based One-Time Password](https://tools.ietf.org/html/rfc4226)）
* TOTP: 基于时间的一次性密码（[Time-Based One-Time Password](https://tools.ietf.org/html/rfc6238)）

`HOTP`和`TOTP`的实现都基于[HMAC-SHA-1](https://tools.ietf.org/html/rfc2104)算法。

`HOTP`的生成算法如下

    HOTP(K,C) = Truncate(HMAC-SHA-1(K,C))

其中：

* `C`是一个8-byte的自增变量。对于客户端，每生成一次性密码，其值加1。对于服务端，每次成功认证客户端产生的一次性密码，其值加1。在`HOTP`生成（客户端）和验证（服务端）过程中，C的值必须同步。
* `K`是客户端和服务端使用的共享密钥，每个客户端的`K`应该都是唯一的。

生成步骤如下：

    Step 1: 使用HMAC-SHA-1算法，利用C和K，生成一个长度为20-byte的40个十六进制字符，即：HS = HMAC-SHA-1(K,C)
    Step 2: 根据前面产品的字符串`HS`，生成一个长度为4-byte的8个十六进制字符，即：Sbits = DT(HS)，DT是根据HS，动态产生Sbits的方法，后面的示例中会提到
    Step 3: 根据前面的Sbits，计算一个HOTP的值，一般为6位数字。

> 2 nibbles (2 hex characters) = 1-byte

`TOTP`可以当做是`HOTP`算法的一个变种，可以将`TOTP`的生成算法定义为：

    TOTP = HOTP(K, T)

`K`同`HOTP`算法中`K`的定义，是客户端和服务端使用的共享密钥，`T`是一个整数，定义如下：

    T = floor((Current Unix time - T0) / X)

其中：

* `T0`是起始的Unix Time，默认为0
* `X`是`T`增长的步长，默认为30

即`T`是以30为步长，当前的Unix Time距初始的Unix Time`T0`增长的数量。

如果`T0`=0，`X`=30，那么当此刻的Unix time是59时，`T`=1，当此刻的Unix time为60时，`T`=2。`TOTP`算法生成的一次性密码，就会每30s变更一次。

# 一次性密码的生成过程
本文以HMAC-SHA-1算法生成的字符串`HS`的值是`0215a7d8c15b492e21116482b6d34fc4e1a9f6ba`为例，介绍一次性密码的生成过程。

如果使用`TOTP`算法进行双因子认证，要让用户在30s内输入40个十六进制的字符，这是一件很难想象的事情。所以我们需要想个办法，将`HS`转换地更加便于输入，而又不失安全性。这就是前面提到的DT（Dynamic Truncation）的处理过程。

为了更清晰地展示生成过程，用下图表示`HS`：

{% img /images/2fa/hotp_step1.png Two-factor authentication step1 %}

前面的图中包含40个字符，每个字符都占4-bits（有16个可能的值0-15），被分成了20组单独的字符串。

我们先去找`HS`的低4位（最后一个字符），作为截取字符串的起始位置。在我们的例子里，最后一个字符是`a`：

{% img /images/2fa/hotp_step2.png Two-factor authentication step2 %}

将十六进制的字符`a`转成十进制数是`10`。

我们将第1组字符串的偏移量用`0`表示，以此类推，如下：

{% img /images/2fa/hotp_step3.png Two-factor authentication step3 %}

然后，从字符串`HS`的第`10`个偏移量开始，截取`4`组字符串（或者是接下来的31-bits）。

> 这样截取的最大偏移量是15+4=19，刚好没有越界

因此，我们通过DT（Dynamic Truncation）处理，将`HS`转换后得到的字符串是`6482b6d3`：

{% img /images/2fa/hotp_step4.png Two-factor authentication step4 %}

将十六进制的`6482b6d3`转成十进制数是`1686288083`。

因为我们需要一个6位的数字，所以和`1000000`进行取模运算：

    1686288083 modulo 1000000

最后的结果是：

    288083

# 使用google-authenticator，开启服务器双因子认证

首先，去你喜欢的android应用市场，或者apple的appStore去安装：“Google Authenticator（google身份验证器）”。

然后登录要开启双因子认证登录的服务器，进行下面的操作。

安装依赖

    yum -y install gcc gcc-c++ make wget pam-devel

安装Google Authenticator

    wget http://google-authenticator.googlecode.com/files/libpam-google-authenticator-1.0-source.tar.bz2
    tar jxvf libpam-google-authenticator-1.0-source.tar.bz2
    cd libpam-google-authenticator-1.0
    make
    sudo make install

配置SSH登录时调用google-authenticator模块

编辑文件`/etc/pam.d/sshd`，添加：

    auth       required     pam_google_authenticator.so

编辑文件`/etc/ssh/sshd_config`，在文件中查找`ChallengeResponseAuthentication`和`UsePAM`，修改为如下内容：

    ChallengeResponseAuthentication yes
    UsePAM yes

重启ssh

    sudo service ssh restart

下面是配置Google Authenticator的相关步骤。

如果要为用户`zhangsan`添加ssh登录时的双因子认证，执行如下命令：
```
    su - zhangsan
    google-authenticator
```

会出现一串问题，让你选`y`或者`n`。

    Do you want authentication tokens to be time-based (y/n) y
    https://www.google.com/chart?chs=200x200&chld=M|0&cht=qr&chl=otpauth://totp/zhangsan@ali%3Fsecret%3DWKHM6UVJNTPYSPTQ
    Your new secret key is: WKHM6UVJNTPYSPTQ
    Your verification code is 434260
    Your emergency scratch codes are:
    30287010
    70585905
    68748337
    15176712
    38041521

上面的这一步，会生成一个base32编码的共享密钥`WKHM6UVJNTPYSPTQ`，即前面的`K`，用于在客户端进行绑定（如果可以翻墙的话，实际上会看到一张二维码，使用Google Authenticator app扫码即可以完成绑定）。

共享密钥使用base32而非base64的原因如下：

* base32编码的字符串，包含了大写英文字母和数字2-7。不会因字体显示问题，把1，8，0和'I','B', 'O'混淆，更利于输入。
* base32编码的字符串，出现在url中时，可以不用进行url编码处理（encode），便于直接使用生成二维码的web服务。

同时，基于当前的Unix time，生成了一个动态验证码`434260`，可用于测试。并生成了5个应急备用验证码（上面的emergency scratch codes），可以在绑定设备丢失的情况下使用（每个应急码只能使用一次）。

剩下的问题，没有特殊癖好，可以都选`y`。

    Do you want me to update your "/home/zhangsan/.google_authenticator" file (y/n) y

    Do you want to disallow multiple uses of the same authentication
    token? This restricts you to one login about every 30s, but it increases
    your chances to notice or even prevent man-in-the-middle attacks (y/n) y

    By default, tokens are good for 30 seconds and in order to compensate for
    possible time-skew between the client and the server, we allow an extra
    token before and after the current time. If you experience problems with poor
    time synchronization, you can increase the window from its default
    size of 1:30min to about 4min. Do you want to do so (y/n) y

    If the computer that you are logging into isn't hardened against brute-force
    login attempts, you can enable rate-limiting for the authentication module.
    By default, this limits attackers to no more than 3 login attempts every 30s.
    Do you want to enable rate-limiting (y/n) y

之后，ssh登录服务器时，会看到类似这样的提示：

    verification code:

这时，打开手机上的google身份验证器App，输入对应的code，如下：

{% img /images/2fa/google-authenticator.png google 验证器 flow %}

reference：

[^1] https://pthree.org/2014/04/15/time-based-one-time-passwords-how-it-works/

[^2] https://garbagecollected.org/2014/09/14/how-google-authenticator-works/

[^3] https://www.blackmoreops.com/2014/06/26/securing-ssh-two-factor-authentication-using-google-authenticator/
