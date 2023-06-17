---
title: "使用yum安装docker"                         
author: "博文"   
date: 2023-05-10       
comments: true  
tags : [                                    
     "docker",
 ]
categories : [                              
     "Linux云计算",
 ]
---
在线搭建 (客户端，服务端)

```bash
wget -P /etc/yum.repos.d/  https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

```bash
yum -y install docker-ce docker-ce-cli containerd.io
systemctl enable docker
```