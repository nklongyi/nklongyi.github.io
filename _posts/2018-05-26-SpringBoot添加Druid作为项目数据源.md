---
layout:     post
title:      SpringBoot添加Druid作为项目数据源
subtitle:   springboot的整合使用
date:       2018-05-26
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - springboot

---

>SpringBoot添加Druid作为项目数据源,Druid是一个关系型数据库连接池，它是阿里巴巴的一个开源项目。Druid支持所有JDBC兼容数据库，包括了Oracle、MySQL、PostgreSQL、SQL Server、H2等。
Druid在监控、可扩展性、稳定性和性能方面具有明显的优势。通过Druid提供的监控功能，可以实时观察数据库连接池和SQL查询的工作情况。使用Druid连接池在一定程度上可以提高数据访问效率。


## 准备工作 

### 添加最新的druid依赖

maven中央仓库查询：http://mvnrepository.com/artifact/com.alibaba/druid

添加最新：

    <!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
			<version>1.1.9</version>
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

  

## 开启Druid监控功能

### 配置文件
    spring.datasouce.type=com.alibaba.druid.pool.DruidDataSource
    # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
    spring.datasource.filters=stat,wall,log4j
    spring.datasource.maxActive=20
    spring.datasource.poolPreparedStatements=true
    spring.datasource.maxPoolPreparedStatementPerConnectionSize=20
    spring.datasource.minIdle=1
    # 配置一个连接在池中最小生存的时间，单位是毫秒
    spring.datasource.timeBetweenEvictionRunsMillis=60000
    spring.datasource.minEvictableIdleTimeMillis=300000
    spring.datasource.validationQuery=select 1 from dual
    spring.datasource.testWhileIdle=true
    # 通过connectProperties属性来打开mergeSql功能；慢SQL记录 
	spring.datasource.connectionProperties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000

### 增加druid的配置文件

在springdruidApplication同级目录下，新建DruidConfiguration文件：

	package com.nklongyi;

	import com.alibaba.druid.support.http.StatViewServlet;
	import com.alibaba.druid.support.http.WebStatFilter;
	import org.springframework.boot.web.servlet.FilterRegistrationBean;
	import org.springframework.boot.web.servlet.ServletRegistrationBean;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;

	/**
		 * Created by longyi on 2018-05-26.
	 */
	@Configuration
	public class DruidConfiguration {

    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean(new StatViewServlet(),"/druid/*");
        servletRegistrationBean.addInitParameter("allow","127.0.0.1");
        servletRegistrationBean.addInitParameter("deny","192.168.1.73");
        servletRegistrationBean.addInitParameter("longinUsername","druid");
        servletRegistrationBean.addInitParameter("longinPassword","123123");

        servletRegistrationBean.addInitParameter("resetEnable","false");

        return servletRegistrationBean;
    }

    @Bean
    public FilterRegistrationBean statFilter(){
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(new WebStatFilter());
        filterRegistrationBean.addUrlPatterns("/*");

        filterRegistrationBean.addInitParameter("exclusions","*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");

        return filterRegistrationBean;
    }


	}

注意，在以上配置中，如果没有配置servletRegistrationBean.addInitParameter("deny","192.168.1.73");则无法正常的启用druid，会有一些错误。

### 查看druid监控面板

[![CfbYCT.md.png](https://s1.ax1x.com/2018/05/26/CfbYCT.md.png)](https://imgchr.com/i/CfbYCT)









