---
layout:     post
title:      Actuator监控端点
subtitle:   spring boot
date:       2018-03-27
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - springboot
    
---

## 简介

在Spring Boot的众多Starter POMs中有一个特殊的模块，它不同于其他模块那样大多用于开发业务功能或是连接一些其他外部资源。它完全是一个用于暴露自身信息的模块，所以很明显，它的主要作用是用于监控与管理。

对于实施微服务的中小团队来说，可以有效地减少监控系统在采集应用指标时的开发量。当然，它也并不是万能的，有时候我们也需要对其做一些简单的扩展来帮助我们实现自身系统个性化的监控需求。

## 初步实践

### 引入POM依赖

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>


### 运行主程序


[![20180327140154.png](https://s7.postimg.org/lrh3q812z/20180327140154.png)](https://postimg.org/image/6ir6cg7ef/)


在浏览器输入：http://localhost:8080/health ，显示：status：up

### 关闭应用，增加配置

在application.properties中新增：management.security.enabled=false

重启应用：输入http://localhost:8080/health：
    
    {
    "status": "UP",
    "diskSpace": {
    "status": "UP",
    "total": 85899341824,
    "free": 15316606976,
    "threshold": 10485760
    }
    }
    

## 端点职能划分

应用配置类：获取应用程序中加载的应用配置、环境变量、自动化配置报告等与Spring Boot应用密切相关的配置类信息。

度量指标类：获取应用程序运行过程中用于监控的度量指标，比如：内存信息、线程池信息、HTTP请求统计等。

操作控制类：提供了对应用的关闭等操作类功能。

### 应用配置类

Spring Boot为了改善传统Spring应用繁杂的配置内容，采用了包扫描和自动化配置的机制来加载原本集中于xml文件中的各项内容。虽然这样的做法，让我们的代码变得非常简洁，但是整个应用的实例创建和依赖关系等信息都被离散到了各个配置类的注解上，这使得我们分析整个应用中资源和实例的各种关系变得非常的困难。而这类端点就可以帮助我们轻松的获取一系列关于Spring 应用配置内容的详细报告，比如：自动化配置的报告、Bean创建的报告、环境属性的报告等。

**/autoconfig：**该端点用来获取应用的自动化配置报告，其中包括所有自动化配置的候选项。同时还列出了每个候选项自动化配置的各个先决条件是否满足。所以，该端点可以帮助我们方便的找到一些自动化配置为什么没有生效的具体原因。该报告内容将自动化配置内容分为两部分：

positiveMatches中返回的是条件匹配成功的自动化配置

negativeMatches中返回的是条件匹配不成功的自动化配置

[![20180327142128.png](https://s7.postimg.org/dd1h2cae3/20180327142128.png)](https://postimg.org/image/qtyfl7kpj/)


每个自动化配置候选项中都有一系列的条件，如果没有匹配成功，就可以从这里查看没有自动化加载成功的原因。

**/beans：**该端点用来获取应用上下文中创建的所有Bean

获取所有bean的信息

**/configprops：**该端点用来获取应用中配置的属性信息报告。prefix属性代表了属性的配置前缀，properties代表了各个属性的名称和值。所以，我们可以通过该报告来看到各个属性的配置路径，比如我们要关闭该端点，就可以通过使用endpoints.configprops.enabled=false来完成设置。

[![20180327143257.png](https://s7.postimg.org/hng4xma8r/20180327143257.png)](https://postimg.org/image/awzno6n2v/)

**/env：**该端点与/configprops不同，它用来获取应用所有可用的环境属性报告。包括：环境变量、JVM属性、应用的配置配置、命令行中的参数。

[![20180327143459.png](https://s7.postimg.org/fvn62te23/20180327143459.png)](https://postimg.org/image/vh4hmrq07/)

ps：另外，为了配置属性的安全，对于一些类似密码等敏感信息，该端点都会进行隐私保护，但是我们需要让属性名中包含：password、secret、key这些关键词，这样该端点在返回它们的时候会使用*来替代实际的属性值。

**/mappings：**该端点用来返回所有Spring MVC的控制器映射关系报告。

[![20180327143651.png](https://s7.postimg.org/6b3jg447f/20180327143651.png)](https://postimg.org/image/debevq9mv/)

**/info：**该端点用来返回一些应用自定义的信息。默认情况下，该端点只会返回一个空的json内容。我们可以在application.properties配置文件中通过info前缀来设置一些属性。

    info.app.name=spring-boot-hello
    info.app.version=v1.0.0


### 度量指标类

度量指标类端点提供的报告内容则是动态变化的，这些端点提供了应用程序在运行过程中的一些快照信息，比如：内存使用情况、HTTP请求统计、外部资源指标等。这些端点对于我们构建微服务架构中的监控系统非常有帮助，由于Spring Boot应用自身实现了这些端点，所以我们可以很方便地利用它们来收集我们想要的信息，以制定出各种自动化策略。

>应用配置类端点所提供的信息报告在应用启动的时候都已经基本确定了其返回内容，可以说是一个静态报告

**/metrics：**该端点用来返回当前应用的各类重要度量指标，比如：内存信息、线程信息、垃圾回收信息等

[![20180327143651.png](https://s7.postimg.org/ncwdhwdu3/20180327143651.png)](https://postimg.org/image/7rf1xy1vr/)

还可以通过/metrics/{name}接口来更细粒度的获取度量信息

[![20180327144551.png](https://s7.postimg.org/ychktsmcb/20180327144551.png)](https://postimg.org/image/jgj1m7axj/)

**/health:**可以自定义actutor中未定义的自动化检测，模块中自带了用于检测磁盘的DiskSpaceHealthIndicator、检测DataSource连接是否可用的DataSourceHealthIndicator等，但是某些start poms还未封装，此时通过自动以实现org.springframework.boot.actuate.health.HealthIndicator接口来做自定义检测。

比如RocketMQ的检测器类：
    
    @Component
    public class RocketMQHealthIndicator implements HealthIndicator {
    
    @Override
    public Health health() {
    int errorCode = check();
    if (errorCode != 0) {
      return Health.down().withDetail("Error Code", errorCode).build();
    }
    return Health.up().build();
    }
    
      	private int check() {
     	// 对监控对象的检测操作
      	}
    }

重写health()函数来实现健康检查，返回的Heath对象中，共有两项内容，一个是状态信息，除了该示例中的UP与DOWN之外，还有UNKNOWN和OUT_OF_SERVICE，可以根据需要来实现返回；还有一个详细信息，采用Map的方式存储，在这里通过withDetail函数，注入了一个Error Code信息，我们也可以填入一下其他信息，比如，检测对象的IP地址、端口等。重新启动应用，并访问/health接口，我们在返回的JSON字符串中，将会包含了如下信息：
    
    "rocketMQ": {
      "status": "UP"
    }

**/dump：**该端点用来暴露程序运行中的线程信息。它使用java.lang.management.ThreadMXBean的dumpAllThreads方法来返回所有含有同步信息的活动线程详情。

**/trace：**该端点用来返回基本的HTTP跟踪信息。默认情况下，跟踪信息的存储采用org.springframework.boot.actuate.trace.InMemoryTraceRepository实现的内存方式，始终保留最近的100条请求记录

[![20180327143651.png](https://s7.postimg.org/c1tptn1uz/20180327143651.png)](https://postimg.org/image/7sozrgylj/)


### 操作控制类

应用配置类和度量类都具有反映app运行动态和静态的信息反馈的能力，而操作控制类则具有更强的控制能力，所以如果要使用必须要开启。

在翟永超的博客中，对方说只要设置：
endpoints.shutdown.enabled=true

**然后对shutdown urlpost即可关闭，但是实际不ok。**

我的方案是：

 1.增加endpoints.shutdown.sensitive=false属性配置

 2.重启应用，开启postman

 3.用postman发起post请求：

 [![20180327143651.png](https://s7.postimg.org/6zhfwabuj/20180327143651.png)](https://postimg.org/image/g79ocziwn/)

然后控制台：

[![20180327154227.png](https://s7.postimg.org/81rmeycp7/20180327154227.png)](https://postimg.org/image/synujmapz/)



正确关闭，done！

