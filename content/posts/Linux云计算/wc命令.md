
---
title: "wc命令"                         
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
# 1、查询目录下文件个数

```bash
ls |wc  -l
```
数一数，这个文件下面还真35个文件，这个命令经常用，<font color="#ffc000">比如查看docker镜像的个数，就可以</font>
```bash
docker images | wc -l
```
> 要知道用法，命令| wc -l，统计输出行数。

# 2、查询文件行数，字节数，字数
```bash
[root@host2 kolla]#  wc -lcw tox.ini
 109  293 3147 tox.ini
```

- c 统计字节数
- l 统计行数
- w 统计字数

这个在查看文件大小时很有用。

注意: 输出列的顺序和数目不受选项的顺序和数目的影响。总是按下述顺序显示并且每项最多一列。

行数、字数、字节数、文件名，<font color="#ffc000">例如查看文件行数: </font>
```bash
[root@host2 kolla]# cat tox.ini |wc -l
109
```