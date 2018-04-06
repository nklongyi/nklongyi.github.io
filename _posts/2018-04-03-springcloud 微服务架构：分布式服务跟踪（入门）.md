---
layout:     post
title:      springcloud 微服务架构：分布式服务跟踪（入门）
subtitle:   spring cloud sleuth的使用
date:       2018-04-04
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - springcloud
    
---

## 简介

随着微服务的规模越来越大，一个客户端发起的请求通常需要经过多个服务处理结束后返回结果，在复杂的微服务架构中几乎每一个前端请求都会形成一条复杂的分布式服务调用链路。

对于全链的追踪，有利于监控整个服务的质量，监控系统的瓶颈和评测可能出现的故障点，在此基础之上制定对应的自动化管理策略。针对这些链追踪的实现，spring cloud提供了sleuth方案，该方案能够为分布式微服务应用提供服务追踪的能力。


## 快速入门

通过简单的实例，对存在调用的服务增加相应的sleuth的配置实现基本的服务跟踪功能，以此来对sleuth有一个初步的概念和认识。

### 准备工作

1.eureka注册中心

 之前我们做的很多次实验，就是采用该注册中心的。

2.微服务的应用trace-1，该服务依赖于应用trace-2，下面具体来实现

#### 创建spring boot项目trace-1

**1.引入项目相关的依赖：**

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    	<modelVersion>4.0.0</modelVersion>
    
    	<groupId>com.nklongyi</groupId>
    	<artifactId>trace-1</artifactId>
    	<version>0.0.1-SNAPSHOT</version>
    	<packaging>jar</packaging>
    
    	<name>trace-1</name>
    	<description>Demo project for Spring Boot</description>
    
    	<parent>
    		<groupId>org.springframework.boot</groupId>
    		<artifactId>spring-boot-starter-parent</artifactId>
    		<version>1.5.11.RELEASE</version>
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
    			<groupId>org.springframework.cloud</groupId>
    			<artifactId>spring-cloud-starter-ribbon</artifactId>
    		</dependency>
    		<dependency>
    			<groupId>org.springframework.cloud</groupId>
    			<artifactId>spring-cloud-starter-sleuth</artifactId>
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
    				<version>Dalston.SR5</version>
    				<type>pom</type>
    				<scope>import</scope>
    			</dependency>
    		</dependencies>
    	</dependencyManagement>
    
    
    </project>

**2.添加配置项，注册中心的配置和本服务的配置**
    
    spring.application.name=trace-1
    server.port=9101
    
    eureka.client.serviceUrl.defaultZone=http://localhost:1001/eureka/

**3.增加主类的注释和定义服务，依赖于trace2**

    package com.nklongyi;
    
    import org.apache.log4j.Logger;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
    import org.springframework.cloud.client.loadbalancer.LoadBalanced;
    import org.springframework.context.annotation.Bean;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.bind.annotation.RestController;
    import org.springframework.web.client.RestTemplate;
    
    
    @RestController
    @EnableDiscoveryClient
    @SpringBootApplication
    public class Trace1Application {
    
    	private final Logger logger = Logger.getLogger(getClass());
    
    	@Bean
    	@LoadBalanced
    	RestTemplate restTemplate(){
    		return new RestTemplate();
    	}
    
    	@RequestMapping(value = "/trace-1", method = RequestMethod.GET)
    	public String trace() {
    		logger.info("===call trace-1===");
    		return restTemplate().getForEntity("http://trace-2/trace-2", String.class).getBody();
    	}
    
    	public static void main(String[] args) {
    		SpringApplication.run(Trace1Application.class, args);
    	}
    }
 

#### 创建spring boot项目trace2

1.POM依赖于trace1相同

2.配置项如下：

    spring.application.name=trace-2
    server.port=9102
    
    eureka.client.serviceUrl.defaultZone=http://localhost:1001/eureka/  

3.主类增加注释 

    package com.nklongyi;
    
    import org.apache.log4j.Logger;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.bind.annotation.RestController;
    
    
    @RestController
    @EnableDiscoveryClient
    @SpringBootApplication
    public class Trace2Application {
    	private final Logger logger = Logger.getLogger(getClass());
    
    	@RequestMapping(value = "/trace-2", method = RequestMethod.GET)
    	public String trace() {
    		logger.info("===<call trace-2>===");
    		return "Trace";
    	}
    
    
    	public static void main(String[] args) {
    		SpringApplication.run(Trace2Application.class, args);
    	}
    }
   


### 测试与验证工作

1.依次启动注册中心、trace1、trace2等项目，可以在注册中心找到应对的instance

2.在浏览器中输入：http://localhost:9101/trace-1，此时可以发现控制台变化：

在trace1应用的控制台：

[![20180327154227.png](https://s18.postimg.org/ermsmomu1/20180327154227.png)](https://postimg.org/image/nmnmx7bmd/)

在trace2应用的控制台：

[![20180327154227.png](https://s18.postimg.org/nmnmxbe89/20180327154227.png)](https://postimg.org/image/bks9364zp/)

从控制台可以看到信息：
2018-04-06 21:01:36.711  INFO **[trace-1,2a0e948cee4c017a,2a0e948cee4c017a,false]** 10032 --- [nio-9101-exec-7] ication$$EnhancerBySpringCGLIB$$c5ec6281 : ===call trace-1===

其中：`trace-1`，记录了应用的名称

第二个值`2a0e948cee4c017a`：Spring Cloud Sleuth生成的一个ID，称为Trace ID，它用来标识一条请求链路。一条请求链路中包含一个Trace ID，多个Span ID。

第三个值`2a0e948cee4c017a`，Spring Cloud Sleuth生成的另外一个ID，称为Span ID，它表示一个基本的工作单元，比如：发送一个HTTP请求。

第四个boolean值，表示是否要将该信息输出到Zipkin等服务中来收集和展示。

Trace ID和Span ID是Spring Cloud Sleuth实现分布式服务跟踪的核心。在一次服务请求链路的调用过程中，会保持并传递同一个Trace ID，从而将整个分布于不同微服务进程中的请求跟踪信息串联起来，以上面输出内容为例，trace-1和trace-2同属于一个前端服务请求来源，所以他们的Trace ID是相同的，处于同一条请求链路中。







