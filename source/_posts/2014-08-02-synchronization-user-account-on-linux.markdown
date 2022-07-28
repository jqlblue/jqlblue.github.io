---
layout: post
title: "在linux服务器之间同步用户账号"
description: "如何在多台linux服务器之间同步用户账号，linux操作系统用户登录过程解析"
keywords: "linux操作系统 用户登录验证 用户账号同步 /etc/passwd /etc/group /etc/shadow"
date: 2014-08-02 17:26
comments: true
categories: [linux, devops]
tags: [linux]
---
最近负责运帷的同事离职了，原先由运帷可以一手搞定的事情，分摊到了几个研发同事的身上。但是多人公用一个账号，实在感觉不爽。
<!-- more -->
由于公司没有几台服务器上，所以可以逐一登录服务器创建新账号。但是对于一个码农而言，这不科学，它违背了`DRY`原则。

当然，也可以配置一个ldap服务器，修改linux用户登录使用ldap验证。但这让我有一种从火窟跳到冰窖的感觉。先不说是否能搞定配置的事情，引入的这个ldap，又会变成另外一个坑。

昨天听一个同事时，我们来上班，要对得起自己的良心。所以我不能让上班时间在纠结中度过，用土方法解决问题先。

## 同步步骤 ##

因为目前有一台服务器是登录的跳板机，所以只需要在跳板机上创建好新账号，然后把用户账号同步到其他机器上就好。
> 如果没有跳板机，也可以随便选一台服务器（A），在A服务器上创建账号，并同步到其他机器上。

* 在跳板机上创建用户账号

* 在要同步的服务器上创建账号，并将该用户在跳板机上如下文件中对于的条目追加到要同步到机器上

`/etc/passwd`， `/etc/group`, `/etc/shadow`

以跳板机ip：`192.168.1.1`，要同步的服务器：`192.168.1.8`，新增用户名：`jqlblue`为例，登录跳板机执行：

    $ useradd jqlblue
    $ ssh -l root -p 22 192.168.1.8 "useradd jqlblue"
    $ grep jqlblue: /etc/group | xargs -I {} ssh -l root -p 22 192.168.1.8 "echo {} >> /etc/group"
    $ grep jqlblue: /etc/passwd | xargs -I {} ssh -l root -p 22 192.168.1.8 "echo {} >> /etc/passwd"
    $ grep jqlblue: /etc/shadow | xargs -I {} ssh -l root -p 22 192.168.1.8 "echo {} >> /etc/shadow"

上述操作，编写成脚本即可。当需要新增或者修改用户时，只需在跳板机上进行操作，同步问题，由脚本来完成。

*上述脚本要在生产环境使用，需要注意如下问题：*
    1 新增用户时，uid或者gid重复的问题
    2 修改用户密码或者组信息后，产生多条记录的问题
