---
layout:     post
title:      springcloud 微服务架构：分布式服务跟踪整合zipkinserver
subtitle:   基于消息中间件收集请求信息
date:       2018-04-06
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - springcloud
    
---

>在上一个文章中，我们再每个服务应用中通过http协议发送请求日志信息到zipkin serve，Spring Cloud Sleuth在整合Zipkin时还实现了通过消息中间件来对跟踪信息进行异步收集的封装。

## 简介

通过结合Spring Cloud Stream，我们可以非常轻松的让应用客户端将跟踪信息输出到消息中间件上，同时Zipkin服务端从消息中间件上异步地消费这些跟踪信息








