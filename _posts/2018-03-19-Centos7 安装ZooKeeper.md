---
layout:     post
title:      Centos7 minimal 安装ZooKeeper
subtitle:   在虚拟机中制作zookeeper的镜像
date:       2018-03-19
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - ZooKeeper
    
---

## 简介

Hadoop生态中所有的集群都依赖于zookeeper，比如storm集群的启动首先就需要配置zookeeper集群，可见zookeeper在整个生态中的作用是基础性的。

包括在淘宝dubbo框架中，也是建议zookeeper作为注册中心。zookeeper作为分布式应用的基础实施的作用开始体现。

此处，我采用centos7 minimal作为os，安装一个zookeeper的镜像，方便后期再本地搭建集群应用。

## 安装步骤（standalone）

1.下载镜像

    [root@localhost ~]# wget http://mirrors.hust.edu.cn/apache/zookeeper/zookeeper-3.4.11/zookeeper-3.4.11.tar.gz

此处我在官网使用的版本是3.4.11，在官网网址的推荐mirror中下载。

2.解压：
   `tar -zvxf zookeeper-3.4.11.tar.gz`

3.配置
  进入到zookeeper目录下，将zoo_sample.cfg改名为zoo.cfg。

      mv zoo_sample.cfg zoo.cfg
      
      ./bin/zkServer.sh start

  然后查看状态：
    
    [root@localhost zookeeper-3.4.11]# ./bin/zkServer.sh status
    ZooKeeper JMX enabled by default
    Using config: /root/zookeeper-3.4.11/bin/../conf/zoo.cfg
    Mode: standalone

关闭防火墙：

    [root@localhost zookeeper-3.4.11]# systemctl stop firewalld.service
    [root@localhost zookeeper-3.4.11]# firewall-cmd --state
    not running



在做dubble测试的时候，发现zookeeper报一个错误：

Opening socket connection to server localhost/0:0:0:0:0:0:0:1:2181. Will not attempt to authenticate

解决方案：

修改vi/etc/hosts，增加;;/ ipv6-localhost ipv6xxxx


