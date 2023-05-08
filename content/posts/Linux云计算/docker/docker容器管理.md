---
title: "docker容器管理"                         
author: "博文"   
date: 2023-05-04    
comments: true  
tags : [                                    
     "docker",
 ]
categories : [                              
     "Linux云计算",
 ]
---
>这篇文章依旧是我看[B站上的视频](https://www.bilibili.com/video/BV1DT411U7v1?p=14&vd_source=4c2c9caf33151d42ae4b296d7b5f6f45)摘录的笔记。主要是docker容器相关的内容。

# 1. 启动容器玩法

`docker run` 等于创建+启动 ，`docker run` 镜像名，**如果镜像不存在本地，则会在线去下载该镜像** 注意
{{< admonition type=warning title=前台运行  >}}
-   容器内的进程必须处于前台运行(例如ping)状态，否则容器就会直接退出。
-   没有运行任何程序，因此容器直接挂掉，容器内必须有一个**程序前台运行**。
{{< /admonition >}}

## 直接跑

比如执行`docker run nginx` ，会直接前台启动容器，启动nginx服务，ctr+c后会取消。进程也就没了。当然实际很少这么用。

## it参数

```bash
docker run -it nginx 
docker run -it nginx bash
docker run -it centos  ping www.baid.com
```

## 后台跑 -d
```bash
[root@centos8 ~]# docker run -d centos ping www.baid.com 
4fecf169ebb48eff5eaf7adad2809361e5b59cfdf18eef93e6279a0e994602b4
```

![](/docker/20230504101709.png)

可以看到除了这个-d，其他的容器都是**exit**状态。要注意，会一直ping。例如下面这个容器我运行很长时间，再查看日志还是在不断刷新ping这个命令。

![](/docker/20230504101912.png)


##  `rm`参数

练习时产生了很多容器，这些容器只是调试用，就可以使用这个参数，**当stop掉容器后就自动删除了。**

```bash
docker stop python
```

## 给容器起名字

```bash
[root@centos8 ~]# docker run -d  --rm --name python centos ping www.baidu.com
0d319a46aba202c3286597a7fe7007d5cad78370cd94a7432fdd28d7c5e336f2
```

## 端口

🍆注意前面的80是宿主机的。后面的80是容器内的。

```bash
docker run -d --name bowen_nginx -p 80:80 nginx
```

```bash
[root@centos8 ~]# docker port 1c2
80/tcp -> 0.0.0.0:80
80/tcp -> :::80
```

随机端口映射

```bash
[root@centos8 ~]# docker run -d --name nginx_random  -P  nginx
3e813c937ff464f722ca61ea6f7254b0991585e6e81f7b6ab457fa295c0f5ce1
[root@centos8 ~]# docker port 3e
80/tcp -> 0.0.0.0:49153
80/tcp -> :::49153
```



>  上面介绍了使用docker run启动容器时的相关参数

# 2. 查看容器日志

```bash
[root@centos8 ~]# docker logs -f python
PING www.a.shifen.com (180.101.50.188) 56(84) bytes of data.
64 bytes from 180.101.50.188 (180.101.50.188): icmp_seq=1 ttl=127 time=9.14 ms
64 bytes from 180.101.50.188 (180.101.50.188): icmp_seq=2 ttl=127 time=9.21 ms
64 bytes from 180.101.50.188 (180.101.50.188): icmp_seq=3 ttl=127 time=10.0 ms
64 bytes from 180.101.50.188 (180.101.50.188): icmp_seq=4 ttl=127 time=10.2 ms
```

> 这个在定位容器启动失败时会用到，我在openstack中遇到horizon容器起不来，就是用的这个命令查看日志报错，报错信息会给我们破案提供一定的参考。


# 3. 进入容器

```bash
[root@centos8 ~]# docker exec -it python bash
```

进来看看, 就能看到ping这个进程，就这三条记录，更多进程是在宿主机跑的，**依赖宿主机内核。**
```bash
[root@e8c702290e7b /]# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 13:18 ?        00:00:00 ping www.baidu.com
root           7       0  1 13:20 pts/0    00:00:00 bash
root          21       7  0 13:21 pts/0    00:00:00 ps -ef
```

# 4. 查看容器详细信息

```bash
docker container inspect python
```

> 这条命令和查看镜像信息类似的，也是一个json格式的，存放了容器的所有信息。


# 5. 基于容器commit新镜像

启动centos容器，我在里面修改yum源，安装vim等命令；然后提交，**前面是容器名，后面是镜像名。

```bash
docker commit e8c702290e7b bowen/centos8.4
```

基于新镜像，启动一个容器，可以看到，里面有了vim命令了。

```bash
[root@centos8 ~]# docker run -it bowen/centos8.4 bash
[root@35493a94ca8c /]# vim bowen.txt
```

# 6. 删除
```bash
[root@centos8 ~]# docker rm 4fecf169ebb4 0c842c8b3677 331afa888c98 4fe280c588a9 -f
4fecf169ebb4
0c842c8b3677
331afa888c98
4fe280c588a9
```

