---
layout:     post
title:      spring boot之微服务实战
subtitle:   spring cloud
date:       2018-03-20
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - springcloud
    
---

## 简介

spring cloud的微服务是基于spring boot，记录spring boot以及spring cloud相关组件的实例。

开发工具：idea

## 安装步骤

1.微服务应用的管理

引入actuator组件

    <dependency>
    			<groupId>org.springframework.boot</groupId>
    			<artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    
	
2.服务的治理 spring cloud Eureka


