---
title: "解决centos8 源报错问题"                         
author: "博文"   
date: 2023-06-17          
comments: true  
tags : [                                    
     "问题解决",
 ]
categories : [                              
     "Linux云计算",
 ]
---
报错信息
```bash
Error: Failed to download metadata for repo 'appstream': Cannot prepare internal mirrorlist: No URLs in mirrorlist
```

原因
centos8 源不维护了,**如果需要更新 CentOS，需要将镜像从 [mirror.centos.org](http://mirror.centos.org/) 更为 [vault.centos.org](https://vault.centos.org/)**

解决
```bash
cd /etc/yum.repos.d/
sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
yum makecache
```

再安装既可。注意不要`yum update -y` ，更新操作系统很费时间，更新完后系统占用的内存和cpu都很大，最后我卸载了，换成最小安装，无桌面，这样系统内存和cpu占用小，就可以腾出来给其他的了。

> 这个操作在最新的centos8容器也适用。