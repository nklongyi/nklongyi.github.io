---
layout:     post
title:      SpringBoot使用FastJson
subtitle:   springboot的整合使用
date:       2018-05-26
author:     longyi
header-img: img/post-bg-ioses.jpg
catalog: true
tags:
    - springboot

---

>fastJson是阿里巴巴旗下的一个开源项目之一，顾名思义它专门用来做快速操作Json的序列化与反序列化的组件。它是目前json解析最快的开源组件没有之一！


## 准备工作 

### 添加最新fastJson依赖

maven中央仓库查询链接：http://mvnrepository.com/artifact/com.alibaba/fastjson

添加最新：

    	<!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>1.2.47</version>
		</dependency>

### 项目依赖如下：
已经包含有JPA SQL 的驱动包，以及web starter，创建项目的时候就已经勾选上。

	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.nklongyi</groupId>
	<artifactId>springdruid</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>springdruid</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.2.RELEASE</version>
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
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
			<version>1.1.9</version>
		</dependency>
			<!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>1.2.47</version>
		</dependency>

		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
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


	</project>

  

## 添加fastJson配置信息类

	package com.nklongyi;

	import com.alibaba.fastjson.serializer.SerializerFeature;
	import com.alibaba.fastjson.support.config.FastJsonConfig;
	import com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.http.MediaType;
	import org.springframework.http.converter.HttpMessageConverter;
	import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

	import java.util.ArrayList;
	import java.util.List;

	/**
 	* Created by longyi on 2018-05-26.
 	*/
	@Configuration
	public class FastJsonConfiguration  implements WebMvcConfigurer{

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {

        FastJsonHttpMessageConverter fastJsonHttpMessageConverter = new FastJsonHttpMessageConverter();
        FastJsonConfig fastJsonConfig = new FastJsonConfig();

        fastJsonConfig.setSerializerFeatures(
                SerializerFeature.WriteNullStringAsEmpty,
                SerializerFeature.WriteMapNullValue,
                SerializerFeature.DisableCircularReferenceDetect
        );
        //处理中文乱码问题
        List<MediaType> fastMediaType = new ArrayList<>();
        fastMediaType.add(MediaType.APPLICATION_JSON_UTF8);
        fastJsonHttpMessageConverter.setSupportedMediaTypes(fastMediaType);


        fastJsonHttpMessageConverter.setFastJsonConfig(fastJsonConfig);
        converters.add(fastJsonHttpMessageConverter);
    }
	}

## 遇到的问题

第一个就是中文乱码问题，加了处理中文的配置，恢复ok

	//处理中文乱码问题
        List<MediaType> fastMediaType = new ArrayList<>();
        fastMediaType.add(MediaType.APPLICATION_JSON_UTF8);
        fastJsonHttpMessageConverter.setSupportedMediaTypes(fastMediaType);

还有一个就是null转换为空串的问题，这个问题，按照WriteNullStringAsEmpty的做法，我这边是没有成功的。
很奇怪，我尝试了多次，发现还是不成功。













