---
layout:     post
title:      Centos7 安装RabbitMQ
subtitle:   spring boot使用rabbiitMQ
date:       2018-03-20
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - 消息中间件
    
---

## 简介

本文要介绍的RabbitMQ就是以AMQP协议实现的一种中间件产品，它可以支持多种操作系统，多种编程语言，几乎可以覆盖所有主流的企业级技术平台。

AMQP是Advanced Message Queuing Protocol的简称，它是一个面向消息中间件的开放式标准应用层协议。AMQP定义了这些特性：

- 消息方向 
- 消息队列
- 消息路由（包括：点到点和发布-订阅模式）
- 可靠性
- 安全性


## 安装rabitmq


进入官方网站：https://www.rabbitmq.com/download.html

### **第一步需要安装Erlang：**

虚拟机：centos7 minimal

参考：https://github.com/rabbitmq/erlang-rpm

新增/etc/yum.repos.d/rabbitmq-erlang.rep，内容如下：
  
To use Erlang 20.x on CentOS 7:

    # In /etc/yum.repos.d/rabbitmq-erlang.repo
    [rabbitmq-erlang]
    name=rabbitmq-erlang
    baseurl=https://dl.bintray.com/rabbitmq/rpm/erlang/20/el/7
    gpgcheck=1
    gpgkey=https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc
    repo_gpgcheck=0
    enabled=1
    
 
然后yum install erlang

安装完成以后：

    [root@localhost ~]# erl -version
    Erlang (SMP,ASYNC_THREADS,HIPE) (BEAM) emulator version 9.3

### **第二步 install rabbitMQ**

这一步我遇到点问题，按照官方的文档做法我失败了，参考了必应链接：https://teczuz.com/install-rabbitmq-server-centos-7/

和官方网站https://www.rabbitmq.com/install-rpm.html

A>wget https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.4/rabbitmq-server-3.7.4-1.el7.noarch.rpm

其中https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.4/rabbitmq-server-3.7.4-1.el7.noarch.rpm为官方网站上的list。

B>rpm --import https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc

C>yum install rabbitmq-server-3.7.4-1.el7.noarch.rpm


添加开机启动：

chkconfig rabbitmq-server on 

或者systemctl enable rabbitmq-server.service（centos7推荐的做法）

### **第三步 启动rabbitMQ**

[root@localhost ~]# systemctl start rabbitmq-server

查看一下执行状态：

    [root@localhost ~]# systemctl status rabbitmq-server
    ● rabbitmq-server.service - RabbitMQ broker
       Loaded: loaded (/usr/lib/systemd/system/rabbitmq-server.service; enabled; vendor preset: disabled)
       Active: active (running) since Fri 2018-03-23 10:33:16 CST; 13s ago
     Main PID: 1989 (beam.smp)
       Status: "Initialized"
       CGroup: /system.slice/rabbitmq-server.service
       ├─1989 /usr/lib64/erlang/erts-9.3/bin/beam.smp -W w -A 64 -P 1048576 -t 5000000 -stbt db -zdbbl 1280000 -K true -- -root /usr/lib64/erlang -progname...
       ├─2132 /usr/lib64/erlang/erts-9.3/bin/epmd -daemon
       ├─2265 erl_child_setup 1024
       ├─2285 inet_gethost 4
       └─2286 inet_gethost 4
    
    Mar 23 10:33:14 localhost.localdomain rabbitmq-server[1989]: ##  ##
    Mar 23 10:33:14 localhost.localdomain rabbitmq-server[1989]: ##  ##  RabbitMQ 3.7.4. Copyright (C) 2007-2018 Pivotal Software, Inc.
    Mar 23 10:33:14 localhost.localdomain rabbitmq-server[1989]: ##########  Licensed under the MPL.  See http://www.rabbitmq.com/
    Mar 23 10:33:14 localhost.localdomain rabbitmq-server[1989]: ######  ##
    Mar 23 10:33:14 localhost.localdomain rabbitmq-server[1989]: ##########  Logs: /var/log/rabbitmq/rabbit@localhost.log
    Mar 23 10:33:14 localhost.localdomain rabbitmq-server[1989]: /var/log/rabbitmq/rabbit@localhost_upgrade.log
    Mar 23 10:33:14 localhost.localdomain rabbitmq-server[1989]: Starting broker...
    Mar 23 10:33:16 localhost.localdomain rabbitmq-server[1989]: systemd unit for activation check: "rabbitmq-server.service"
    Mar 23 10:33:16 localhost.localdomain systemd[1]: Started RabbitMQ broker.
    Mar 23 10:33:16 localhost.localdomain rabbitmq-server[1989]: completed with 0 plugins.

关闭防火墙：`systemctl stop firewalld.service`

开启web插件：`rabbitmq-plugins enable rabbitmq_management`

    [root@localhost ~]# rabbitmq-plugins enable rabbitmq_management
    The following plugins have been configured:
      rabbitmq_management
      rabbitmq_management_agent
      rabbitmq_web_dispatch
    Applying plugin configuration to rabbit@localhost...
    The following plugins have been enabled:
      rabbitmq_management
      rabbitmq_management_agent
      rabbitmq_web_dispatch
    
    started 3 plugins.

打开链接：http://192.168.202.170:15672/

但是输入guest 密码和账户却提示：User can only log in via localhost 

然后开**始添加admin用户，并给予权限。**

    [root@localhost ~]# chown -R rabbitmq:rabbitmq /var/lib/rabbitmq/
    
    [root@localhost ~]# rabbitmqctl add_user admin 12345
    Adding user "admin" ...
    [root@localhost ~]# rabbitmqctl set_user_tags admin administrator
    Setting tags for user "admin" to [administrator] ...
    [root@localhost ~]# rabbitmqctl set_permissions -p / admin “.*” “.*” “.*”
    Setting permissions for user "admin" in vhost "/" ...

现在可以用admin 和12345密码登录了，get！


## spring BOOT整合

采用的springboot的版本为1.5.10

### 引入依赖

      <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-amqp</artifactId>
		</dependency>



链接出现问题如下：

1.java IO EXception

2.can not create queue

看了很多人的解决方案，基本都不行，尝试如下：

增加配置项，该项内容可以参考面板的用户界面：spring.rabbitmq.virtual-host=/

我尝试了了解rabbitMQ，并且增加一个queue hello，参考链接：https://blog.csdn.net/dandanzmc/article/details/52262850

然后再重启应用，ok了，再控制面板出现了链接信息。

2018-03-23 14:15:33.419  INFO 10068 --- [cTaskExecutor-1] o.s.a.r.c.CachingConnectionFactory       : Created new connection: rabbitConnectionFactory#51d7a771:0/SimpleConnection@4a8f8a8d [delegate=amqp://admin@192.168.202.170:5672/, localPort= 65100]

运行Test案例：

    Sender : hello Fri Mar 23 14:21:14 CST 2018

然后再主控的console收到：
    
    Receiver : hello Fri Mar 23 14:21:14 CST 2018

改写了一下测试，每隔100ms的queue队列发送msg，在面板可以看到变化：


[![20180323142831.png](https://s7.postimg.org/x3qfstruj/20180323142831.png)](https://postimg.org/image/rsbj845rr/)



[![20180323142831.png](https://s7.postimg.org/nuo9j4yij/20180323142831.png)](https://postimg.org/image/yu9guqoxj/)
    

更多的细致的内容，可以参考rabbitMQ官方的说明文档来进行测试。

还是那句老话，就是对任何产品的整合和使用，要多看官方文档，多研究，不过很多时候看看别人写的东西（虽然过时），但是可以了解别人做事的思路。三人行，必有吾师，多学习才能进步，多总结才能归为知识，多思考才有成长。
 	



