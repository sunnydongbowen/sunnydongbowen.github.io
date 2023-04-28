---
title: "ansible"                         
author: "博文"   
date: 2023-04-27          
comments: true  
tags : [                                    
     "Linux基础",
 ]
categories : [                              
     "Linux云计算",
 ]
---

 ✏️ 记录了在批量部署openstack云平台时学习到的ansible命令，确实很方便！是使用py开发的。有时间可以看看源码，不少做运维平台，主机管理是基于这个做的web。

# 1. 远程执行命令

指定非部署节点执行

```bash
ansible -i deploy.ini no_deploy  -m shell -a "python3 /offline_deploy/offline.py"
```

指定所有节点执行

```bash
ansible -i deploy.ini all -m shell -a "python3 /offline_deploy/allnode.py"
```


deploy.ini内容如下：

```
[deploy]
cloud1
[no_deploy]
cloud2
cloud3
```

# 2. 远程推送文件

```bash
ansible -i bw.ini all -m copy  -a "src=xxxx dest=xxxx"
```

这两个功能很好用，之前用**paramiko模块编写的py脚本，实现批量执行命令和批量推送文件，这是建立在知道ip基础上的**，也很好用。部署opentack云环境，用ansible和kolla-ansible在一台主机上实现部署，很方便。
ansible可以指定传输整个目录及权限
```
ansible -i /root/deploy.ini no_deploy -m copy -a "src=/etc/ceph/  dest=/etc/ceph mode=777"
```

[ansible常用模块之文件操作](https://blog.csdn.net/weixin_44791884/article/details/105026529)

# 3. 查看日志
如果想查看详细的日志，可以加上 -vvv 参数

```
-vvv
```

# 4. 使用示例

-   批量在执行非部署节点脚本，17台服务器

```bash
ansible -i deploy.ini no_deploy  -m shell -a "python3 /root/online_install_notdeploynode.py"
```

-   批量传送文件

```bash
ansible -i /root/deploy.ini no_deploy -m copy -a "src=deploynode.py  dest=/root"
```

注意传送文件时**尽量不要批量传输大文件**，否则**带宽不足**会比一台一台传输还慢的。我们在实际部署的时候就尝试过用它批量传输大文件，结果比单对单传输还慢。带宽达到瓶颈了。

# 5. 小结

>  ansible 在批量管理主机的时候还是相当有用的。我目前只是用到了下面两种来辅助我的大规模部署。

- 批量传输文件
- 批量执行命令