---
title: "认识docker和虚拟化"                         
author: "博文"   
date: 2023-05-01        
comments: true  
tags : [                                    
     "docker",
 ]
categories : [                              
     "Linux云计算",
 ]
---
>之前其实使用docker也挺多了，却没系统的学习它。现在学了go，可以带着把docker和k8s系统学习一下。这篇文章是我看B站上下面这套视频1~12摘录的笔记。

{{< bilibili BV1DT411U7v1 1 >}}


# 1. 部署方式

### 纯服务器部署

正常步骤
1.  安装系统
2.  部署软件，修改配置文件
3.  启动

纯物理服务器部署的缺点
-   部署慢
-   成本高
-   效率低
-   资源浪费
-   难迁移和扩展

### 虚拟机部署形式

这种方式的局限性，每个虚拟机都是完整的操作系统，占用资源大，成本很高。当然，它也有它的优点，虚拟化相关的东西就不在这里提了。


# 2. 虚拟化技术


[虚拟化介绍](https://developer.aliyun.com/article/768343)  详细的可以看这篇阿里云的文章，介绍的很好。目前主流的虚拟化平台，VMware的虚拟化技术，亚马逊云，阿里云，还有金山云，华为云，京东云等基于openstack开源技术整合成的。开源的这些底层技术依然是kvm-qemu+livbirt这些。

不过这篇文章里面还提出了超轻虚拟机，亚马逊和google，微软都有在做。类似这个**microVM，其实就是砍掉了大量的无用的功能。微型操作系统。这种目前是不符合市场需求的。** 还未大量普及。

这些虚拟化技术，是和硬件相关，硬件辅助虚拟化技术，AMD-V，Inter V-T开启。这里不详细介绍了。

# 3. 容器技术

如果只是需要单独的隔离环境，轻量级的。那容器就够了，不需要单独的物理服务器和虚拟机，而且可以避免无限增加虚拟机，物理机，节约资源。

![Untitled](/docker/20230503103506.png)

docker和虚拟机的对比，具有启动快，更加轻量的优势。之前从事的一直是虚拟机相关这种重量级的虚拟化。涉及openstack，kvm, vmware还有各种云平台这些。可以说这里面涉及的内容太多太多了。下面是chatgpt关于这两个的对比，回答的很好。

{{< admonition type=note title=docker和虚拟机对比  open=false >}}

Docker和虚拟机是两种不同的虚拟化技术，它们都有各自的优缺点。

虚拟机是一种完全**虚拟化技术**，它可以在一台物理服务器上运行多个操作系统。虚拟机软件会模拟出一套完整的硬件环境，包括CPU、内存、硬盘等，然后在这个环境中运行一个完整的操作系统。因此，每个虚拟机都拥有自己的内核、进程空间和文件系统。虚拟机之间是完全隔离的，它们可以运行不同版本的操作系统和软件。

相比之下，Docker是一种**容器化技术**，它可以在一台宿主机上运行多个容器。Docker容器是一种**轻量级的虚拟化技术**，它利用宿主机的操作系统内核，通过名称空间和控制组等机制，将容器中的进程隔离出来。每个容器都拥有自己的文件系统和进程空间，但它们共享宿主机的操作系统内核。因此，Docker容器比虚拟机**更轻量级、更快速，可以更容易地创建和销毁**。

下面是Docker和虚拟机之间的一些比较：

1.**资源占用**：由于虚拟机需要模拟硬件环境，因此它们需要更多的资源（如内存和CPU）。而Docker容器共享宿主机的内核和其他资源，因此它们需要更少的资源。

2.**启动时间**：由于虚拟机需要启动完整的操作系统，因此启动时间更长。而Docker容器只需要启动容器进程，因此启动时间更快。

3.**隔离性**：虚拟机之间是完全隔离的，它们可以运行不同版本的操作系统和软件。而Docker容器共享宿主机的操作系统内核，因此容器之间的隔离性相对较弱。

4.**映像文件大小**：由于Docker容器共享宿主机的操作系统内核，因此容器的映像文件相对较小。而虚拟机需要包含完整的操作系统和应用程序，因此映像文件较大。

综上所述，虚拟机和Docker都有各自的优缺点，选择哪种技术取决于具体的应用场景和需求。如果需要完全隔离和安全性较高的环境，可以选择虚拟机；如果需要更快的部署和更轻量级的环境，可以选择Docker容器。

{{< /admonition >}}

所以docker 具有更快的启动时间，可以保证一致性环境，持续交付和部署，  更便捷的迁移环境。Google 2004年就开始使用docker了。14年推出了k8s。京东也在14年大量使用docker，由kvm虚拟机到docker，淘宝也是这这样。现在很多小厂大多也是用的docker技术，毕竟不是每个公司都有这个能力去在内部构建openstack，云桌面等这些大型虚拟化技术的。所以docker还是很值得学习，不仅仅大厂的众多服务由虚拟机转向了容器技术，小厂的使用更是广泛。

{{< admonition type=warning title=容器不是万能的👉   >}}
要提醒的是，**容器也不是万能的，无法虚拟硬件，也无法替代物理服务器，虚拟机。这三个各自有不同的使用场景**。
{{< /admonition >}}

# 4. docker组成部分

![](/docker/20230502175100.png)
 
 > 从上图可以看到，docker是cs架构的
 
1.  启动服务端，就是docker的进程。
2.  docker client，客户端使用REST API和Docker daemon进行访问
3.  外层RESTAPI 接口，提供和daemon交互的api接口，或者docker 提供的指令来操作。
4.  容器，最核心的东西
5.  镜像，构建容器，将应用程序所需的环境打包为镜像文件
6.  网络
7.  存储

![](/docker/20230503105627.png)

### 镜像

镜像是一个只读模板，用于创建容器，也可以通过Dockerfile文本描述镜像的内容。镜像的概念类似于编程开发里面向对象的类，从一个基类开始 (基础镜像Base image)构建容器的过程，就是运行镜像，生成容器实例。

Docker镜像的描述文件是Dockerfile，包含了如下的指令

1. FROM 定义基础镜像

2. MAINTAINER 作者

3. RUN 运行Linux命令

4. ADD 添加文件/目录

5. ENV 环境变量

6. CMD 运行进程

>  当然还有更多，这个后面会继续介绍。

### 容器

容器是一个镜像的运行实例，镜像 -> 容器。创建容器的过程
1. 获取镜像，如 docker pull centos ，从镜像仓库拉取
2. 使用镜像创建容器
3. 分配文件系统，挂载一个读写层，在读写层加载镜像。
4. 分配网络/网桥接口，创建一个网络接口，让容器和宿主机通信
5. 容器获取IP地址
6. 执行容器命令，如/bin/bash
7. 反馈容器启动结果。

### 仓库
镜像仓库 保存，上传/下载镜像

docker提供了registry仓库，它其实也是一个容器。可以基于该容器构建私有仓库。这个在离线的时候会用到。在之前openstack项目中，我就是使用它构建了离线的docker私有仓库。因为是军工项目，是不允许连外网的。

也可也使用docker hub互联网共有镜像仓库。

# 5. 安装docker

基于虚拟化，安装教程有很多，**不同操作系统不同版本**安装方式也不一样，就不多说了，在线安装很简单，配置源后，一条命令安装就好了。离线的话需要安装离线仓库，这个在其他文章有介绍。安装完成后

```bash
[root@centos8 ~]# docker version
Client: Docker Engine - Community
 Version:           20.10.18
 API version:       1.41
 Go version:        go1.18.6
 Git commit:        b40c2f6
 Built:             Thu Sep  8 23:11:56 2022
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.18
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.18.6
  Git commit:       e42327a
  Built:            Thu Sep  8 23:10:04 2022
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.8
  GitCommit:        9cd3357b7fd7218e4aec3eae239db1f68a5a6ec6
 runc:
  Version:          1.1.4
  GitCommit:        v1.1.4-0-g5fd4c4d
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

可以看到，这里有docker客户端，服务端，还显示了所使用的go版本，还有guild的时间。

{{< admonition type=warning title=注意不同docker版本的坑   >}}

我在做openstack项目中，就遇到，最后定位是docker用了最新的版本，最新版本把里面的一个参数去掉了，而我在脚本里写了去配置这个参数，就导致启动失败。
在实际工作中，为了保证稳定性，避免出现一些未知的坑，尽可能选择比较常用的版本。但也不能选择过于落后的版本。

{{< /admonition >}}
# 6. 使用docker搭建nginx

pull 镜像

```bash
[root@centos8 ~]# docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
3f9582a2cbe7: Pull complete
9a8c6f286718: Pull complete
e81b85700bc2: Pull complete
73ae4d451120: Pull complete
6058e3569a68: Pull complete
3a1b8f201356: Pull complete
Digest: sha256:aa0afebbb3cfa473099a62c4b32e9b3fb73ed23f2a75a65ce1d4b4f55a5c2ef2
Status: Downloaded newer image for nginx:latest
```

查找可用镜像
```bash
[root@centos8 ~]# docker search  nginx
NAME                                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
nginx                                             Official build of Nginx.                        18234     [OK]
linuxserver/nginx                                 An Nginx container, brought to you by LinuxS…   188
bitnami/nginx                                     Bitnami nginx Docker Image                      153                  [OK]
ubuntu/nginx
```

查看镜像
```bash
[root@centos8 ~]# docker image ls
REPOSITORY         TAG       IMAGE ID       CREATED         SIZE
nginx              latest    904b8cb13b93   13 days ago     142MB
redis              latest    9da089657551   5 months ago    117MB
hashicorp/consul   1.10.0    747e0258fcf2   21 months ago   117MB
[root@centos8 ~]# docker images
REPOSITORY         TAG       IMAGE ID       CREATED         SIZE
nginx              latest    904b8cb13b93   13 days ago     142MB
redis              latest    9da089657551   5 months ago    117MB
hashicorp/consul   1.10.0    747e0258fcf2   21 months ago   117MB
```

删除镜像
```bash
docker rmi 镜像id
```

启动容器

```bash
[root@centos8 ~]# docker run -d -p 80:80 nginx
15b206795c17fabbc70396d8f72f3348c985d505ba7ad50481d8a68bc3936321
```

查看容器
```bash
[root@centos8 ~]# docker ps
CONTAINER ID   IMAGE                     COMMAND                  CREATED          STATUS          PORTS                                                                                                                                                           NAMES
15b206795c17   nginx                     "/docker-entrypoint.…"   24 seconds ago   Up 22 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp                                                                                                                               stoic_burnell
5dce7cfca9dd   hashicorp/consul:1.10.0   "docker-entrypoint.s…"   3 months ago     Up 2 months     8300-8302/tcp, 8500/tcp, 8301-8302/udp, 8600/tcp, 8600/udp                                                                                                      consul-client
dec4d3119de0   hashicorp/consul:1.10.0   "docker-entrypoint.s…"   3 months ago     Up 2 months     0.0.0.0:8500->8500/tcp, :::8500->8500/tcp, 8300-8302/tcp, 8301-8302/udp, 0.0.0.0:8600->8600/tcp, :::8600->8600/tcp, 0.0.0.0:8600->8600/udp, :::8600->8600/udp   consul-server
9234bb009dcd   redis                     "docker-entrypoint.s…"   5 months ago     Up 2 months     0.0.0.0:6379->6379/tcp, :::6379->6379/tcp
```

pc端输入80端口，可以看到访问成功

![Untitled](/docker/20230503114153.png)

> 看到这里，是不是想到了consul搭建，很类似？通过上面的示例，简单了解一下容器的用法。


# 7. docker生命周期

>  可以看一下这张图，后面会详细介绍。

![](/docker/20230503155603.png)
# 8. 三个基础镜像

我们可以通过pull命令先把基础镜像拉取下来
```bash
docker pull ubuntu
docker pull centos
docker run -it centos bash
```
解释一下启动的命令参数
- -i 交互式命令操作

- -t 开启终端

- bash 在进入容器后执行的命令

**exit退出，退出后容器就stop掉了。这个要注意。因为没有-d**

![](/docker/20230503173600.png)