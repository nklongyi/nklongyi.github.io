---
layout:     post
title:      Spring Cloud构建微服务架构：服务网关（基础）
subtitle:   服务网关spring cloud zuul
date:       2018-04-02
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - springcloud
    
---

## 简介

服务网关是微服务架构中一个不可或缺的部分。通过服务网关统一向外系统提供REST API的过程中，除了具备服务路由、均衡负载功能之外，它还具备了权限控制等功能。Spring Cloud Netflix中的Zuul就担任了这样的一个角色，为微服务架构提供了前门保护的作用，同时将权限控制这些较重的非业务逻辑内容迁移到服务路由层面，使得服务集群主体能够具备更高的可复用性和可测试性。

## 基础实践

将之前的eureka server、eureka-client、eureka-consumer全部启动，然后去注册中心确认已经启动完毕。

下面来新建网关项目，新建项目api-gateway

### 添加依赖

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    	<modelVersion>4.0.0</modelVersion>
    
    	<groupId>com.nklongyi</groupId>
    	<artifactId>api-gateway</artifactId>
    	<version>0.0.1-SNAPSHOT</version>
    	<packaging>jar</packaging>
    
    	<name>api-gateway</name>
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
    			<artifactId>spring-boot-starter</artifactId>
    		</dependency>
    		<dependency>
    			<groupId>org.springframework.cloud</groupId>
    			<artifactId>spring-cloud-starter-zuul</artifactId>
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
 
### 应用主类添加注释EnableZuulProxy

    package com.nklongyi;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.netflix.zuul.EnableZuulProxy;
    
    @EnableZuulProxy
    @SpringBootApplication
    public class ApiGatewayApplication {
    
    	public static void main(String[] args) {
    		SpringApplication.run(ApiGatewayApplication.class, args);
    	}
    }
    
### 添加项目配置
    
    spring.application.name=api-gateway
    server.port=1101
    
    eureka.client.service-url.defaultZone=http://localhost:1001/eureka/
   
至此为止，项目就可以开启了。。


启动该应用，一个默认的服务网关就构建完毕了。由于Spring Cloud Zuul在整合了Eureka之后，具备默认的服务路由功能，即：当我们这里构建的api-gateway应用启动并注册到eureka之后，服务网关会发现上面我们启动的两个服务eureka-client和eureka-consumer，这时候Zuul就会创建两个路由规则。每个路由规则都包含两部分，一部分是外部请求的匹配规则，另一部分是路由的服务ID。 

访问以下地址：http://localhost:1101/eureka-client/dc

Services: [eureka-consumer-ribbons, eureka-client, eureka-consumer, api-gateway] 

访问：http://localhost:1101/eureka-consumer/consumer

Services: [eureka-consumer-ribbons, eureka-client, eureka-consumer, api-gateway]

可以发现智能网关已经替我们做好了路由，智能路由。

## 传统路由配置

所谓的传统路由配置方式就是在**不依赖于服务发现机制**的情况下，通过在配置文件中具体指定每个路由表达式与服务实例的映射关系来实现API网关对外部请求的路由。

没有Eureka和Consul的服务治理框架帮助的时候，我们需要根据服务实例的数量采用不同方式的配置来实现路由规则：

**单实例配置：**
    
    zuul.routes.user-service.path=/user-service/**
    zuul.routes.user-service.url=http://localhost:8080/

该配置实现了对符合/user-service/**规则的请求路径转发到http://localhost:8080/地址的路由规则，比如，当有一个请求http://localhost:1101/user-service/hello被发送到API网关上，由于/user-service/hello能够被上述配置的path规则匹配，所以API网关会转发请求到http://localhost:8080/hello地址。

**多实例配置：**
    
    zuul.routes.user-service.path=/user-service/**
    zuul.routes.user-service.serviceId=user-service
    
    ribbon.eureka.enabled=false
    user-service.ribbon.listOfServers=http://localhost:8080/,http://localhost:8081/

存在多个实例，API网关在进行路由转发时需要实现负载均衡策略，于是这里还需要Spring Cloud Ribbon的配合。由于在Spring Cloud Zuul中自带了对Ribbon的依赖，所以我们只需要做一些配置即可，比如上面示例中关于Ribbon的各个配置，它们的具体作用如下。

不论是单实例还是多实例的配置方式，我们都需要为每一对映射关系指定一个名称，也就是上面配置中的<route>，每一个<route>就对应了一条路由规则。每条路由规则都需要通过path属性来定义一个用来匹配客户端请求的路径表达式，并通过url或serviceId属性来指定请求表达式映射具体实例地址或服务名。

在Spring Cloud Netflix中，Zuul巧妙的整合了Eureka来实现面向服务的路由。实际上，我们可以直接将API网关也看做是Eureka服务治理下的一个普通微服务应用。它除了会将自己注册到Eureka服务注册中心上之外，也会从注册中心获取所有服务以及它们的实例清单。所以，在Eureka的帮助下，API网关服务本身就已经维护了系统中所有serviceId与实例地址的映射关系。当有外部请求到达API网关的时候，根据请求的URL路径找到最佳匹配的path规则，API网关就可以知道要将该请求路由到哪个具体的serviceId上去。由于在API网关中已经知道serviceId对应服务实例的地址清单，那么只需要通过Ribbon的负载均衡策略，直接在这些清单中选择一个具体的实例进行转发就能完成路由工作了。








