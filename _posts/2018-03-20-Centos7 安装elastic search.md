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

软件版本：目前最新6.2.2

## 安装步骤

### 1.下载必要的软件包

 1.a 去官方网站wget相关的包 ，官方网站：https://www.elastic.co/downloads/elasticsearch

     [root@localhost ~]# wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.2.2.tar.gz

 1.b jdk环境的安装（安装jdk8），Elastic是用jave实现的，需要jre环境。

     参考链接：https://www.digitalocean.com/community/tutorials/how-to-install-java-on-centos-and-fedora
    
    ME：yum install java-1.8.0-openjdk-devel

    [root@localhost ~]# javac -version
    javac 1.8.0_161


### 2.运行

 按照官方网站的提示，Run bin/elasticsearch。
 提示：

    [root@localhost elasticsearch-6.2.2]# bin/elasticsearch
    OpenJDK 64-Bit Server VM warning: If the number of processors is expected to increase from one, then you should configure the number of parallel GC threads appropriately using -XX:ParallelGCThreads=N
    [2018-03-21T13:36:15,381][WARN ][o.e.b.ElasticsearchUncaughtExceptionHandler] [] uncaught exception in thread [main]
    org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root
    	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:125) ~[elasticsearch-6.2.2.jar:6.2.2]
    	at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:112) ~[elasticsearch-6.2.2.jar:6.2.2]
    	at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86) ~[elasticsearch-6.2.2.jar:6.2.2]
    	at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:124) ~[elasticsearch-cli-6.2.2.jar:6.2.2]
    	at org.elasticsearch.cli.Command.main(Command.java:90) ~[elasticsearch-cli-6.2.2.jar:6.2.2]
    	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:92) ~[elasticsearch-6.2.2.jar:6.2.2]
    	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:85) ~[elasticsearch-6.2.2.jar:6.2.2]
    Caused by: java.lang.RuntimeException: can not run elasticsearch as root
    	at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:105) ~[elasticsearch-6.2.2.jar:6.2.2]
    	at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:172) ~[elasticsearch-6.2.2.jar:6.2.2]
    	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:323) ~[elasticsearch-6.2.2.jar:6.2.2]

核心错误：root运行了该应用

方法：
  1.增加elastic 用户：`[root@localhost elasticsearch-6.2.2]# useradd elastic`
  
  2.cd 到home下的elastic用户目录，cp elasticsearch-6.2.2到该目录下

  3.更改权限：`chown -R elastic:elastic elasticsearch-6.2.2`

  4.切换用户：`su elastic`

  5.运行：`[elastic@localhost elasticsearch-6.2.2]$ ./bin/elasticsearch`

  状态：
    
	……
    [2018-03-21T15:16:57,759][INFO ][o.e.p.PluginsService ] [iRK8e6Y] loaded module [analysis-common]
    [2018-03-21T15:16:57,759][INFO ][o.e.p.PluginsService ] [iRK8e6Y] loaded module [ingest-common]
    [2018-03-21T15:16:57,759][INFO ][o.e.p.PluginsService ] [iRK8e6Y] loaded module [lang-expression]
    [2018-03-21T15:16:57,759][INFO ][o.e.p.PluginsService ] [iRK8e6Y] loaded module [lang-mustache]
    [2018-03-21T15:16:57,759][INFO ][o.e.p.PluginsService ] [iRK8e6Y] loaded module [lang-painless]
    [2018-03-21T15:16:57,759][INFO ][o.e.p.PluginsService ] [iRK8e6Y] loaded module [mapper-extras]
    [2018-03-21T15:16:57,759][INFO ][o.e.p.PluginsService ] [iRK8e6Y] loaded module [parent-join]
    [2018-03-21T15:16:57,759][INFO ][o.e.p.PluginsService ] [iRK8e6Y] loaded module [percolator]
    [2018-03-21T15:16:57,759][INFO ][o.e.p.PluginsService ] [iRK8e6Y] loaded module [rank-eval]
    [2018-03-21T15:16:57,759][INFO ][o.e.p.PluginsService ] [iRK8e6Y] loaded module [reindex]
    [2018-03-21T15:16:57,759][INFO ][o.e.p.PluginsService ] [iRK8e6Y] loaded module [repository-url]
    [2018-03-21T15:16:57,759][INFO ][o.e.p.PluginsService ] [iRK8e6Y] loaded module [transport-netty4]
    [2018-03-21T15:16:57,760][INFO ][o.e.p.PluginsService ] [iRK8e6Y] loaded module [tribe]
    [2018-03-21T15:16:57,760][INFO ][o.e.p.PluginsService ] [iRK8e6Y] no plugins loaded
    [2018-03-21T15:17:03,352][INFO ][o.e.d.DiscoveryModule] [iRK8e6Y] using discovery type [zen]
    [2018-03-21T15:17:04,272][INFO ][o.e.n.Node   ] initialized
    [2018-03-21T15:17:04,272][INFO ][o.e.n.Node   ] [iRK8e6Y] starting ...
    [2018-03-21T15:17:04,727][INFO ][o.e.t.TransportService   ] [iRK8e6Y] publish_address {127.0.0.1:9300}, bound_addresses {[::1]:9300}, {127.0.0.1:9300}
    [2018-03-21T15:17:04,742][WARN ][o.e.b.BootstrapChecks] [iRK8e6Y] max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
    [2018-03-21T15:17:04,742][WARN ][o.e.b.BootstrapChecks] [iRK8e6Y] max number of threads [3816] for user [elastic] is too low, increase to at least [4096]
    [2018-03-21T15:17:04,742][WARN ][o.e.b.BootstrapChecks] [iRK8e6Y] max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
    [2018-03-21T15:17:07,861][INFO ][o.e.c.s.MasterService] [iRK8e6Y] zen-disco-elected-as-master ([0] nodes joined), reason: new_master {iRK8e6Y}{iRK8e6YeRRWnbLfqJpHP0A}{NOMctzKcSqK5ms9ETUvZqg}{127.0.0.1}{127.0.0.1:9300}
    [2018-03-21T15:17:07,871][INFO ][o.e.c.s.ClusterApplierService] [iRK8e6Y] new_master {iRK8e6Y}{iRK8e6YeRRWnbLfqJpHP0A}{NOMctzKcSqK5ms9ETUvZqg}{127.0.0.1}{127.0.0.1:9300}, reason: apply cluster state (from master [master {iRK8e6Y}{iRK8e6YeRRWnbLfqJpHP0A}{NOMctzKcSqK5ms9ETUvZqg}{127.0.0.1}{127.0.0.1:9300} committed version [1] source [zen-disco-elected-as-master ([0] nodes joined)]])
    [2018-03-21T15:17:07,914][INFO ][o.e.g.GatewayService ] [iRK8e6Y] recovered [0] indices into cluster_state
    [2018-03-21T15:17:07,939][INFO ][o.e.h.n.Netty4HttpServerTransport] [iRK8e6Y] publish_address {127.0.0.1:9200}, bound_addresses {[::1]:9200}, {127.0.0.1:9200}
    [2018-03-21T15:17:07,939][INFO ][o.e.n.Node   ] [iRK8e6Y] started
    

  开个terminal，运行：

    [root@localhost ~]# curl http://localhost:9200/
    {
      "name" : "iRK8e6Y",
      "cluster_name" : "elasticsearch",
      "cluster_uuid" : "4mtzuxehRJCONFD2j81cKg",
      "version" : {
    "number" : "6.2.2",
    "build_hash" : "10b1edd",
    "build_date" : "2018-02-16T19:01:30.685723Z",
    "build_snapshot" : false,
    "lucene_version" : "7.2.1",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
      },
      "tagline" : "You Know, for Search"
    }
    

### 3.开启外网访问

按照要求，编辑conf目录下的yml配置文件

把 network.host 和 http.port 前面的 备注去掉，改为自己的。

然后我们把防火墙也关了 

systemctl stop firewalld.service

systemctl disable firewalld.service   禁止防火墙开机启动

重启启动，发现很多问题，提示很多错误，相关的错误解决方案的链接：[http://blog.csdn.net/weini1111/article/details/60468068](http://blog.csdn.net/weini1111/article/details/60468068)


## java客户端连接

新建一个maven项目，pom内容如下：

     <dependencies>
    <dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>transport</artifactId>
    <version>5.5.2</version>
    </dependency>
    </dependencies>
    
然后新建一个demo类，内容如下：
    
    package com.nklongyi.demo1;
    
    import org.elasticsearch.client.transport.TransportClient;
    import org.elasticsearch.common.settings.Settings;
    import org.elasticsearch.common.transport.InetSocketTransportAddress;
    import org.elasticsearch.transport.client.PreBuiltTransportClient;
    
    import java.net.InetAddress;
    import java.net.UnknownHostException;
    
    /**
     * Created by longyi on 2018-03-21.
     */
    public class Demo1 {
    
    private static String host= "192.168.202.160";
    private static int port=9002;
    /**
     * 
     */
    public static void main(String[] args) throws UnknownHostException {
       InetAddress address = InetAddress.getByName(Demo1.host);
    TransportClient client = new PreBuiltTransportClient(Settings.EMPTY)
    .addTransportAddress(new InetSocketTransportAddress(address,Demo1.port));
    
    System.out.println(client);
    
    client.close();
    
    }
    }
    

运行：

org.elasticsearch.transport.client.PreBuiltTransportClient@3c49fab6





