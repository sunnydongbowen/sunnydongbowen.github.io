---
title: "ifconfig与ping"                         
author: "博文"   
date: 2023-06-21      
comments: true  
tags : [                                    
     "Linux基础",
 ]
categories : [                              
     "Linux云计算",
 ]
---
## ipconfig
---
一台计算机中可能有多个物理网卡，和更多个虚拟网卡
- 查看ip信息
```bash
ifconfig | grep inet | grep -v inet6
        inet 172.19.0.1  netmask 255.255.0.0  broadcast 172.19.255.255
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet 172.16.1.155  netmask 255.255.254.0  broadcast 172.16.1.255
        inet 127.0.0.1  netmask 255.0.0.0
```
回环网卡一般用来测试本机网卡是否正常

## ping
---
ping 在工作中也经常用，查看网络通信。下面表示发送56bytes的数据包到目标主机，返回64字节，<font color="#ffc000">时间越短表示网络情况越好</font>
```bash
[root@cloud1 ~]# ping 172.16.1.156
PING 172.16.1.156 (172.16.1.156) 56(84) bytes of data.
64 bytes from 172.16.1.156: icmp_seq=1 ttl=64 time=0.296 ms
64 bytes from 172.16.1.156: icmp_seq=2 ttl=64 time=0.122 ms
64 bytes from 172.16.1.156: icmp_seq=3 ttl=64 time=0.141 ms
64 bytes from 172.16.1.156: icmp_seq=4 ttl=64 time=0.178 ms
64 bytes from 172.16.1.156: icmp_seq=5 ttl=64 time=0.245 ms
```

```bash
[root@cloud1 ~]# ping 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.035 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.029 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.033 ms
```
> 在B站中有一个关于用go实现ping命令的，可以试一下。更有助于理解底层的协议。