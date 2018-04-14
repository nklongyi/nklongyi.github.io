---
layout:     post
title:      kubernetes的集群安装和测试
subtitle:   探索使用k8s生产应用
date:       2018-04-14
author:     longyi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - docker
    
---

## 修改主机hostname for centos7

第一步：修改/etc/sysconfig/network文件

        #>vi /etc/sysconfig/network

        添加或修改:

                NETWORKING=yes

                HOSTNAME=slave3

第二步：修改/etc/hosts文件

        #>vi /etc/hosts

        修改 127.0.0.1这行中的 localhost.localdomain为 slave3

        修改 ::1这行中的localhost.localdomain 为slave3

第三步 ：修改/etc/hostname文件(此步不操作，怎么修改都没有用)

        删除文件中的所有文字，在第一行添加slave3

第四步：重启并验证

       #>reboot -f 

       #> hostnamectl

## 



