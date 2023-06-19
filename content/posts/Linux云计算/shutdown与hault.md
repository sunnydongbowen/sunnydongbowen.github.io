---
title: "shutdown与hault"                         
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
## 区别
---
在CentOS7上验证结论：
- halt 是强制关机，需要手动关闭电源；只关闭系统，不切断电源。
- 而poweroff 、shutdown都会先给 ACPI （Advanced Configuration and Power Management Interface）一个命令，先关闭OS然后关闭电源。poweroff和shutdown其实没什么区别。
```bash
[root@localhost iso]#  ls -l /sbin/poweroff
lrwxrwxrwx. 1 root root 16 12月  6 2020 /sbin/poweroff -> ../bin/systemctl
```
记住常用的就可以了，一般像放长假，会执行关机操作。

## shutdown
---
在管理服务器过程中，应该重启，加-r 参数，而不是关机。除非节假日放假服务器关机，要慎用这个命令。练习的时候最好别用正式的机器，万一出错了麻烦了，用虚拟机。
```bash
# 重新启动系统
shutdown  -r now 

# 立刻关机
shutdown now

# 今天20:25关机
shutdown 20:25

# 过10min后关机
shutdown +10

取消关机
shutdown -c

默认是一分钟关闭
shutdown 
```

