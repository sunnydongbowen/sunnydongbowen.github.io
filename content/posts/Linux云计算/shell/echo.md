---
title: "echo命令"                         
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

- echo

```bash
[root@os55 ~]# echo test test.txt
test test.txt
```

- echo 与> , 在特定的场合，也是需要直接覆盖模式的，类似python的w模式

```bash
[root@os55 ~]# cat test.txt
[root@os55 ~]# echo test > test.txt
[root@os55 ~]# cat test.txt
test
```

- > > ，类似python的a模式
```bash
[root@os55 ~]# ls >> test.txt
[root@os55 ~]# cat test.txt
test
```