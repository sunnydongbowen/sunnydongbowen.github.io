---
title: "解决VMware centos8 网络问题"                         
author: "博文"   
date: 2023-06-13        
comments: true  
tags : [                                    
     "问题解决",
     "网络"
 ]
categories : [                              
     "Linux云计算",
 ]
---
报错信息如下(<font color="#ffc000">这个报错不止遇到过一次了</font>)

```csharp
Connection 'ens33' is not available on device ens33 because device is strictly unmanaged
```

解决方法
查看托管状态
```bash
nmcli n
```
显示 disabled 则为本文遇到的问题，如果是 enabled 则可以不用往下看了。 如果不是，则开启托管，重启服务。
```bash
nmcli n on
systemctl restart NetworkManager
```

这个错误我遇到过两次，第一次不是因为上面的原因。第二次则是按照上面方法解决的。这是我在准备k8s搭建的时候使用VMware 安装虚拟机遇到的错误。
[参考链接](https://www.cnblogs.com/yadongliang/p/14124031.html)  这个链接后面还有一些nmcli的命令，可以参考一下。

