---
layout:     post
title:      Spring Cloud构建微服务架构：分布式配置中心(2)
subtitle:   Spring Cloud Config的整合使用
date:       2018-04-02
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - springcloud
    
---

##高可用问题

### 传统方式


通常在生产环境，Config Server与服务注册中心一样，我们也需要将其扩展为高可用的集群。在之前实现的config-server基础上来实现高可用非常简单，不需要我们为这些服务端做任何额外的配置，只需要遵守一个配置规则：将所有的Config Server都指向同一个Git仓库，这样所有的配置内容就通过统一的共享文件系统来维护，而客户端在指定Config Server位置时，只要配置Config Server外的均衡负载即可。


### 注册为服务的方式

虽然通过服务端负载均衡已经能够实现，但是作为架构内的配置管理，本身其实也是可以看作架构中的一个微服务。所以，另外一种方式更为简单的方法就是把config-server也注册为服务，这样所有客户端就能以服务的方式进行访问。通过这种方法，只需要启动多个指向同一Git仓库位置的config-server就能实现高可用了。


#### 1.配置config server

1.在原有项目中添加依赖，用来注册到eureka的注册中心：

    <dependency>
    		<groupId>org.springframework.cloud</groupId>
    		<artifactId>spring-cloud-starter-eureka</artifactId>
    	</dependency>

2.在配置application.properties中添加：

    spring.application.name=config-server
    server.port=1201
    # 配置服务注册中心
    eureka.client.serviceUrl.defaultZone=http://localhost:1001/eureka/
    # git仓库配置
    spring.cloud.config.server.git.uri=https://github.com/nklongyi/springcloudconfigdemo.git

直接去掉yml中的内容

3.增加主类注释

    package com.nklongyi;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
    import org.springframework.cloud.config.server.EnableConfigServer;
    
    @EnableDiscoveryClient
    @EnableConfigServer
    @SpringBootApplication
    public class ConfigServerGitApplication {
    
    	public static void main(String[] args) {
    		SpringApplication.run(ConfigServerGitApplication.class, args);
    	}
    }
    
4.启动eureka-server，然后启动config-server。

打开http://localhost:1001/，可以查看到对应的实例已经注册到服务中心。


#### 2.配置config-client

1.增加2个依赖

    <dependency>
    			<groupId>org.springframework.cloud</groupId>
    			<artifactId>spring-cloud-starter-eureka</artifactId>
    		</dependency>
    <dependency>
    			<groupId>org.springframework.boot</groupId>
    			<artifactId>spring-boot-starter-actuator </artifactId>
    		</dependency>
    
2**.新建bootstrap.properties.这里是bootstrap，不是application，否则启动的时候直接去localhost:8888端口去fetch config信息。**

    # 配置服务注册中心
    eureka.client.serviceUrl.defaultZone=http://localhost:1001/eureka/
    spring.application.name=configclient
    server.port=2001
    
    spring.cloud.config.discovery.enabled=true
    spring.cloud.config.discovery.serviceId=config-server
    spring.cloud.config.profile=dev
    management.security.enabled= false

最后一个securty的问题，再后面refresh的时候就知道了，没有这个false，post refresh的时候返回的json包会提示没有权限（未授权）。


3.在git仓库下新建configclient-dev.properties文件，然后内容为：

    from=git-dev-1.0

4.在主类上增加注释EnableDiscoveryClient

5.增加一个testcontroller，内容如下：

    package com.nklongyi.controller;
    
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.cloud.context.config.annotation.RefreshScope;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;
    
    /**
     * Created by longyi on 2018-04-02.
     */
    @RefreshScope
    @RestController
    public class TestController {
    @Value("${from}")
    private String from;
    
    @RequestMapping("/from")
    public String from() {
    return this.from;
    }
    }
    

作用：获取配置文件中的from 这个key的内容，类似于在本地properites获取内容一样，非常精巧。

#### 最后启动该项目

启动成功后，先去注册中心看看，然后再浏览器输入：http://localhost:2001/from

能够正确返回内容。


## 更新配置

将git仓库的from内容变更，然后我们再刷新http://localhost:2001/from，发现还是没有变更。

于是我们再postman中做一个post操作：

[![20180327154227.png](https://s7.postimg.org/q0i74y4rf/20180327154227.png)](https://postimg.org/image/mgw9f521j/)

此时我们再访问http://localhost:2001/from，发现就可以看到最新的变更。

>功能还可以同Git仓库的Web Hook功能进行关联，当有Git提交变化时，就给对应的配置主机发送/refresh请求来实现配置信息的实时更新。但是，当我们的系统发展壮大之后，维护这样的刷新清单也将成为一个非常大的负担，而且很容易犯错，那么有什么办法可以解决这个复杂度呢？后续我们将继续介绍如何通过Spring Cloud Bus来实现以消息总线的方式进行通知配置信息的变化，完成集群上的自动化更新。