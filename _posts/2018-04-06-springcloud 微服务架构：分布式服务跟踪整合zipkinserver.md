---
layout:     post
title:      springcloud 微服务架构：分布式服务跟踪整合zipkinserver
subtitle:   twinter zipkin server的使用
date:       2018-04-06
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - springcloud
    
---

>ELK平台可以实现对日志信息收集、存储、分析、搜索等强大的功能，对追踪信息的管理和使用变得非常便利，但是在ELK平台对数据的分析缺少对请求链中各服务节点之间时间延迟的关注，通过请求链的时间分析来确定链式请求中的性能瓶颈，或者是实现对分布式系统的自动化控制策略。对于这样的问题，就引入了zipkin。

## 简介

zipkin是twinter的一个开源项目，它基于Google Dapper实现。我们可以使用它来收集各个服务器上请求链路的跟踪数据，并通过它提供的REST API接口来辅助我们查询跟踪数据以实现对分布式系统的监控程序，从而及时地发现系统中出现的延迟升高问题并找出系统性能瓶颈的根源。除了面向开发的API接口之外，它也提供了方便的UI组件来帮助我们直观的搜索跟踪信息和分析请求链路明细，比如：可以查询某段时间内各用户请求的处理时间等。

[![20180327154227.png](https://s18.postimg.org/ph9o6l5qh/20180327154227.png)](https://postimg.org/image/wx8xsdtfp/)

展示了Zipkin的基础架构，它主要有4个核心组件构成：

- Collector：收集器组件，它主要用于处理从外部系统发送过来的跟踪信息，将这些信息转换为Zipkin内部处理的Span格式，以支持后续的存储、分析、展示等功能。
- Storage：存储组件，它主要对处理收集器接收到的跟踪信息，默认会将这些信息存储在内存中，我们也可以修改此存储策略，通过使用其他存储组件将跟踪信息存储到数据库中。
- RESTful API：API组件，它主要用来提供外部访问接口。比如给客户端展示跟踪信息，或是外接系统访问以实现监控等。
- Web UI：UI组件，基于API组件实现的上层应用。通过UI组件用户可以方便而有直观地查询和分析跟踪信息。

Spring Cloud Sleuth中对Zipkin的整合进行了自动化配置的封装，所以我们可以很轻松的引入和使用它。


## 快速入门

### 搭建zipkin server

创建一个springboot项目zipkin-server，引入相关依赖：

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    	<modelVersion>4.0.0</modelVersion>
    
    	<groupId>com.nklongyi</groupId>
    	<artifactId>zipkin-server</artifactId>
    	<version>0.0.1-SNAPSHOT</version>
    	<packaging>jar</packaging>
    
    	<name>zipkin-server</name>
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
    			<artifactId>spring-boot-starter</artifactId>
    		</dependency>
    		<dependency>
    			<groupId>io.zipkin.java</groupId>
    			<artifactId>zipkin-server</artifactId>
    		</dependency>
    		<dependency>
    			<groupId>io.zipkin.java</groupId>
    			<artifactId>zipkin-autoconfigure-ui</artifactId>
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

配置文件：

    spring.application.name=zipkin-server
    server.port=9411

对主类进行注释

    package com.nklongyi;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import zipkin.server.EnableZipkinServer;
    
    
    @EnableZipkinServer
    @SpringBootApplication
    public class ZipkinServerApplication {
    
    	public static void main(String[] args) {
    		SpringApplication.run(ZipkinServerApplication.class, args);
    	}
    }
    
然后就可以启动该项目了，启动完毕后，打开http://localhost:9411，界面如下：

[![20180327154227.png](https://s18.postimg.org/9x2ag23bt/20180327154227.png)](https://postimg.org/image/hd1k1ur11/)

### 在每个服务中加入zipkin服务和配置

对每个服务添加依赖：

    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-sleuth-zipkin</artifactId>
    </dependency>

然后再配置文件中，增加zipkin server的地址信息

    spring.zipkin.base-url=http://localhost:9411

重新启动服务应用，访问trace1服务，正常返回。

### 测试zipkin的链追踪功能

不断的刷新连接http://localhost:9101/trace-1，然后再日志里找到true

    2018-04-06 22:53:58.100  INFO [trace-1,29d2a2b94a19b44d,29d2a2b94a19b44d,false] 9380 --- [nio-9101-exec-5] ication$$EnhancerBySpringCGLIB$$9e4ee5c7 : ===call trace-1===
    2018-04-06 22:53:58.285  INFO [trace-1,a59e909644259eff,a59e909644259eff,false] 9380 --- [nio-9101-exec-7] ication$$EnhancerBySpringCGLIB$$9e4ee5c7 : ===call trace-1===
    2018-04-06 22:53:58.452  INFO [trace-1,4ee069471bea3079,4ee069471bea3079,false] 9380 --- [nio-9101-exec-9] ication$$EnhancerBySpringCGLIB$$9e4ee5c7 : ===call trace-1===
    2018-04-06 22:53:58.637  INFO [trace-1,28d5634b6d726d2f,28d5634b6d726d2f,true] 9380 --- [nio-9101-exec-1] ication$$EnhancerBySpringCGLIB$$9e4ee5c7 : ===call trace-1===

此时我们就可以发现，其实发送链的信息是抽样的（至少默认是抽样的），true说明有信息发送给了zipkin server。然后我们可以去zipkin server面板去查看该请求链：

[![20180327154227.png](https://s18.postimg.org/4bkgpa261/20180327154227.png)](https://postimg.org/image/vm5rx752t/)

然后具体点击trace-1，可以看到具体的调用详情：

[![20180406233409.png](https://s18.postimg.org/jkae3hbeh/20180406233409.png)](https://postimg.org/image/s2ju7thx1/)

[![20180406233445.png](https://s18.postimg.org/rd11v9715/20180406233445.png)](https://postimg.org/image/u7478p979/)

[![20180406233545.png](https://s18.postimg.org/ch2inoy7d/20180406233545.png)](https://postimg.org/image/pl830dq91/)

点击dependencies选项，则界面如下：

[![20180406233824.png](https://s18.postimg.org/75nm34p15/20180406233824.png)](https://postimg.org/image/qaqvcw3p1/)








