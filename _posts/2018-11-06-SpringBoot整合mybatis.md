---
layout:     post
title:      SpringBoot整合mybatis
subtitle:   springboot2.0.6的整合使用
date:       2018-11-06
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - springboot

---

>整合mybatis是SSM框架中的主要部分，此处主要是简单记录一下整合流程，不深入。最近研究spring源码，仅仅作为了解整合。

springboot推崇的是注解方式，建议最少化xml配置文件来完成整个项目的配置，跟Grails一样遵守“约定优于配置”的规则，这里面有一个悖论，原来我们提倡配置与代码实现是松耦合的，现在这种注解的方式是将配置与代码进行了紧耦合，那么孰好孰坏？单纯从技术角度而言，松耦合自然很好，但是大量的xml文件让整个维护工作混乱不堪。实际工作中，配置xml都是由开发来做，那么既然是开发来做，为何不写在代码里？这似乎又很有道理。


## 准备工作 

### 添加mybatis依赖包

maven中央仓库查询：http://mvnrepository.com/artifact/com.alibaba/druid

添加最新：

   		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>1.3.2</version>
		</dependency>

### 项目依赖如下：
偷懒，将pom文件直接复制过来，添加了以上的mytabis的包

	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.nklongyi</groupId>
	<artifactId>onlinetestingsystem</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>onlinetestingsystem</name>
	<description>OnlineTesting for students to Learn Olypic math</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.6.RELEASE</version>
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
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-aop</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>com.ibeetl</groupId>
			<artifactId>beetl-framework-starter</artifactId>
			<version>1.1.63.RELEASE</version>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
        <!--HikariCP 连接池-->
        <dependency>
            <groupId>com.zaxxer</groupId>
            <artifactId>HikariCP</artifactId>
            <version>2.6.l</version>
        </dependency>
		<!-- https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter -->
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>1.3.2</version>
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

  

### 数据源的配置

	spring.datasource.url=jdbc:mysql://192.168.202.130:3306/test?characterEncoding=utf8
	spring.datasource.username=root
	spring.datasource.password=xxxxx
	spring.datasource.driver-class-name=com.mysql.jdbc.Driver

### 新建DAO包

在DAO包中，新建一个mapper

    package com.nklongyi.DAO;

	import com.nklongyi.Domain.PersonDO;
	import org.apache.ibatis.annotations.*;

	import java.util.List;

	/**
 		* Created by longyi on 2018-11-08.
 	*/
	@Mapper
	public interface PersonMapper {

    @Insert("insert into person(name,age) values(#{name},#{age})")
    @Options(useGeneratedKeys = true,keyColumn = "id",keyProperty = "id")
    void insert(PersonDO personDO);

    @Update("update person set name=#{name},age=#{age} where id = #{id}")
    Long update(PersonDO personDO);

    @Delete("delete from person where id = #{id}")
    Long delete(@Param("id") Long id);

    @Select("select id,name,age from person")
    List<PersonDO> selectAll();

    @Select("select id,name,age from person where id=#{id}")
    PersonDO selectById(@Param("id") Long id);
	}

再写一个restcontroller做测试：

    
	package com.nklongyi.Controller;

import com.nklongyi.DAO.PersonMapper;
import com.nklongyi.Domain.PersonDO;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

/**
 * Created by longyi on 2018-11-08.
 */
@RestController
public class TestControlleer {

    @Autowired
    private PersonMapper personMapper;

    @RequestMapping("/save")
    public Integer save() {
        PersonDO personDO = new PersonDO();
        personDO.setId(1);
        personDO.setName("张三");
        personDO.setAge(18);
        personMapper.insert(personDO);
        return personDO.getId();
    }

    @RequestMapping("/update")
    public Long update() {
        PersonDO personDO = new PersonDO();
        personDO.setId(2);
        personDO.setName("旺旺");
        personDO.setAge(12);
        return personMapper.update(personDO);
    }

    @RequestMapping("/delete")
    public Long delete() {
        return personMapper.delete(11L);
    }

    @RequestMapping("/selectById")
    public PersonDO selectById() {
        return personMapper.selectById(2L);
    }

    @RequestMapping("/selectAll")
    public List<PersonDO> selectAll() {
        return personMapper.selectAll();
    }


	}

实际做的过程中，确实可以写入数据、select数据出来也能通过restfull的形式返回。这里有个事情很奇怪，就是idea对于mapper的注入，会提示无法解析，但是runtime的时候是没有出错的，有点奇怪，后续我再看看。

这里面跟JPA的内容有个很大的区别，就是数据库的自由度相对HIbernate 要大很多。

>下次我在一些生产项目做一些尝试，目前只是实验性的了解。mybatis里面有很多设计模式的东西，暂时没有时间看源码，先抓紧时间看看spring的源码先。














