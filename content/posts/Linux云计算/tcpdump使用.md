---
title: "tcpdump使用"                         
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

>  ✍️ tcpdump命令在服务端定位问题时是非常有用的，可以清晰看到数据的流转过程，帮助我们分析和定位问题。<font color="#ffc000">做CDN项目时在各个模块的服务器抓包，下载下来，使用wireshark分析全流程，能很清晰看到过程是什么样子的,数据包走到了哪里。</font>不记得参数可以使用help命令查一下

- 抓包并保存
```bash
tcpdump  -n -vvv -c 1000 -w /root/tcpdump_save.cap
```

- 不保存，直接显示（18年去面网络研究院的时候问过！Linux上如何看，他想问的应该是这个！）
```bash
tcpdump  -n -vvv -i any  host 192.168.31.131
```

- 抓指定网卡的包
```bash
tcpdump -i eth0 -c 10000 -w /tmp/temp.cap  # eth0的包，抓到10000个包后退出
```

> 其余的命令可以详细看help文档。