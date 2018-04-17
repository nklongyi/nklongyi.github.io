---
layout:     post
title:      docker的docker-machine docker-swarm使用与安装
subtitle:   docker machine的安装与使用
date:       2018-04-16
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - docker
    
---

## 简介

Docker Machine 是 Docker 官方编排（Orchestration）项目之一，负责在多种平台上快速安装 Docker 环境。Docker Machine 项目基于 Go 语言实现！

## 安装

在linux系统上，直接下载对应的二进制包即可

    $ sudo curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-`uname -s`-`uname -m` > /usr/local/bin/docker-machine
    $ sudo chmod +x /usr/local/bin/docker-machine

完成后，查看版本信息：
    
    [root@localhost ~]# docker-machine -v
    docker-machine version 0.13.0, build 9ba6da9

## 使用

### 创建本地虚拟主机

    $ docker-machine create -d virtualbox test

你也可以在创建时加上如下参数，来配置主机或者主机上的 Docker。

- --engine-opt dns=114.114.114.114 配置 Docker 的默认 DNS

- --engine-registry-mirror https://registry.docker-cn.com 配置 Docker 的仓库镜像
 
- --virtualbox-memory 2048 配置主机内存
 
- --virtualbox-cpu-count 2 配置主机 CPU

更多参数请使用 `docker-machine create --driver virtualbox --help` 命令查看。

参考https://docs.docker.com/machine/drivers/generic/，来设置，根据不同的驱动类型来设置。

### 查看主机

    $ docker-machine ls

创建主机成功后，可以通过 `env` 命令来让后续操作对象都是目标主机。

    $ docker-machine env test

后续根据提示在命令行输入命令之后就可以操作 test 主机。

也可以通过 SSH 登录到主机。
    
    $ docker-machine ssh test
    
    docker@test:~$ docker --version
    Docker version 17.10.0-ce, build f4ffd25

连接到主机之后你就可以在其上使用 Docker 了


参考链接：https://yeasy.gitbooks.io/docker_practice/content/machine/usage.html

## docker之dockerswarm

Docker Swarm 是 Docker 官方三剑客项目之一，提供 Docker 容器集群服务，是 Docker 官方对容器云生态进行支持的核心方案。

使用它，用户可以将多个 Docker 主机封装为单个大型的虚拟 Docker 主机，快速打造一套容器云平台。

### swarm基本概念

Swarm 是使用 SwarmKit 构建的 Docker 引擎内置（原生）的集群管理和编排工具。


#### 节点

运行docker的主机可以创建一个集群，也可以主动加入一个已经存在的swarm集群，那么这个运行的docker主机就是一个swarm集群节点。

节点分为：管理节点（manager node）和工作节点（worker node）

管理节点用于 Swarm 集群的管理，docker swarm 命令基本只能在管理节点执行（节点退出集群命令 docker swarm leave 可以在工作节点执行）。一个 Swarm 集群可以有多个管理节点，但只有一个管理节点可以成为 leader，leader 通过 `raft` 协议实现。

>raft协议是分布式一致性的算法，paxos算法的一种变形，但是本质是一样的，都是类似于选举一致性算法。

**工作节点**（worker node）是任务执行节点，管理节点将服务 (service) 下发至工作节点执行。

**管理节点**默认也作为工作节点。你也可以通过配置让服务只运行在管理节点。

#### 服务和任务

任务 （Task）是 Swarm 中的最小的调度单位，目前来说就是一个单一的容器。

服务 （Services） 是指一组任务的集合，服务定义了任务的属性。服务有两种模式：

- replicated services 按照一定规则在各个工作节点上运行指定个数的任务。

- global services 每个工作节点上运行一个任务

两种模式通过 `docker service create` 的 `--mode` 参数指定。

#### 创建swarm集群

本次实验我采用的是虚拟机节点，ip地址为192.168.202.200，hostname改为manager.

    [root@manager ~]# ifconfig
    ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
    inet 192.168.202.200  netmask 255.255.255.0  broadcast 192.168.202.255
    inet6 fe80::1857:c1e7:fd9a:7ca  prefixlen 64  scopeid 0x20<link>
    inet6 fe80::4c96:e940:dc32:595b  prefixlen 64  scopeid 0x20<link>
    inet6 fe80::c0cd:f14b:3489:2ae0  prefixlen 64  scopeid 0x20<link>
    ether 00:0c:29:b5:fe:08  txqueuelen 1000  (Ethernet)
    RX packets 68  bytes 7186 (7.0 KiB)
    RX errors 0  dropped 0  overruns 0  frame 0
    TX packets 44  bytes 6757 (6.5 KiB)
    TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    
    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
    inet 127.0.0.1  netmask 255.0.0.0
    inet6 ::1  prefixlen 128  scopeid 0x10<host>
    loop  txqueuelen 1  (Local Loopback)
    RX packets 0  bytes 0 (0.0 B)
    RX errors 0  dropped 0  overruns 0  frame 0
    TX packets 0  bytes 0 (0.0 B)
    TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0


且该机器已经正确安装了docker，查看docker版本：
    
    [root@manager ~]# docker --version
    Docker version 18.03.0-ce, build 0520e24

执行docker swarm init 报错：

    [root@manager ~]# docker swarm init
    Error response from daemon: --live-restore daemon configuration is incompatible with swarm mode

根据官方的解释https://forums.docker.com/t/error-response-from-daemon-live-restore-daemon-configuration-is-incompatible-with-swarm-mode/28428/2：

编辑vi /etc/docker/daemon.json 将live-restort设置为false

然后重启docker 服务 ：service docker restart

    [root@manager ~]# docker swarm init
    Swarm initialized: current node (7qfdus18d7b4w27avexwxmdtj) is now a manager.
    
    To add a worker to this swarm, run the following command:
    
    docker swarm join --token SWMTKN-1-4fsxyfh6bxget5svop5nslrxjavur2ocb3ak832x4koi2e4p2c-ajdc6ajhv9nwh50ki2k5qgyvn 192.168.202.200:2377
    
    To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions


如果你的 Docker 主机有多个网卡，拥有多个 IP，必须使用 --advertise-addr 指定 IP.

查看集群节点：

    [root@manager ~]# docker node ls
    IDHOSTNAMESTATUS  AVAILABILITYMANAGER STATUS  ENGINE VERSION
    7qfdus18d7b4w27avexwxmdtj *   manager Ready   Active  Leader  18.03.0-ce
 

#### 部署服务

在主机上输入：docker service create --replicas 1 -p 80:80 --name nginx nginx:1.13.7-alpine

查看服务：

    [root@manager ~]# docker service ls
    ID  NAMEMODEREPLICASIMAGE PORTS
    o94x2qddcd7tnginx   replicated  1/1 nginx:1.13.7-alpine   *:80->80/tcp


在浏览器输入：http://192.168.202.200/，就可以看到nginx的服务界面。

查看服务详情：

    [root@manager ~]# docker service ps nginx
    ID  NAMEIMAGE NODEDESIRED STATE   CURRENT STATE   ERROR   PORTS
    tdafo01hiqoxnginx.1 nginx:1.13.7-alpine   manager Running Running about an hour ago  

查看服务日志：

    [root@manager ~]# docker service logs nginx
    nginx.1.tdafo01hiqox@manager| 10.255.0.2 - - [16/Apr/2018:16:55:32 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36" "-"
    nginx.1.tdafo01hiqox@manager| 2018/04/16 16:55:32 [error] 5#5: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 10.255.0.2, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "192.168.202.200", referrer: "http://192.168.202.200/"
    nginx.1.tdafo01hiqox@manager| 10.255.0.2 - - [16/Apr/2018:16:55:32 +0000] "GET /favicon.ico HTTP/1.1" 404 571 "http://192.168.202.200/" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36" "-"
    nginx.1.tdafo01hiqox@manager| 10.255.0.2 - - [16/Apr/2018:16:55:35 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36" "-"
    nginx.1.tdafo01hiqox@manager| 10.255.0.2 - - [16/Apr/2018:16:55:35 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36" "-"
    nginx.1.tdafo01hiqox@manager| 10.255.0.2 - - [16/Apr/2018:16:55:36 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36" "-"
    nginx.1.tdafo01hiqox@manager| 10.255.0.2 - - [16/Apr/2018:17:45:35 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36" "-"
    nginx.1.tdafo01hiqox@manager| 10.255.0.2 - - [16/Apr/2018:17:45:36 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36" "-" 

移除服务：

    docker service rm nginx 

页面就无法访问了。

#### docker machine创建虚拟节点

    [root@manager ~]# docker-machine create -d generic --generic-ip-address=192.168.10.1 manager1
    
因为我们用的是普通的linux部署docker，所以使用generic driver通用driver，其他driver可以参考：https://docs.docker.com/machine/drivers/

    [root@manager ~]# docker-machine create -d generic --generic-ip-address=192.168.10.1 manager1
    Running pre-create checks...
    Creating machine...
    (manager1) No SSH key specified. Assuming an existing key at the default location.
    Waiting for machine to be running, this may take a few minutes...


时间太长了，ctrl+c关闭了。









