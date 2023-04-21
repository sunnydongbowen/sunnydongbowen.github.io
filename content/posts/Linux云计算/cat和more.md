---
title: "cat|more|grep"                         
author: "博文"   
date: 2023-04-19          
comments: true  
tags : [                                    
     "Linux基础",
 ]
categories : [                              
     "Linux云计算",
 ]
---

## 1、more


{{< admonition type=note title="more"  >}}
- 从头开始，在屏幕最下方表示显示了多少百分比, 分屏显示，与man命令一样的操作

- 按空格键进行下翻一页或f 

- 回车键滚动一行

- q 退出

- b 回滚看

- /word 搜索
{{< /admonition >}}


## 2、cat

> cat   一次性显示所有的内容

-   cat -n 编号，所有行，包括空行
-   cat -b 不包括空行，编号

cat命令还包含文本合并等其他功能

一般来说，全文搜索，用cat | grep 居多一些

## 3、grep🏵️

强大的文本搜索工具，下面这三个选项很常用！务必记住！

-   -i 忽略大小写
-   -v 过滤掉含有的内容
-   -n 显示行号

下面的示例**oskey是文件名**。

```bash
[root@cloud1 ~]# grep -in end oskey
27:-----END RSA PRIVATE KEY-----
```

正则表达式

-   搜索以b开头的行

```bash
[root@cloud1 ~]# grep ^b oskey
bhbuU4DPwGtHPnP5HQUVOEmkwQQeOltYh9TlO1EPnGDF0JcPSzoK97SNvvx/bhPW
```

-   搜索以==结尾的行

```bash
[root@cloud1 ~]# grep ==$ oskey
xs+QeIJsXIlq0gKQ14d2k1hRN8REHjn04rLZ51C5JE+hq+IpGSGMHQ==
```

-   递归查找，这个在搜索日志的时候很强大

```shell
grep -R aaa *.log
```