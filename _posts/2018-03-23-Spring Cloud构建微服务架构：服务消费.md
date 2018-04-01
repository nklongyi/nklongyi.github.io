---
layout:     post
title:      Spring Cloud构建微服务架构：服务消费(2)
subtitle:   服务消费
date:       2018-04-01
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - springcloud
    
---

## 简介

>我们已经成功地将服务提供者：eureka-client或consul-client注册到了Eureka服务注册中心或Consul服务端上了，同时我们也通过DiscoveryClient接口的getServices获取了当前客户端缓存的所有服务清单，那么接下来我们要学习的就是：如何去消费服务提供者的接口



## 使用LoadBalancerClient

LoadBalancerClient接口的命名中，我们就知道这是一个负载均衡客户端的抽象定义，下面我们就看看如何使用Spring Cloud提供的负载均衡器客户端接口来实现服务的消费。

idea 创建项目eureka_consumer项目


### 引入依赖包

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    	<modelVersion>4.0.0</modelVersion>
    
    	<groupId>com.nklongyi</groupId>
    	<artifactId>eureka-consumer</artifactId>
    	<version>0.0.1-SNAPSHOT</version>
    	<packaging>jar</packaging>
    
    	<name>eureka-consumer</name>
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
    
### 配置注册中心和应用

    spring.application.name=eureka-consumer
    server.port=2101
    
    eureka.client.serviceUrl.defaultZone=http://localhost:1001/eureka/

### 主函数

    package com.nklongyi;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
    import org.springframework.context.annotation.Bean;
    import org.springframework.web.client.RestTemplate;
    
    @EnableDiscoveryClient
    @SpringBootApplication
    public class EurekaConsumerApplication {
    
    	@Bean
    	public RestTemplate restTemplate(){
    		return new RestTemplate();
    	}
    
    	public static void main(String[] args) {
    		SpringApplication.run(EurekaConsumerApplication.class, args);
    	}
    }
    
添加注解EnableDiscoveryClient，同时初始化bean restTemplate。

### 创建消费接口，消费eureka_client 提供的dc接口

    package com.nklongyi.controller;
    
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.cloud.client.ServiceInstance;
    import org.springframework.cloud.client.loadbalancer.LoadBalancerClient;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RestController;
    import org.springframework.web.client.RestTemplate;
    
    /**
     * Created by longyi on 2018-04-01.
     */
    @RestController
    public class DcController {
    
    @Autowired
    LoadBalancerClient loadBalancerClient;
    
    @Autowired
    RestTemplate restTemplate;
    
    @GetMapping("/consumer")
    public String dc() {
    ServiceInstance serviceInstance = loadBalancerClient.choose("eureka-client");
    String url = "http://" + serviceInstance.getHost() + ":" + serviceInstance.getPort() + "/dc";
    System.out.println(url);
    return restTemplate.getForObject(url, String.class);
    }
    }
    

解析一下：

1.去注册中心找到服务eureka-client

2.获取服务的host和端口号，拼接api

3.通过restTemplate获取相关服务

### 测试过程

1.启动注册中心

2.启动eureka_client

3.启动消费端

可以打开http://localhost:1001/，随时观察服务：

[![QQ_20180401161611.png](https://s14.postimg.org/3rm5mase9/QQ_20180401161611.png)](https://postimg.org/image/80qvogvnh/)

此时调用：http://localhost:2101/consumer

刚开始会发现调用出错的（不知道是否其它人有遇到这种现象，报500错误），大约过了10秒钟以后，才顺利访问

Services: [eureka-consumer, eureka-client]

