---
layout:     post
title:      Centos7 安装MongoDB
subtitle:   springboot的整合使用
date:       2018-03-23
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Mongdb
    
---

## 简介

MongoDB是一个基于分布式文件存储的数据库，它是一个介于关系数据库和非关系数据库之间的产品，其主要目标是在键/值存储方式（提供了高性能和高度伸缩性）和传统的RDBMS系统（具有丰富的功能）之间架起一座桥梁，它集两者的优势于一身。

较常见的，我们可以直接用MongoDB来存储键值对类型的数据，如：验证码、Session等；由于MongoDB的横向扩展能力，也可以用来存储数据规模会在未来变的非常巨大的数据，如：日志、评论等；由于MongoDB存储数据的弱类型，也可以用来存储一些多变json数据，如：与外系统交互时经常变化的JSON报文。而对于一些对数据有复杂的高事务性要求的操作，如：账户交易等就不适合使用MongoDB来存储。

官方网站：https://www.mongodb.com/

## 安装mondb镜像

### **第一步需要安装Db：**

[root@localhost ~]# vi /etc/yum.repos.d/mongodb-org.repo

添加：

    [mongodb-org-3.4]
    name=MongoDB Repository
    baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/
    gpgcheck=1
    enabled=1
    gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc


    执行：yum repolist



安装：

擦……竟然网络拦截了……what a pity！


----------

将电脑带回家，重新进行安装

执行：yum repolist

[root@localhost ~]# yum install mongodb-org

……
    
    Created symlink from /etc/systemd/system/multi-user.target.wants/mongod.service to /usr/lib/systemd/system/mongod.service.
      Installing : mongodb-org-shell-3.4.14-1.el7.x86_64  2/5 
      Installing : mongodb-org-tools-3.4.14-1.el7.x86_64  3/5 
      Installing : mongodb-org-mongos-3.4.14-1.el7.x86_64 4/5 
      Installing : mongodb-org-3.4.14-1.el7.x86_645/5 
      Verifying  : mongodb-org-mongos-3.4.14-1.el7.x86_64 1/5 
      Verifying  : mongodb-org-tools-3.4.14-1.el7.x86_64  2/5 
      Verifying  : mongodb-org-shell-3.4.14-1.el7.x86_64  3/5 
      Verifying  : mongodb-org-3.4.14-1.el7.x86_644/5 
      Verifying  : mongodb-org-server-3.4.14-1.el7.x86_64 5/5 
    
    Installed:
      mongodb-org.x86_64 0:3.4.14-1.el7   
    
    Dependency Installed:
      mongodb-org-mongos.x86_64 0:3.4.14-1.el7   mongodb-org-server.x86_64 0:3.4.14-1.el7   mongodb-org-shell.x86_64 0:3.4.14-1.el7   mongodb-org-tools.x86_64 0:3.4.14-1.el7  
    
    Complete!



