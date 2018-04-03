---
layout:     post
title:      spring cloud bus的解决方案
subtitle:   应用于spring cloud config
date:       2018-04-03
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - 消息中间件
    - spring cloud
    
---

## 简介

消息代理，message broker，俗称消息中间件，相关的好处在前面讲解rabbitMQ已经有所涉及。此处主要通过实践的方式来说明spring cloud bus来解决多节点的config同步刷新问题。

## 尝试

### 准备

1.将eureka注册中心开启

2.将config-server开启

3.将之前的rabbitMq启动（防火墙全部关闭）

### 改造config-client

1.添加依赖，引入bus

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    	<modelVersion>4.0.0</modelVersion>
    
    	<groupId>com.nklongyi</groupId>
    	<artifactId>config-client</artifactId>
    	<version>0.0.1-SNAPSHOT</version>
    	<packaging>jar</packaging>
    
    	<name>config-client</name>
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
    			<groupId>org.springframework.boot</groupId>
    			<artifactId>spring-boot-starter-actuator </artifactId>
    		</dependency>
    		<dependency>
    			<groupId>org.springframework.cloud</groupId>
    			<artifactId>spring-cloud-starter-config</artifactId>
    		</dependency>
    		<dependency>
    			<groupId>org.springframework.cloud</groupId>
    			<artifactId>spring-cloud-starter-bus-amqp</artifactId>
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
    

2.添加rabbitMq的配置项
    
    # 配置服务注册中心
    eureka.client.serviceUrl.defaultZone=http://localhost:1001/eureka/
    spring.application.name=configclient
    server.port=2001
    spring.cloud.config.discovery.enabled=true
    spring.cloud.config.discovery.serviceId=config-server
    spring.cloud.config.profile=dev
    management.security.enabled= false
    ## 配置rabbitMq
    spring.rabbitmq.host=192.168.202.170
    spring.rabbitmq.port=5672
    spring.rabbitmq.username=admin
    spring.rabbitmq.password=12345

3.启动

在启动日志中寻找/bus/refresh,如下：

[![20180403110123.png](https://s7.postimg.org/po0ygsr4r/20180403110123.png)](https://postimg.org/image/tkeacsc47/)

此时我们访问：http://localhost:2001/from，可以获取正确的配置信息。

### 测试

1.修改git仓库配置，提交到仓库；

2.打开链接from，发现没有更新

3.用postman做测试，post到/bus/refresh。

实际测试ok，更新.

## 架构优化

对于集群环境下，上一个案例中我们需要触发单个实例，触发哪个实例合适？实例与实例之间就是有区别了，如何做到后端的无差别，下面就是对集群结构进行的调整：

[![20180403110123.png](https://s7.postimg.org/6wz16k497/20180403110123.png)](https://postimg.org/image/r4cgyv1qf/)

变动点：

1.config-server连接到消息总线

2.bus refresh不再发到具体的实例，发送到配置服务器上即可触发集群配置更新，同时可以通过增加destination参数来指定更新实例。


简单测试一下：

1.将config-server增加bus依赖和actuator

2.将配置改为：

    ## 配置rabbitMq
    spring.rabbitmq.host=192.168.202.170
    spring.rabbitmq.port=5672
    spring.rabbitmq.username=admin
    spring.rabbitmq.password=12345
    #允许发送post测试
    management.security.enabled= false

最后一个很重要，否则post过去就是401，务必注意。

3.修改配置

4.查看实例变化（无变化）

5.给config-server post bus/refresh

6.过一会就发现了实例有了更新





