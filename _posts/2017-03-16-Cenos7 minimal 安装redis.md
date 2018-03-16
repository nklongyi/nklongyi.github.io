---
layout:     post
title:      Centos7 安装redis
subtitle:    redis
date:       2018-03-16
author:     龙一
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - redis
---

> 用Centos7的miniamal镜像，安装redis


## 前置

安装基本的工具：

    yum install wget
    
安装环境：
    
    yum install gcc c++`

下载redis：

    wget http://download.redis.io/releases/redis-3.2.9.tar.gz

解压：`tar -zxvf redis-3.2.9.tar.gz`

    cd redis-3.2.9

    make

    make PREFIX=/usr/local/redis install

## 使用

    cd /usr/local/redis/

进入到安装文件redis目录下：
    
    cd ~
    cd redis-3.2.9
    cp redis.conf /usr/local/redis/


启动redis就是执行redis里的bin里的redis-server命令
    
     cd /usr/local/redis/
    ./bin/redis-server

![](http://blog.java1234.com/static/userImages/20170702/1498990002450032408.jpg)


这是一种直接在前端启动redis的方式。

还有一种是以守护进程（daemon）的方式运行，实际中的redis集群都是这种方式。

vi redis.conf

找到daemon，将daemonize no 改为yes

然后执行：

    [root@localhost redis]# ./bin/redis-server ./redis.conf 

此时通过进程命令查看：

    [root@localhost redis]# ps -aux | grep redis
    root   7647  0.1  0.7 136928  7528 ?Ssl  15:35   0:00 ./**bin/redis-server 127.0.0.1:6379**
    
    root   7651  0.0  0.0 112660   976 pts/0S+   15:36   0:00 grep --color=auto redis


关闭redis服务：

    [root@localhost redis]# ./bin/redis-cli shutdown

 	在查看：
    [root@localhost redis]# ps -aux | grep redis
     root   7655  0.0  0.0 112660   972 pts/0S+   15:38   0:00 grep --color=auto redis

## redis基本使用

redis启动，然后运行：

    [root@localhost redis]# ./bin/redis-cli 
    127.0.0.1:6379> 

    127.0.0.1:6379> set name nklongyi
    OK
    127.0.0.1:6379> get name
    "nklongyi"

	127.0.0.1:6379> set hello world
    OK
    127.0.0.1:6379> get hello
    "world"
    127.0.0.1:6379> keys *
    1) "hello"
    2) "name"
    127.0.0.1:6379> del hello
    (integer) 1
    127.0.0.1:6379> keys *
    1) "name"

>外部机器访问的时候，对centos7的防火墙要放开端口，或者直接关闭防火墙 `systemctl stop firewalld.service`







