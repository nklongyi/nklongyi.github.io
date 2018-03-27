---
layout:     post
title:      Centos7 install node 并且设置淘宝镜像
subtitle:   node npm 
date:       2018-03-20
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - node
    
---

## 安装

1.进入官网：https://nodejs.org/en/download/

[![20180327103029.png](https://s7.postimg.org/4cnoqimmj/20180327103029.png)](https://postimg.org/image/r1cvq3407/)






用F12查看对应的地址，在terminal上：wget xxxx


2.命令行解压：`tar -xJf node-v8.9.1-linux-x64.tar.xz`

这里注意，非tar.gz文件，而是xz。

3.我将解压后的node包移动到了/usr/local/目录下，便于统一管理
mv node /usr/local/node-v8

4.编写环境变量文件

vi /etc/profile

在文件尾部添加：

export NODE_HOME=/usr/local/node-v8

export PATH=$NODE_HOME/bin:$PATH

5.环境变量生效命令

source /etc/profile

6.查看版本

node -v

npm -v

## 设置taobao镜像

npm config set registry https://registry.npm.taobao.org

验证是否成功：

    [root@localhost elasticsearch-head-master]# npm config get registry
    https://registry.npm.taobao.org/
    

参考链接：https://blog.csdn.net/quuqu/article/details/64121812


