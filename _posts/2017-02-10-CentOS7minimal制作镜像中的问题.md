---
layout:     post
title:      centos7minimal版本制作基本镜像的问题
subtitle:    纯净版镜像
date:       2018-02-10
author:     龙一
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - linux
---

> 制作centos镜像，方便集群复制，centos7minimal版本


## 前言

主要是用来做一些集群用的镜像，采用centos7 minimal

1.设置静态网络IP
  
   编辑/etc/sysconfig/network-scripts目录下网卡，将onboot改为yes,service network restart，然后ip addr查看是否网络有变更。
   
    TYPE=Ethernet
    PROXY_METHOD=none
    BROWSER_ONLY=no
    BOOTPROTO=static # 改dhcp为static
    IPADDR=192.168.202.128   
    GATEWAY=192.168.202.2
    NETMASK=255.255.255.0
    DNS1=223.5.5.5
    DNS2=223.6.6.6


上面的配置，就是根据自己的需要增加dns netmask ipaddr等，设置完，重启网络服务即可。

2.ping baidu不ok

  这种情况下，检查自己的gatway和dns是否配置正确即可。


3.最小化安装ifconfig找不到

     yum install net-tools 




