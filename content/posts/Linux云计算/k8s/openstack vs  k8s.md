
---
title: "openstack vs k8s"                         
author: "博文"   
date: 2023-06-11         
comments: true  
tags : [                                    
     "k8s",
 ]
categories : [                              
     "Linux云计算",
 ]
---
|openstack|k8s|
|---|---|
|面向vm，管理vm(但openstack的magnum, kata，zune等项目都是与容器结)|面向容器，管理docker，底层是容器技术|
|架构复杂，部署也相对复杂|架构和部署相对简单的多|
|py为主|Go为主|
|趋于稳定，经历过上万台服务器的测试|业界新宠|
|IAAS层|PAAS层+sass|
|重量级，启动慢|轻量级，启动快|
|很多大公司都很看中，电信，阿里，京东等等，小公司很难用的来，需要专业的运维人才|小公司很重视，CICD这些，都用的k8s|
|侧重虚拟化，需要虚拟化技术积累，也需要相关硬件知识|侧重集群管理、调度、应用编排|

从精力的角度上讲，这两个只能选一个深入研究下去。人的精力毕竟是有限的。

从工作角度看，显然是**k8s+golang+docker**更具有竞争力，薪资和岗位也偏多，学习起来需要的资源也更小！

**`openstack+kvm+python`**，它整个框架过于庞杂！虽然`k8s+golang+docker`无法取代openstack这一套，很多小公司，如果只是内部应用，更偏向k8s一些！找工作也想对好找一些，抛开他俩前景，就从找工作的角度上看，k8s+golang+docker更胜一筹，现在很多中小企业都在使用k8s系统了。

他们有时候甚至是共存的，k8s运行在openstack虚拟机里。或者是用k8s部署openstack环境，b站上关于openstack的教程不但很少，都比较老了！自己现在研究的，是基于centos8+V版本的，实操部分只能参考着学一下原理。


> 对于openstack，太重量级了，掌握搭建、使用、基础运维即可。