---
title: "解决VMware centos8 网络问题"                         
author: "博文"   
date: 2023-06-13        
comments: true  
tags : [                                    
     "问题解决",
     "网络",
     "系统安装"
 ]
categories : [                              
     "Linux云计算",
 ]
---
## 问题1
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
><font color="#ffc000"> 同样的，nmcli命令也不要去死记硬背。知道有这个东西，用到了去查！去记是记不住的</font>！

## 问题2
---
VMware 虚拟机可以获取到ip，但无法ping 通外网。可以ping通同网段，但<font color="#92d050">ping 8.8.8.8 和114不通。说明无法解析外网，也无法通外网</font>。我对比了一下，觉得还是网关和dns的问题。但dns查看后配置的和正常的一样。<font color="#92d050">注意断开代理</font>。

![](/linux基础/20230613210227.png)

最后解决方法，nmtui，选择网卡。其他改成自动获取，ip要写死。这是为了防止ip后面变化。

