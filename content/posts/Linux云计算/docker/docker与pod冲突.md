---
title: "docker与pod冲突"                         
author: "博文"   
date: 2023-06-14    
comments: true  
tags : [                                    
     "docker",
 ]
categories : [                              
     "Linux云计算",
 ]
---
在安装docker时，如果是centos8桌面版，会自带pod，需要将其卸载。再安装和启动docker.<font color="#ffc000"> 不然会冲突，装不上docker的。</font>
```bash
dnf remove podman 
yum erase podman buildah
```

