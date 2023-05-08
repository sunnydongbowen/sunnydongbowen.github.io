---
title: "docker镜像管理"                         
author: "博文"   
date: 2023-05-03      
comments: true  
tags : [                                    
     "docker",
 ]
categories : [                              
     "Linux云计算",
 ]
---
>这篇文章依旧是我看[B站上的视频13~20](https://www.bilibili.com/video/BV1DT411U7v1?p=14&vd_source=4c2c9caf33151d42ae4b296d7b5f6f45)摘录的笔记。主要是docker镜像相关的内容。

# 1.分层原理

![](/docker/20230503175626.png)

1.  最底层（Bootfsboot-file system)主要包含bootloader和kernelder主要是引导加载kernel，Linux刚启动时会加 **bootfs文件系统**
2.  R**ootfs (root-file system)** ，在bootfs之上包含的就是典型Linux系统中的/dev、/proc、/bin、/etc等标准目录和文件rootfs就是各种不同操作系统的发行版，比如Ubuntu，Centos等等
3.  **Union File System  docker通过联合文件系统**，**将上述的不同的每一层，整合为一个文件系统，为用户隐藏了多层的视角**。
4.  只能修改最上层。

![](/docker/20230503190308.png)

> docker镜像是在基础镜像之后，安装软件，配置软件，添加新的层，构建出来这种现象在学习dockerfile构建时候，更为清晰


# 2. 2个问题

> 当通过一个image启动容器时，docker会在该image最顶层，**添加一个读写文件系统作为容器**，然后运行该容器 。 docker镜像本质是基于**UnionFS管理的分层文件系统**

{{< admonition type=question title=docker镜像为什么才几百兆?   >}}

答:  因为docker镜像只有rootfs和其他镜像层，**共用宿主机的linux内核 (bootfs)**，因此很小!

{{< /admonition >}}


{{< admonition type=question title=nginx镜像有133MB，但是nginx安装包只有几M >}}

答:  因为doker的nginx镜像是分层的，nginx安装包的就几M，镜像用于运行nginx的像文件，依赖镜像上一层，和**基础像发行版**等，所以nginx镜像和正常安装包来说又很大。

{{< /admonition >}}

> 上面两个问题，其实最关键的还是在基础镜像，因为共用内核，所以它和虚拟机镜像比起来很小，但又因为它有一个基础发行版的镜像和其他依赖镜像，所以它和单独安装包比起来又很大。


# 3. 为什么分层

镜像分层一大好处就是共享资源，例如有多个镜像都来自于同一个base镜像，那么在docker host只需要存储一份base镜像。内存里也只需要加载一份host，即可为多个容器服务 即使多个容器共享一个base镜像，某个容器修改了base镜像的内容，例如修改/etc/下配置文件，其他容器的/etc/下内容是不会被修改的，修改动作只限制在单个容器内，这就是容器的**写入时复制特性 (Copy-on.write)**，**改的是最顶层的，所以不影响由base镜像构建出来的其他容器**。


![Untitled](/docker/20230503191956.png)

当容器启动后，一个新的可写层被加载到镜像的顶部,这一层通常被称为 **容器层**。

![](/docker/20230503192425.png)

其实在上篇博客中，我介绍docker搭建nginx服务时，可以看到它pull的不止一个，而是多个。这是因为docker镜像的分层技术。我们可以进入nginx容器，查看它使用的底层操作系统镜像，也就是发行版镜像。
```bash
[root@centos8 ~]# docker exec -it 15b206795c17  bash
root@15b206795c17:/# cat /etc/*-release
PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
NAME="Debian GNU/Linux"
VERSION_ID="11"
VERSION="11 (bullseye)"
VERSION_CODENAME=bullseye
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```

我们启动几个不同的容器，他们发行版操作系统不同，**但是内核却是一样的**。
```bash
uname -r
```

![](/docker/20230503211036.png)         ![](/docker/20230503211131.png)



# 4. 镜像的分层管理

比如访问cat /etc/password.它会去image找，没找到，继续往下找，找到debian下面有，复制到最顶层的容器层。所有对容器的修改动作，都只会发生在**容器层** 里，只有**容器层 是可写的，其余镜像层 都是只读的。**


|文件操作|说明|
|---|---|
|添加文件|在容器中创建文件时，新文件被添加到容器层中|
|读取文件|在容器中读取某个文件时，Docker 会从上往下依次在各镜像层中查找此文件。一旦找到，立即将其复制到容器层，然后打开并读入内存。|
|修改文件|在容器中修改已存在的文件时，Docker 会从上往下依次在各镜像层中查找此文件。一旦找到，立即将其复制到容器层，然后修改之。|
|删除文件|在容器中删除文件时，Docker 也是从上往下依次在镜像层中查找此文件。找到后，会在容器层中记录下此删除操作。 (只是记录删除操作)|

 >只有当需要修改时才复制一份数据，这种特性被称作 **Copy-on-Write**。可见，容器层保存的是镜像变化的部分，不会对镜像本身进行任何修改。这样就解释了我们前面提出的问题: 容**器层记录对镜像的修改，所有镜像层都是只读的，不会被容器修改，所以镜像可以被多个容器共享。**

docker镜像分层管理的方式大大便捷了docker镜像的分发和存储。Docker hub是为全世界的仓库。 docker镜像代表一个容器的文件系统内容 。镜像层级技术属于 **联合文件系统** 。容器是一个动态的环境，每一层镜像里的文件都属于静态内容。dockerfile里的ENV、VOLUME、CMD等内容都会落实到容器环境里。


# 5. 使用流程

进行开发，需要安装很多第三方的工具，比如etcd,kafka,mysql,nginx等等，mac，windows，我又不想搞乱当前机器的环境mac装一个nginx，二进制安装，编译安装，brew install nginx...

1.  下载安装docker工具
2.  获取该软件的docker镜像 (你以后想要用的各种工具，基本上都能够搜索到合适的镜像去用)，下载nginx镜像， docker pull nginx
3.  运行该镜像，然后启动了一个容器，这个nginx服务就运行在容器中
4.  停止容器，删除该镜像，哦了，你的电脑上，好像就没有使用过nginx一样


# 6. 镜像操作

{{< admonition type=note title=pull镜像   >}}

```bash
docker pull nginx
```
{{< /admonition >}}

{{< admonition type=note title=查找可用镜像   >}}
```bash
docker search  nginx
```
{{< /admonition >}}


{{< admonition type=note title=查看镜像   >}}
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
{{< /admonition >}}

{{< admonition type=warning title=删除镜像   >}}
删除镜像, 删除前要确保没有其他容器在使用它。

```bash
docker rmi 镜像id
```

```bash
[root@centos8 ~]# docker rmi hello-world
Untagged: hello-world:latest
Untagged: hello-world@sha256:ffb13da98453e0f04d33a6eee5bb8e46ee50d08ebe17735fc0779d0349e889e9
Deleted: sha256:feb5d9fea6a5e9606aa995e879d862b825965ba48de054caab5ef356dc6b3412
Deleted: sha256:e07ee1baac5fae6a26f30cabfe54a36d3402f96afda318fe0a96cec4ca393359
```

>  删除的时候可以根据镜像id，完整的id或者**前三位**都可以。

批量删除，这是危险操作！

```bash
# 这两个是全部删除，一般不怎么用
docker rmi `docker images -aq`
docker rm `docker ps -aq`
```


{{< /admonition >}}

{{< admonition type=note title=查看镜像位置   >}}
```bash
docker info|grep Root
Docker Root Dir: /var/lib/docker
```
{{< /admonition >}}

{{< admonition type=warning title=退出容器后主动删除   >}}
```bash
docker run -it --rm  centos bash
```
{{< /admonition >}}

{{< admonition type=note title=只显示镜像id   >}}

```bash
[root@centos8 ~]# docker images  -q
904b8cb13b93
74f2314a03de
9da089657551
5d0da3dc9764
747e0258fcf2
```
{{< /admonition >}}



{{< admonition type=note title=导入导出镜像   >}}

假设我们使用centos8镜像创建了容器，但里面的yum源是老的，而且缺少一些命令，如vim，lsof等，就可以手动安装，然后提交。提交后导入导出镜像。这个离线的时候会用到。这个操作后面会介绍。

```bash
docker image save centos > centos
[root@centos8 ~]# docker image load -i centos
Loaded image: centos:latest
```

{{< /admonition >}}

{{< admonition type=note title=查看镜像详细信息   >}}
这其实是一个json，里面存放了镜像的所有信息。
```bash
docker image inspect 74f2314a03de
```
{{< /admonition >}}