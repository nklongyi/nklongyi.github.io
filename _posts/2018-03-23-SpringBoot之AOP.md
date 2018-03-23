---
layout:     post
title:      SpringBoot之AOP
subtitle:   springboot的整合使用
date:       2018-03-23
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - springboot
    
---

## 简介

AOP （aspect orientated program），面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。无侵入，是它显著的一个特点。打开数据库/关闭数据库，事务操作，记录日志、权限校验等等这些功能，不关乎具体的业务逻辑，如果写死在程序中就破坏了程序代码的逻辑，而基于AOP的形式无侵入，使得业务逻辑与该功能分离，良好解耦。


## springBoot实践

对一个web url请求进行拦截，日志记录相关的请求、处理类等信息，待处理完毕后返回的时候再进行拦截处理。

思路：

1.定义一个切面类

2.定义切点

3.围绕切点，织入相关操作（before after around AfterThrowing等）

**实现AOP的切面主要有以下几个要素：**

- 使用@Aspect注解将一个java类定义为切面类

- 使用@Pointcut定义一个切入点，可以是一个规则表达式，比如下例中某个package下的所有函数，也可以是一个注解等。
   根据需要在切入点不同位置的切入内容

- 使用@Before在切入点开始处切入内容

- 使用@After在切入点结尾处切入内容

- 使用@AfterReturning在切入点return内容之后切入内容（可以用来对处理返回值做一些加工处理）

- 使用@Around在切入点前后切入内容，并自己控制何时执行切入点自身的内容

- 使用@AfterThrowing用来处理当切入内容部分抛出异常之后的处理逻辑

**理论上可以理解为：定义一个切面，引入切点，围绕切点织入相关的操作。** 我脑海中对这一块的理解犹如圆的切线，切线就是这个切面，然后再这个切点处织入相关的操作就是我们的before之类的动作了。



### 1.引入依赖


	<dependency>
	    <groupId>org.springframework.boot</groupId>
	    <artifactId>spring-boot-starter-web</artifactId>
	</dependency>
	<dependency>
	    <groupId>org.springframework.boot</groupId>
	    <artifactId>spring-boot-starter-aop</artifactId>
	</dependency>

### 2.定义一个处理类

    package com.nklongyi.controller;
    
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.bind.annotation.RequestParam;
    import org.springframework.web.bind.annotation.RestController;
    
    /**
     * Created by longyi on 2018-03-23.
     */
    @RestController
    public class HelloController {
    
    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String hello(@RequestParam String name){
    return "hello"+name;
    }
    }

### 3.定义切面类

    package com.nklongyi.aop;
    
    import org.apache.log4j.Logger;
    import org.aspectj.lang.JoinPoint;
    import org.aspectj.lang.annotation.AfterReturning;
    import org.aspectj.lang.annotation.Aspect;
    import org.aspectj.lang.annotation.Before;
    import org.aspectj.lang.annotation.Pointcut;
    import org.springframework.stereotype.Component;
    import org.springframework.web.context.request.RequestContextHolder;
    import org.springframework.web.context.request.ServletRequestAttributes;
    
    import javax.servlet.http.HttpServletRequest;
    import java.util.Arrays;
    
    /**
     * Created by longyi on 2018-03-23.
     */
    @Aspect
    @Component
    public class WebLogAspect {
    //log4j
    private Logger logger = Logger.getLogger(getClass());
    
    
    @Pointcut("execution(public * com.nklongyi.controller..*.*(..))")
    public void weblog(){}
    
    @Before("weblog()")
    public void doBefore(JoinPoint joinPoint){
    // 接收到请求，记录请求内容
    ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
    HttpServletRequest request = attributes.getRequest();
    
    // 记录下请求内容
    logger.info("URL : " + request.getRequestURL().toString());
    logger.info("HTTP_METHOD : " + request.getMethod());
    logger.info("IP : " + request.getRemoteAddr());
    logger.info("CLASS_METHOD : " + joinPoint.getSignature().getDeclaringTypeName() + "." + joinPoint.getSignature().getName());
    logger.info("ARGS : " + Arrays.toString(joinPoint.getArgs()));
    
    }
    
    @AfterReturning(returning = "ret", pointcut = "weblog()")
    public void doAfterReturning(Object ret) throws Throwable {
    // 处理完请求，返回内容
    logger.info("RESPONSE : " + ret);
    }
    
    }


启动项目，然后再浏览器输入：http://localhost:8080/hello?name=didi，此时console打出如下信息：

    2018-03-23 17:12:47.378  INFO 7732 --- [   main] o.s.j.e.a.AnnotationMBeanExporter: Registering beans for JMX exposure on startup
    2018-03-23 17:12:47.420  INFO 7732 --- [   main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
    2018-03-23 17:12:47.423  INFO 7732 --- [   main] com.nklongyi.Springboot05Application : Started Springboot05Application in 2.263 seconds (JVM running for 2.598)
    2018-03-23 17:14:24.897  INFO 7732 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]   : Initializing Spring FrameworkServlet 'dispatcherServlet'
    2018-03-23 17:14:24.897  INFO 7732 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet: FrameworkServlet 'dispatcherServlet': initialization started
    2018-03-23 17:14:24.915  INFO 7732 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet: FrameworkServlet 'dispatcherServlet': initialization completed in 18 ms
    2018-03-23 17:14:24.938  INFO 7732 --- [nio-8080-exec-1] com.nklongyi.aop.WebLogAspect: URL : http://localhost:8080/hello
    2018-03-23 17:14:24.938  INFO 7732 --- [nio-8080-exec-1] com.nklongyi.aop.WebLogAspect: HTTP_METHOD : GET
    2018-03-23 17:14:24.939  INFO 7732 --- [nio-8080-exec-1] com.nklongyi.aop.WebLogAspect: IP : 0:0:0:0:0:0:0:1
    2018-03-23 17:14:24.940  INFO 7732 --- [nio-8080-exec-1] com.nklongyi.aop.WebLogAspect: CLASS_METHOD : com.nklongyi.controller.HelloController.hello
    2018-03-23 17:14:24.940  INFO 7732 --- [nio-8080-exec-1] com.nklongyi.aop.WebLogAspect: ARGS : [didi]
    2018-03-23 17:14:24.942  INFO 7732 --- [nio-8080-exec-1] com.nklongyi.aop.WebLogAspect: RESPONSE : hellodidi



### AOP切面的优先级

由于通过AOP实现，程序得到了很好的解耦，但是也会带来一些问题，比如：我们可能会对Web层做多个切面，校验用户，校验头信息等等，这个时候经常会碰到切面的处理顺序问题。

所以，我们需要定义每个切面的优先级，我们需要@Order(i)注解来标识切面的优先级。i的值越小，优先级越高。假设我们还有一个切面是CheckNameAspect用来校验name必须为didi，我们为其设置**`@Order(10)`**，而上文中WebLogAspect设置为**@Order(5)**，所以WebLogAspect有更高的优先级，这个时候执行顺序是这样的：

- 在@Before中优先执行@Order(5)的内容，再执行@Order(10)的内容

- 在@After和@AfterReturning中优先执行@Order(10)的内容，再执行@Order(5)的内容

所以我们可以这样子总结：

**在切入点前的操作，按order的值由小到大执行**

**在切入点后的操作，按order的值由大到小执行**