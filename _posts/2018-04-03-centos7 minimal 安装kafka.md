---
layout:     post
title:      centos install Kafka
subtitle:   spring cloud消息代理的实践
date:       2018-04-04
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - 消息中间件
    - spring cloud
    
---

## 简介

kafka，一个高吞吐量的消息中间件，是apache的一个顶级项目，官方网址为：http://kafka.apache.org/，据说在PB级的数据量级上可以实现每秒百万的吞吐，在一些大型互联网公司都有其应用场景。



## 安装

### 安装之前务必安装好jdk

 ` [root@localhost kafka_2.11-1.1.0]# yum install java-1.8.0-openjdk-devel`

否则执行命令行出错，zookeeper是用java开发的。

### 去官方网站下载对应的安装包

  执行命令行 wget ，从官方链接load包（源码包）

### 启动zookeeper

 ` [root@localhost kafka_2.11-1.1.0]# ./bin/zookeeper-server-start.sh config/zookeeper.properties `
   
   zookeeper.properties这里采用默认配置（用来做测试），实际生产中可以使用外部zookeeper集群

  打出日志：

      [2018-04-03 18:46:33,742] INFO Server environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-04-03 18:46:33,742] INFO Server environment:java.io.tmpdir=/tmp (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-04-03 18:46:33,742] INFO Server environment:java.compiler=<NA> (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-04-03 18:46:33,742] INFO Server environment:os.name=Linux (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-04-03 18:46:33,742] INFO Server environment:os.arch=amd64 (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-04-03 18:46:33,742] INFO Server environment:os.version=3.10.0-693.17.1.el7.x86_64 (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-04-03 18:46:33,742] INFO Server environment:user.name=root (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-04-03 18:46:33,742] INFO Server environment:user.home=/root (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-04-03 18:46:33,742] INFO Server environment:user.dir=/root/kafka_2.11-1.1.0 (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-04-03 18:46:33,758] INFO tickTime set to 3000 (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-04-03 18:46:33,758] INFO minSessionTimeout set to -1 (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-04-03 18:46:33,758] INFO maxSessionTimeout set to -1 (org.apache.zookeeper.server.ZooKeeperServer)
    [2018-04-03 18:46:33,771] INFO binding to port 0.0.0.0/0.0.0.0:2181 (org.apache.zookeeper.server.NIOServerCnxnFactory)


### 启动kafka

    [root@localhost kafka_2.11-1.1.0]# ./bin/kafka-server-start.sh config/server.properties 

采用的默认配置，连接本地zookeeper的2181端口，实际中应配置为外部zookeeper集群。










