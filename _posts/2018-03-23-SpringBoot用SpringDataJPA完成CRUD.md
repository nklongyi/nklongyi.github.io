---
layout:     post
title:      springboot使用SpringDataJPA完成CRUD
subtitle:   springboot的整合使用
date:       2018-05-26
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - springboot


>使用springData JPA完成数据库的CRUD操作


## 准备工作 

IDEA创建springinitial项目，勾选web jpa mysql 三个选项，maven配置如下所示：

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    	<modelVersion>4.0.0</modelVersion>
    
    	<groupId>com.nklongyi</groupId>
    	<artifactId>springbootjpa</artifactId>
    	<version>0.0.1-SNAPSHOT</version>
    	<packaging>jar</packaging>
    
    	<name>springbootjpa</name>
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

## 配置数据源

在application.properties中：
    
    spring.datasource.url=jdbc:mysql://192.168.202.130:3306/test?characterEncoding=utf8
    spring.datasource.driver-class-name=com.mysql.jdbc.Driver
    spring.datasource.username=root
    spring.datasource.password=xxxx
    
    spring.jpa.database=mysql
    spring.jpa.show-sql=true
    spring.jpa.hibernate.ddl-auto=create-drop

## 实例类

新建UserEntity
    
    package com.nklongyi.Entity;
    
    
    
    import javax.persistence.*;
    import java.io.Serializable;
    
    /**
     * Created by longyi on 2018-05-26.
     */
    @Entity
    @Table(name="t_user")
    public class UserEntity implements Serializable{
    @Id
    @GeneratedValue
    @Column(name = "t_id")
    private long id;
    @Column(name="t_name")
    private String name;
    @Column(name = "t_age")
    private int age;
    @Column(name = "t_address")
    private String address;
    
    public long getId() {
    return id;
    }
    
    public void setId(long id) {
    this.id = id;
    }
    
    public String getName() {
    return name;
    }
    
    public void setName(String name) {
    this.name = name;
    }
    
    public int getAge() {
    return age;
    }
    
    public void setAge(int age) {
    this.age = age;
    }
    
    public String getAddress() {
    return address;
    }
    
    public void setAddress(String address) {
    this.address = address;
    }
    
    }

## 创建JPA

创建UserJPA接口并且继承SpringDataJPA内的接口作为父类，如所示：

    public interface UserJPA extends JpaRepository<UserEntity,Long>,JpaSpecificationExecutor<UserEntity>,Serializable {
    }
    
UserJPA继承了JpaRepository接口（SpringDataJPA提供的简单数据操作接口）、JpaSpecificationExecutor（SpringDataJPA提供的复杂查询接口）、Serializable（序列化接口）

## 创建Controller

将JPA注入到Controller中

    package com.nklongyi.controller;
    
    import com.nklongyi.Entity.UserEntity;
    import com.nklongyi.Jpa.UserJPA;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.bind.annotation.RestController;
    
    import java.util.List;
    
    /**
     * Created by longyi on 2018-05-26.
     */
    @RestController
    @RequestMapping(value = "/user")
    public class UserController {
    
    @Autowired
    private UserJPA userJPA;
    
    @RequestMapping(value = "/list",method = RequestMethod.GET)
    public List<UserEntity> list(){
    return userJPA.findAll();
    }
    
    @RequestMapping(value = "/save",method = RequestMethod.GET)
    public UserEntity save(UserEntity userEntity){
    return userJPA.save(userEntity);
    }
    
    @RequestMapping(value = "/delete",method = RequestMethod.GET)
    public List<UserEntity> delete(Long id){
    userJPA.deleteById(id);
    return userJPA.findAll();
    }
    
    }

## 实际运行项目

run项目，打开链接：http://127.0.0.1:8080/user/list

返回的为[]

### 添加用户

打开链接：http://127.0.0.1:8080/user/save?name=admin&age=22&address=jinan

[![20180526114820.png](https://s9.postimg.cc/q94z82py7/20180526114820.png)](https://postimg.cc/image/5c8r3erx7/)

### 查看用户列表

[![20180526114820.png](https://s9.postimg.cc/wzlghlkun/20180526114820.png)](https://postimg.cc/image/wmu2bf2kr/)

### 删除用户

[![20180526114820.png](https://s9.postimg.cc/nrt814yfj/20180526114820.png)](https://postimg.cc/image/49ykl71hn/)



至此，我们已经完成JPA的简单案例。关于JPA板块，由于很多操作是不可控的，因此我们在很多情况下并不使用，当然了对于一些对时效性要求比较高 业务场景简单的业务使用JPA可以达到事半功倍的效果。






