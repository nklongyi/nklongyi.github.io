---
layout:     post
title:      springcloud服务中心与注册、发现
subtitle:   eureka
date:       2018-04-01
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - springcloud
    
---

## 简介


## 实现注册中心

Spring Cloud Eureka是Spring Cloud Netflix项目下的服务治理模块。而Spring Cloud Netflix项目是Spring Cloud的子项目之一，主要内容是对Netflix公司一系列开源产品的包装，它为Spring Boot应用提供了自配置的Netflix OSS整合。通过一些简单的注解，开发者就可以快速的在应用中配置一下常用模块并构建庞大的分布式系统。它主要提供的模块包括：服务发现（Eureka），断路器（Hystrix），智能路由（Zuul），客户端负载均衡（Ribbon）等。

### **具体步骤：**

引入依赖：

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    	<modelVersion>4.0.0</modelVersion>
    
    	<groupId>com.nklongyi</groupId>
    	<artifactId>eureka-server</artifactId>
    	<version>0.0.1-SNAPSHOT</version>
    	<packaging>jar</packaging>
    
    	<name>eureka-server</name>
    	<description>eureka-server</description>
    
    	<parent>
    		<groupId>org.springframework.boot</groupId>
    		<artifactId>spring-boot-starter-parent</artifactId>
    		<version>1.5.10.RELEASE</version>
    		<relativePath/> <!-- lookup parent from repository -->
    	</parent>
    
    	<properties>
    		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    		<java.version>1.8</java.version>
    	</properties>
    
    	<dependencies>
    		<dependency>
    			<groupId>org.springframework.boot</groupId>
    			<artifactId>spring-boot-starter</artifactId>
    		</dependency>
    
    			<dependency>
    				<groupId>org.springframework.cloud</groupId>
    				<artifactId>spring-cloud-starter-eureka-server</artifactId>
    			</dependency>
    
    		<dependency>
    			<groupId>org.springframework.boot</groupId>
    			<artifactId>spring-boot-starter-test</artifactId>
    			<scope>test</scope>
    		</dependency>
    	</dependencies>
    
    	<build>
    		<plugins>
    			<plugin>
    				<groupId>org.springframework.boot</groupId>
    				<artifactId>spring-boot-maven-plugin</artifactId>
    			</plugin>
    		</plugins>
    	</build>
    
    	<dependencyManagement>
    		<dependencies>
    			<dependency>
    				<groupId>org.springframework.cloud</groupId>
    				<artifactId>spring-cloud-dependencies</artifactId>
    				<version>Dalston.SR1</version>
    				<type>pom</type>
    				<scope>import</scope>
    			</dependency>
    		</dependencies>
    	</dependencyManagement>
    
    </project>

属性配置properties：
    
    spring.application.name=eureka-server
    server.port=1001
    
    eureka.instance.hostname=localhost
    eureka.client.register-with-eureka=false
    eureka.client.fetch-registry=false

在主函数上添加注解：
    
    package com.nklongyi;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
    
    @EnableEurekaServer
    @SpringBootApplication
    public class EurekaServerApplication {
    
    	public static void main(String[] args) {
    		SpringApplication.run(EurekaServerApplication.class, args);
    	}
    }
    
启动，在浏览器输入：http://localhost:1001/

出现下图：

[![QQ_20180401161611.png](https://s14.postimg.org/ourhgis8h/QQ_20180401161611.png)](https://postimg.org/image/505fued0t/)

## 创建服务提供方

创建项目，eureka_client:

1.添加依赖：

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    	<modelVersion>4.0.0</modelVersion>
    
    	<groupId>com.nklongyi</groupId>
    	<artifactId>eureka-client</artifactId>
    	<version>0.0.1-SNAPSHOT</version>
    	<packaging>jar</packaging>
    
    	<name>eureka-client</name>
    	<description>Demo project for Spring Boot</description>
    
    	<parent>
    		<groupId>org.springframework.boot</groupId>
    		<artifactId>spring-boot-starter-parent</artifactId>
    		<version>1.5.10.RELEASE</version>
    		<relativePath/> <!-- lookup parent from repository -->
    	</parent>
    
    	<properties>
    		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    		<java.version>1.8</java.version>
    	</properties>
    
    	<dependencies>
    		<dependency>
    			<groupId>org.springframework.boot</groupId>
    			<artifactId>spring-boot-starter-web</artifactId>
    		</dependency>
    		<dependency>
    			<groupId>org.springframework.cloud</groupId>
    			<artifactId>spring-cloud-starter-eureka</artifactId>
    		</dependency>
    
    		<dependency>
    			<groupId>org.springframework.boot</groupId>
    			<artifactId>spring-boot-starter-test</artifactId>
    			<scope>test</scope>
    		</dependency>
    	</dependencies>
    
    	<build>
    		<plugins>
    			<plugin>
    				<groupId>org.springframework.boot</groupId>
    				<artifactId>spring-boot-maven-plugin</artifactId>
    			</plugin>
    		</plugins>
    	</build>
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR1</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
    
    
    </project>


2.实现dc处理接口

    package com.nklongyi.controller;
    
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.cloud.client.discovery.DiscoveryClient;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RestController;
    
    /**
     * Created by longyi on 2018-04-01.
     */
    @RestController
    public class DcController {
    
    @Autowired
    DiscoveryClient discoveryClient;
    
    @GetMapping("/dc")
    public String dc() {
    String services = "Services: " + discoveryClient.getServices();
    System.out.println(services);
    return services;
    }
    }


3.添加配置：

    spring.application.name=eureka-client
    server.port=2001
    eureka.client.serviceUrl.defaultZone=http://localhost:1001/eureka/ 

4.在主函数中添加@EnableDiscoveryClient，启动


在服务中心的IP地址中http://localhost:1001/，我们看到：

[![QQ_20180401161611.png](https://s14.postimg.org/5urzg95e9/QQ_20180401161611.png)](https://postimg.org/image/k17qbhg99/)

同时我们通过访问：http://localhost:2001/dc，返回：

Services: [eureka-client]

## Spring Cloud Consul

Spring Cloud Consul项目是针对Consul的服务治理实现。Consul是一个分布式高可用的系统，它包含多个组件，但是作为一个整体，在微服务架构中为我们的基础设施提供服务发现和服务配置的工具。它包含了下面几个特性：

 
- 服务发现
- 健康检查
- Key/Value存储
- 多数据中心

由于Spring Cloud Consul项目的实现，我们可以轻松的将基于Spring Boot的微服务应用注册到Consul上，并通过此实现微服务架构中的服务治理。

去官网下载服务端：https://www.consul.io/intro/getting-started/install.html

1.解压以后，放电脑某个位置

2.添加环境变量PATH，注意分号

启动终端，输入consul，如下：

    longyi@longyi-PC ~
    $ consul
    Usage: consul [--version] [--help] <command> [<args>]
    
    Available commands are:
    agent  Runs a Consul agent
    catalogInteract with the catalog
    event  Fire a new event
    exec   Executes a command on Consul nodes
    force-leaveForces a member of the cluster to enter the "left" state
    info   Provides debugging information for operators.
    join   Tell Consul agent to join cluster
    keygen Generates a new encryption key
    keyringManages gossip layer encryption keys
    kv Interact with the key-value store
    leave  Gracefully leaves the Consul cluster and shuts down
    lock   Execute a command holding a lock
    maint  Controls node or service maintenance mode
    membersLists the members of a Consul cluster
    monitorStream logs from a Consul agent
    operator   Provides cluster-level tools for Consul operators
    reload Triggers the agent to reload configuration files
    rttEstimates network round trip time between nodes
    snapshot   Saves, restores and inspects snapshots of Consul server state
    validate   Validate config files/directories
    versionPrints the Consul version
    watch  Watch for changes in Consul

表示安装成功；

启动：consul agent -dev

    $ consul agent -dev
    ==> Starting Consul agent...
    ==> Consul agent running!
       Version: 'v1.0.6'
       Node ID: '4668929a-0650-b5d8-6aa7-83c5967539e7'
     Node name: 'longyi-PC'
    Datacenter: 'dc1' (Segment: '<all>')
    Server: true (Bootstrap: false)
       Client Addr: [127.0.0.1] (HTTP: 8500, HTTPS: -1, DNS: 8600)
      Cluster Addr: 127.0.0.1 (LAN: 8301, WAN: 8302)
       Encrypt: Gossip: false, TLS-Outgoing: false, TLS-Incoming: false
    
    ==> Log data will now stream in as it occurs:
    
    2018/04/01 17:10:40 [DEBUG] Using random ID "4668929a-0650-b5d8-6aa7-83c5967539e7" as node ID
    2018/04/01 17:10:40 [INFO] raft: Initial configuration (index=1): [{Suffrage:Voter ID:4668929a-0650-b5d8-6aa7-83c5967539e7 Address:127.0.0.1:8300}]
    2018/04/01 17:10:40 [INFO] raft: Node at 127.0.0.1:8300 [Follower] entering Follower state (Leader: "")
    2018/04/01 17:10:40 [INFO] serf: EventMemberJoin: longyi-PC.dc1 127.0.0.1
    2018/04/01 17:10:40 [INFO] serf: EventMemberJoin: longyi-PC 127.0.0.1
    2018/04/01 17:10:40 [INFO] consul: Adding LAN server longyi-PC (Addr: tcp/127.0.0.1:8300) (DC: dc1)
    2018/04/01 17:10:40 [INFO] consul: Handled member-join event for server "longyi-PC.dc1" in area "wan"
    2018/04/01 17:10:40 [INFO] agent: Started DNS server 127.0.0.1:8600 (udp)
    2018/04/01 17:10:40 [INFO] agent: Started DNS server 127.0.0.1:8600 (tcp)
    2018/04/01 17:10:40 [INFO] agent: Started HTTP server on 127.0.0.1:8500 (tcp)
    2018/04/01 17:10:40 [INFO] agent: started state syncer
    2018/04/01 17:10:40 [WARN] raft: Heartbeat timeout from "" reached, starting election
    2018/04/01 17:10:40 [INFO] raft: Node at 127.0.0.1:8300 [Candidate] entering Candidate state in term 2
    2018/04/01 17:10:40 [DEBUG] raft: Votes needed: 1
    2018/04/01 17:10:40 [DEBUG] raft: Vote granted from 4668929a-0650-b5d8-6aa7-83c5967539e7 in term 2. Tally: 1
    2018/04/01 17:10:40 [INFO] raft: Election won. Tally: 1
    2018/04/01 17:10:40 [INFO] raft: Node at 127.0.0.1:8300 [Leader] entering Leader state
    2018/04/01 17:10:40 [INFO] consul: cluster leadership acquired
    2018/04/01 17:10:40 [INFO] consul: New leader elected: longyi-PC
    2018/04/01 17:10:40 [DEBUG] consul: Skipping self join check for "longyi-PC" since the cluster is too small
    2018/04/01 17:10:40 [INFO] consul: member 'longyi-PC' joined, marking health alive
    2018/04/01 17:10:41 [DEBUG] Skipping remote check "serfHealth" since it is managed automatically
    2018/04/01 17:10:41 [INFO] agent: Synced node info
    2018/04/01 17:10:41 [DEBUG] agent: Node info in sync
    2018/04/01 17:10:41 [DEBUG] Skipping remote check "serfHealth" since it is managed automatically
    2018/04/01 17:10:41 [DEBUG] agent: Node info in sync
    
打开： Started HTTP server on 127.0.0.1:8500 (tcp)

[![QQ_20180401161611.png](https://s14.postimg.org/4m8p8nd75/QQ_20180401161611.png)](https://postimg.org/image/lzizni8i5/)

启动consul客户端，出现如下图，eureka_clien的状态是fail。

[![QQ_20180401161611.png](https://s14.postimg.org/6s302uj4x/QQ_20180401161611.png)](https://postimg.org/image/50a17xzrx/)

并没有出现passing状态。

去console看相关日志：

      2018/04/01 17:20:15 [WARN] agent: Check "service:eureka-client-2001" is now critical
    2018/04/01 17:20:25 [WARN] agent: Check "service:eureka-client-2001" is now critical
    2018/04/01 17:20:35 [WARN] agent: Check "service:eureka-client-2001" is now critical
    2018/04/01 17:20:40 [DEBUG] consul: Skipping self join check for "longyi-PC" since the cluster is too small

并不知道相关的原因。待后续深入了解后再仔细看看这一块的原因！