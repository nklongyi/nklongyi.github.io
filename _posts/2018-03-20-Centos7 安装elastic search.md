---
layout:     post
title:      Centos7 minimal 安装ElasticSearch
subtitle:   在虚拟机中制作ElasticSearch镜像
date:       2018-03-20
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - elasticsearch
    
---

## 简介

ElasticSearch 属于Elastic系列的产品之一，是互联网公司ELK日志分析平台的其中一个组成部件。

实验环境：centos7 minimal（已经安装部分小命令行工具）

IP地址：192.168.202.160

目的：基本了解Elastic search，基于RestFull的形式可以无侵入的整合到现有的系统。

## 安装步骤

### 1.下载必要的软件包

 1.a 去官方网站wget相关的包 ，官方网站：https://www.elastic.co/downloads/elasticsearch

     [root@localhost ~]# wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.2.2.tar.gz

 1.b jdk环境的安装（安装jdk8），Elastic是用jave实现的，需要jre环境。

     参考链接：https://www.digitalocean.com/community/tutorials/how-to-install-java-on-centos-and-fedora
    
    ME：yum install java-1.8.0-openjdk-devel

    [root@localhost ~]# javac -version
    javac 1.8.0_161


