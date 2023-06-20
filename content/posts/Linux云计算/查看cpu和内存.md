---
title: "查看cpu和内存"                         
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
> 在拿到服务器，去了解服务器的硬件和软件信息是很有必要的，内存，cpu，网络，存储这些性能信息等。

- 查看内存型号
```bash
dmidecode -t memory
free -m
free -h
```

- CPU型号和主频
```bash
cat /proc/cpuinfo | more
```

- CPU相关
```bash
# 	查看物理cpu个数
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l
#  	查看每个cpu核数
cat /proc/cpuinfo| grep "cpu cores"| uniq
#   逻辑cpu个数
cat /proc/cpuinfo| grep "processor"| wc
#   总个数
grep proc /proc/cpuinfo
```

```bash
lscpu | grep '^CPU(s)'
```