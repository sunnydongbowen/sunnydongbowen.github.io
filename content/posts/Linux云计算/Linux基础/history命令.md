---
title: "history命令"                         
author: "博文"   
date: 2023-06-18         
comments: true  
tags : [                                    
     "Linux基础",
 ]
categories : [                              
     "Linux云计算",
 ]
---
>  history命令主要用在查看执行过的命令，很有帮助。

修改默认显示的行数，默认是1000行
```bash
vim /etc/profile
HISTSIZE=10
source /etc/profile
```

再次输入history命令，只显示10行了

删除
```bash
history -d746  #  清除指定行
history -c     #  全部清除，做镜像的时候有用到
```

快速执行,输入 !编号
```bash
[root@os11 ~]# history
  745  vim /etc/profile
  746  history
  747  who
  748  history
[root@os11 ~]# !747
who
root     pts/0        2021-10-29 13:33 (192.168.1.16)
root     pts/2        2021-10-29 11:34 (192.168.1.16)
```

不记忆方法，<font color="#ffc000">这个做坏事用的，哈哈</font>。
```bash

vim /etc/profile
HISTCONTROL=ignorespace  #加入这一行

source /etc/profile
```
在输入的命令前，<font color="#00b050">可以打一个空格，这样是不记录这条命令的</font>