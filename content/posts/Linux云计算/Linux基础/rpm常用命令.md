
---
title: "rpm常用命令"                         
author: "博文"   
date: 2023-06-20         
comments: true  
tags : [                                    
     "Linux基础",
 ]
categories : [                              
     "Linux云计算",
 ]
---

>  ✍️ 在工作中，如果服务端操作系统是centos系列，会经常用到rpm命令，<font color="#ffc000">安装、升级、卸载软件包等</font>

- 卸载
```bash
  rpm -e  modulename            #卸载
  rpm -e --nodeps  modulename
```

- 查询
```bash
rpm -qa | grep mariadb
```

- 安装与升级
```bash
rpm -Uvh        --force
rpm -ivh   ...  --force
```

- 查看某个文件是哪个包安装产生的
```bash
rpm -qf /usr/share/openstack-puppet/modules/glance/manifests/api.pp
puppet-glance-13.3.1-1.el7.noarch
```

- 查看包产生了哪些文件
```bash
rpm -ql nginx     
```