---
layout: post
title: "在生产环境部署sentry进行错误收集和提醒"
keywords: "deploy sentry in product with event logging and aggregation php"
description: "如何在线上生产环境部署sentry哨兵，对运行时错误进行收集，并通过邮件提醒等方式，及时帮助我们发现线上问题。改善服务质量"
date: 2013-12-18 11:10
comments: true
categories: [devops]
tags: [devops, sentry, monitor]
---
Sentry正如其名，是一个实时的日志聚合平台，可以通过捕获程序事件（`Error`，`Exception`），或者主动上报的方式将错误信息等进行收集汇总和提醒，以帮助我们及时发现项目中的问题。
<!-- more -->
Sentry Server端是使用python语言开发的，目前有如下平台的客户端sdk：

`Python`，`PHP`，`Ruby`，`Javascript`，`Java`，`Nodejs`，`IOS`

项目地址：https://github.com/getsentry/sentry

本文以收集`PHP`错误为例。
### 安装步骤 ###
Sentry的文档清晰且完善，包括`安装`，`配置`，`调优`以及`客户端调用`，正式使用之前，建议看看，以加深理解。地址：http://sentry.readthedocs.org/en/latest/

#### python环境安装 ####
Sentry需要python2.5以上，本文以`python2.7.3`为例，使用`virtualenv`进行环境隔离，使用`pip`安装需要的包
{% codeblock python2.7.3-install.sh lang:bash %}
cd ~
yum install -y bzip2-devel.x86_64
yum install -y sqlite-devel.x86_64
yum install -y readline-devel.x86_64
wget http://www.python.org/ftp/python/2.7.3/Python-2.7.3.tar.bz2
tar jxvf Python-2.7.3.tar.bz2
cd Python-2.7.3
./configure --prefix=/usr/local/python2.7.3
make
sudo make install

wget https://pypi.python.org/packages/source/d/distribute/distribute-0.6.49.tar.gz --no-check-certificate
tar zxvf distribute-0.6.49.tar.gz
cd distribute-0.6.49
sudo /usr/local/python2.7.3/bin/python setup.py install
sudo /usr/local/python2.7.3/bin/easy_install virtualenv
sudo /usr/local/python2.7.3/bin/easy_install -i http://e.pypi.python.org/simple virtualenvwrapper
{% endcodeblock %}
至此，就完成了python2.7.3和pip，以及virtualenv的安装，使用如下命令进行测试

    /usr/local/python2.7.3/bin/python

#### 安装Sentry server ####
初始化安装目录

    mkdir -p /data/server/python-envs

添加相关环境变量

    vim ~/.bashrc

添加：

    export WORKON_HOME=/data/server/python-envs
    export VIRTUALENVWRAPPER_PYTHON=/usr/local/python2.7.3/bin/python
    export VIRTUALENVWRAPPER_VIRTUALENV=/usr/local/python2.7.3/bin/virtualenv
    source /usr/local/python2.7.3/bin/virtualenvwrapper.sh

使环境变量生效

    source ~/.bashrc

安装Sentry server

    mkvirtualenv sentry
    pip install sentry
    pip install sentry[mysql]
    pip install sentry[mysql] --upgrade

修改`~/.bashrc`，添加如下代码，以便登录后自动切换到相关python环境

    source /data/server/python-envs/sentry/bin/activate

### 快速配置 ###
或许你还没有做好决定，只是想尽快体验下Sentry，在完成上面的安装之后，通过下面三个步骤即可满足你的愿望：

1 初始化配置

    sentry init ~/sentry.conf.py
2 修改配置

修改初始配置中的如下两项就行

`SENTRY_WEB_HOST`，`SENTRY_URL_PREFIX`，如：

    SENTRY_URL_PREFIX = 'http://10.16.15.1:9000'
    SENTRY_WEB_HOST = '10.16.15.1'
3 创建超级管理员帐号，启动server

    sentry --config=~/sentry.conf.py upgrade
    sentry --config=~/sentry.conf.py createsuperuser
    sentry --config=~/sentry.conf.py start

然后就可以通过url http://server_host:port ，使用创建的帐号登录系统后台，进行项目，帐号等管理，和已收集日志的查看等等

### 配置在生产环境中使用 ###
#### Sentry server ####
*我们在生产环境下的使用状况：*

* 使用`mysql`作为后端数据存储

* 使用`celery`任务队列（`broker`使用`redis`），处理数据入库，发送邮件提醒等工作

* 同时，使用`redis`作为`Update Buffers`，用于将频繁出现的相同事件合并，这在高并发时会相当有用

* 使用`memcache`作为前端`Cache`，管理后台通过轮询的访问获取是否有新的事件提醒，使用`memcache`，可以减轻直接查询数据库的压力

* 使用`Udp`协议发送并接收相关事件

* 使用`Nginx`反向代理前端http请求，并使用`HttpLimitReqModule`限制请求的发送速率

* 使用`supervisor`管理`celery`和`sentry`server

*相关安装步骤：*

    pip install redis hiredis nydus
    pip install redis hiredis nydus --upgrade
    pip install python-memcached
    pip install gevent
    pip install eventlet
    pip install supervisor

*初始化配置*

    mkdir -p /data/server/sentry/etc
    sentry init /data/server/sentry/etc/sentry.conf.py

*创建超级管理员帐号*

    sentry --config=/data/server/sentry/etc/sentry.conf.py upgrade
    sentry --config=/data/server/sentry/etc/sentry.conf.py createsuperuser

*初始化supervisor配置*

    echo_supervisord_conf > /data/server/sentry/etc/supervisord.conf

*配置Sentry*

示例配置请参见 https://gist.github.com/jqlblue/8018185

修改`/data/server/sentry/etc/supervisord.conf`，添加：

    [program:web]
    command=/data/server/python-envs/sentry/bin/sentry --config=/data/server/sentry/etc/sentry.conf.py start
    process_name=%(program_name)s_%(process_num)02d
    numprocs=3
    numprocs_start=0
    startsecs=5
    startretries=3
    stopsignal=QUIT
    stopwaitsecs=10
    stopasgroup=true
    killasgroup=true
    environment=SENTRY_CONF="/data/server/sentry/etc/sentry.conf.py"
    directory=/data/server/python-envs/sentry/

    [program:sentry_udp]
    command=/data/server/python-envs/sentry/bin/sentry --config=/data/server/sentry/etc/sentry.conf.py start udp
    process_name=sentry_udp_server
    numprocs=1
    numprocs_start=0
    startsecs=5
    startretries=3
    stopsignal=QUIT
    stopwaitsecs=10
    stopasgroup=true
    killasgroup=true
    environment=SENTRY_CONF="/data/server/sentry/etc/sentry.conf.py"
    directory=/data/server/python-envs/sentry/

    [program:celeryd]
    command=/data/server/python-envs/sentry/bin/sentry celery worker -c 6 -P processes -l WARNING -n worker-%(process_num)02d.worker
    process_name=%(program_name)s_%(process_num)02d
    numprocs=1
    numprocs_start=0
    startsecs=1
    startretries=3
    stopsignal=TERM
    stopwaitsecs=10
    stopasgroup=false
    killasgroup=true
    environment=SENTRY_CONF="/data/server/sentry/etc/sentry.conf.py"
    directory=/data/server/python-envs/sentry/

*管理Sentry server*

* 启动superviord

执行如下命令，同时，`celery`，`sentry web`，`sentry udp server`也将随之启动

    supervisord -c /data/server/sentry/etc/supervisord.conf

* 停止sentry相关server

执行如下命令

    supervisorctl -c /data/server/sentry/etc/supervisord.conf stop all

* 停止superviord

执行如下命令，同时，已启动的`centry`相关server也将停止

    supervisorctl -c /data/server/sentry/etc/supervisord.conf stop all

`supervisor`更多使用方法请参见 http://supervisord.org/

`nginx`配置请参见 https://gist.github.com/jqlblue/8019629

#### Sentry client ####

可以通过在程序中`registerExceptionHandler`和`registerErrorHandler`将相关信息即时发送至server端。

相关sdk项目地址 https://github.com/getsentry/raven-php

实例化`Raven_Client`时使用的`DSN`中的`public:secret`可以在使用管理员登录后台后，在`项目`-`设置`下面查看到。示例地址：http://sentry_host/team_name/project_name/docs/php/

我们采用通过增量读取php error log，使用crontab将错误信息上报。

基于sentry php sdk修改之后的代码地址：https://gist.github.com/jqlblue/8019312

安装依赖

    yum install -y logcheck.noarch

`logcheck`中的`logtail2`用于增量读取日志，`flock`用于防止定时任务堆积。

> 另外，需要安装php的sockets扩展

添加定时任务
    * * * * * /usr/bin/flock -xn /tmp/sentry_client.lock /opt/php-5.5.4/bin/php /path/client.php --project=project_name 2>&1 > /dev/null
