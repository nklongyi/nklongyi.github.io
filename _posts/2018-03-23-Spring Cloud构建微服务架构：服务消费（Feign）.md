---
layout:     post
title:      Spring Cloud构建微服务架构：服务消费（4）
subtitle:   Spring Cloud Feign使用
date:       2018-04-01
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - springcloud
    
---

## 简介

Spring Cloud Feign是一套基于Netflix Feign实现的声明式服务调用客户端。它使得编写Web服务客户端变得更加简单。我们只需要通过创建接口并用注解来配置它既可完成对Web服务接口的绑定。

它具备可插拔的注解支持，包括Feign注解、JAX-RS注解。它也支持可插拔的编码器和解码器。Spring Cloud Feign还扩展了对Spring MVC注解的支持，同时还整合了Ribbon和Eureka来提供均衡负载的HTTP客户端实现。



## 实践出真知

创建springboot项目，

### 添加依赖，增加Feign依赖项

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    	<modelVersion>4.0.0</modelVersion>
    
    	<groupId>com.nklongyi</groupId>
    	<artifactId>eureka-consumer-feign</artifactId>
    	<version>0.0.1-SNAPSHOT</version>
    	<packaging>jar</packaging>
    
    	<name>eureka-consumer-feign</name>
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
    			<artifactId>spring-boot-starter-actuator</artifactId>
    		</dependency>
    
    		<dependency>
    			<groupId>org.springframework.cloud</groupId>
    			<artifactId>spring-cloud-starter-feign</artifactId>
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


### 定义接口，用来FeignClient来绑定服务

    package com.nklongyi.service;
    
    import org.springframework.cloud.netflix.feign.FeignClient;
    import org.springframework.web.bind.annotation.GetMapping;
    
    /**
     * Created by longyi on 2018-04-01.
     */
    @FeignClient("eureka-client")
    public interface DcClient {
    
    @GetMapping("/dc")
    String consumer();
    }
    

### 主类支持Feign

    package com.nklongyi;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
    import org.springframework.cloud.netflix.feign.EnableFeignClients;
    
    @EnableFeignClients
    @EnableDiscoveryClient
    @SpringBootApplication
    public class EurekaConsumerFeignApplication {
    
    	public static void main(String[] args) {
    		SpringApplication.run(EurekaConsumerFeignApplication.class, args);
    	}
    }
    
### 修改调用类

将与服务绑定的接口注入到调用类，犹如调用本地应用一样。
    
    package com.nklongyi.controller;
    
    import com.nklongyi.service.DcClient;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RestController;
    
    /**
     * Created by longyi on 2018-04-01.
     */
    @RestController
    public class DcController {
    @Autowired
    DcClient dcClient;
    
    @GetMapping("/consumer")
    public String dc() {
    return dcClient.consumer();
    }
    }
    

### 配置

    spring.application.name=eureka-consumer-feign
    server.port=2101
    
    eureka.client.serviceUrl.defaultZone=http://localhost:1001/eureka/



通过Spring Cloud Feign来实现服务调用的方式更加简单了，通过@FeignClient定义的接口来统一的生命我们需要依赖的微服务接口。而在具体使用的时候就跟调用本地方法一点的进行调用即可。由于Feign是基于Ribbon实现的，所以它自带了客户端负载均衡功能，也可以通过Ribbon的IRule进行策略扩展。另外，Feign还整合的Hystrix来实现服务的容错保护，在Dalston版本中，Feign的Hystrix默认是关闭的。待后文介绍Hystrix带领大家入门之后，我们再结合介绍Feign中的Hystrix以及配置方式。

### 测试

1.开启注册中心

2.开启eureka client

3.开启feign客户端

[![QQ_20180401161611.png](https://s14.postimg.org/atoomcg9t/QQ_20180401161611.png)](https://postimg.org/image/q2em049y5/)

调用http://localhost:2101/consumer：

Services: [eureka-client]