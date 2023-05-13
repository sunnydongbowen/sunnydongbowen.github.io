---
title: "dockerfile指令学习"                         
author: "博文"   
date: 2023-05-08  
comments: true  
tags : [                                    
     "docker",
 ]
categories : [                              
     "Linux云计算",
 ]
---
## dockerfile简介

定制docker镜像的方式有两种:

-   手动修改容器内容，commit后 导出新的镜像
-   基于dockerfile自行编写指令，基于指令流程创建镜像。


{{< admonition type=note title="虚拟机方式安装"  >}} 

> 需求安装一个mysql

1.  开启vmware
2.  运行某一个虚拟机，centos7
3.  centos7安装mysql yum instal1 mysql-server.....
5.  通过脚本，或者命令，启动mysq1即可

上面部署缓慢，且改变了宿主机环境，删除较为麻烦，占用宿主机3306端口

{{< /admonition >}}

{{< admonition type=tip title="容器化部署"  >}} 

> 我们要学的是基于容器去运行mysql

1.  开始vmware
2.  运行虚拟机centos7(宿主机)
3.  安装docker容器软件
4.  获取mysql镜像即可，docker pull mysql:tag (你无法自由控制，该mysql的基础镜像是什么发行版，你获取的镜像，是别人定制好，你下载使用的，debian，你希望得到一个基于centos7.8的发行版运行mysql)
5.  直接运行该镜像，通过端口映射，运行mysql
6.  访问宿主机的一个映射端口，访问到容器内的mysql

{{< /admonition >}}


## 自定义镜像

> **`FROM`** 这个镜像的妈妈是谁? (指定基础镜像)
> 
> `MAINTAINER` 告诉别人，谁负责养它? (指定维护者信息，可以没有)
> 
> `RUN` 你想让它干啥 (在命令前面加上RUN即可)，**最重要**！
> 
> `ADD` 给它点创业资金 (`COPY`文件，会自动解压)，添加宿主机文件到容器内
> 
> `WORKDIR` 我是cd,今天刚化了妆 (设置当前工作目录)
> 
> `VOLUME` 给它一个存放行李的地方 (设置卷，挂载主机目录)
> 
> `EXPOSE`它要打开的门是啥 (指定对外的端口)，比如nginx的80端口
> 
> `CMD` 奔跑吧，兄弟! (指定容器启动后的要干的事情)，比如nginx run的时候实际上执行的是一个脚本
> 
> `COPY` 复制文件     `ENV` 环境变量        `ENTRYPOINT` 容器启动后执行的命令

比如进入redis容器，直接进入data目录，就是workdir参数

```bash
[root@centos8 ~]# docker exec -it myredis bash
root@9234bb009dcd:/data#
```

## 熟悉构建流程

编写dockerfile

```bash
[root@centos8 ~]# cat dockerfile
FROM nginx
RUN echo '<meta charset=utf8>用docker运行nginx服务.' > /usr/share/nginx/html/index.html
```

构建镜像

```bash
[root@centos8 ~]# docker build .
Sending build context to Docker daemon  6.755MB
Step 1/2 : FROM nginx
 ---> 904b8cb13b93
Step 2/2 : RUN echo '<meta charset=utf8>用docker运行nginx服务.' > /usr/share/nginx/html/index.html
 ---> Running in c96287227a8c
Removing intermediate container c96287227a8c
 ---> 38665a6c9e9d
Successfully built 38665a6c9e9d
```

修改镜像名

```bash
docker tag  38665a6c9e9d  bowe2_nginx
```

启动镜像

```bash
docker run -d -p 82:80 bowe2_nginx
```

![Untitled](/docker/20230508211251.png)

访问，就可以看到是我们写入的那个

## dockerfile指令

### copy

copy指令从宿主机复制文件/目录到新的一层镜像内如

```bash
copy chaoge.py   /home/

#支持多个文件，以及通配符形式复制，语法要满足Golang的filepath.Match
copy chaoge* /tmp/cc?.txt. /home/
# COPY指令能够保留源文件的元数据，如权限，访问时间等等，这点很重要
```

### add

> 特性和COPY基本一致，不过多了些功能

1.  源文件是一个URL，此时docker引擎会下载该链接 放入目标路径，且权限自动，若这不是期望结果，还得增加一层RUN
2. 源文件是一个URL，且是一个压缩包，不会自动解压，也得单独用RUN指令解压
3. 源文件是一个压缩文件，且是gzip，bzip2，xz，tar情况，ADD指令会自动解压缩该文件到目标路径

### cmd: 在容器运行命令

指定启动容器时执行的命令，每个 Dockerfile只能有一条 CMD 命令。如果指定了多条命令，只有最后一条会被执行。如果用户启动容器时候指定了运行的命令，则会覆盖掉 CMD 指定的命令。

用法，注意是双引号 `CMD["参数1"，"参数2”]` 在指定了entrypoint指令后，用CMD指定具体的**参数**。 docker不是虚拟机，容器就是一个进程，既然是进程，那么程序在启动的时候需要指定些运行参数，这就是CMD指令作用。

例如centos镜像默认的CMD是/bin/bash，直接docker run -it centos会直接进入bash解释器也可以启动容器时候，指定参数，`docker run -it centos cat /etc/s-releasea`。

CMD运行shell命令，也会被转化为shell形式 例如 CMD echo $PATH 会被转化为 CMD ["sh","-c","echo $PATH"]


{{< admonition type=warning title="前台运行"  >}} 
 要注意的是，docker不是虚拟机，虚拟机里的程序运行，基本上都是在后台运行，利用systemctl运行，**但是容器内没有后台进程的概念，容器内运行程序，必须在前台运行。**

容器就是为了主进程而存在的，主进程如果退出了，容器也就失去意义，自动退出。例如有一个经典问题 
```bash
CMD systemctl start nginx 
``` 

这样的写法是错误的，容器会立即退出 因为systemctl start nginx是希望以守护进程形式启动nginx，且CMD命令会转化为` CMD ["sh","-c","systemctl start nginx"] `这样的命令主进程是sh解释器。执行完毕后立即结束了，因此容器也就退出了。 因此正确的做法应该是

```bash
CMD["nginx”，-g”，daemon off;"]
```


{{< /admonition >}}

### endpoint

配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖。每个 Dockerfile 中只能有一个 ENTRYPOINT，当指定多个时，只有最后一个起效。

****CMD和ENDPOINT的区别****

-   CMD指令指定的容器启动时命令可以被docker run指定的命令覆盖；ENTRYPOINT指令指定的命令不能被覆盖，而是将docker run指定的参数当做ENTRYPOINT指定命令的参数。
-   CMD与ENTRYPOINT同时存在时，**CMD指令可以为ENTRYPOINT指令设置默认参数，而且CMD可以被docker run指定的参数覆盖**；

[CMD ENTRYPOINT 区别 终极解读](https://blog.csdn.net/u010900754/article/details/78526443?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2~default~CTRLIST~Rate-1-78526443-blog-125316515.pc_relevant_landingrelevant&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2~default~CTRLIST~Rate-1-78526443-blog-125316515.pc_relevant_landingrelevant&utm_relevant_index=1)

这个链接讲的不错。CMD如果传了，那么就会覆盖掉dockerfile里的CMD，dockerfile里的CMD就不会执行了。官网的解释也很详细了。不会的就去看官网文档。

[Dockerfile reference](https://docs.docker.com/engine/reference/builder/#cmd)

{{< admonition type=tip      title=小结  >}} 
总结下一般该怎么使用：一般还是会用entrypoint的中括号形式作为docker 容器启动以后的默认执行命令，里面放的是不变的部分，可变部分比如命令参数可以使用cmd的形式提供默认版本，也就是run里面没有任何参数时使用的默认参数。如果我们想用默认参数，就直接run，否则想用其他参数，就run 里面加参数。

{{< /admonition >}}


### ENV与ARG

示例
```bash
ENV NAME="yuchao"
ENV AGE="18"
ENV MYSOL_VERSION=5 .6
```

后续所有的操作，通过$NAME 就可以直接获取变量值了，维护dockerfile脚本时更友好.

{{< admonition type=tip      title=区别  >}} 
ARG和ENV一样设置环境变量 区别在于ENV 无论是在镜像构建时，还是容器运行，该变量都可以使用。ARG只是用于构建镜像需要设置的变量，容器运行时就消失了

{{< /admonition >}}

### VOLUME
示例，编写dockerfile文件
```bash
FROM centos
MAINTAINER bowen
VOLUME ["/data1","/data2"]
```

构建镜像和创建容器
```bash
docker build .
docker run ...
docker ps -a |head -2   # 查看
docker inspect 容器id
```

找到容器这两个信息

```bash
"Mounts": [
            {
                "Type": "volume",
                "Name": "37c12aa6ea2727718ad658d895482bb5edc6b8ceb49fb3bd1f1473a4d3e6a429",
                "Source": "/var/lib/docker/volumes/37c12aa6ea2727718ad658d895482bb5edc6b8ceb49fb3bd1f1473a4d3e6a429/_data",
                "Destination": "/data2",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",
                "Name": "8a21059acb2108b6528ce12b3add82b20a820dfadf066b11dc5b874f4827b14f",
                "Source": "/var/lib/docker/volumes/8a21059acb2108b6528ce12b3add82b20a820dfadf066b11dc5b874f4827b14f/_data",
                "Destination": "/data1",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
```

可以看到source就是宿主机的目录，/data是容器内的目录。这就是挂载。

-   容器数据挂载的方式，通过dockerfile，指定VOLUME目录
-   通过docker run -v 参数，直接设置需要映射挂载的目录

### EXPOSE

用于指定容器运行时对外提供的端口服务

```bash
docker port 容器
docker run -p 宿主机端口: 容器端口
docker run -P #作用是随机宿主机端口: 容器内端口
```

### WORKDIR

用于在dockerfile中，目录的切换，更改工作目录。

```bash
WORKDIR /opt
```

### USER

改变环境，用于切换用户，例如
```bash
USER root 
USER bowen
```

## 再来回顾一下构建流程

编写dockerfile文件
```bash
FROM centos:7.8.2003
RUN rpm --rebuilddb && yum install epel-release -y 
RUN rpm --rebuilddb && yum install curl -y 
ENTRYPOINT ["curl","-s","http://ipinfo.io/ip"]
```

构建容器，这一步会返回容器id，这里是可以看出来容器分层，可能会用到缓存

```bash
docker build .
```

![](/docker/20230509172100.png)

修改tag
```bash
docker tag 73a centos_curl
```

启动容器
```bash
[root@centos8 dockerfile_centos7.8]# docker run centos_curl -I
HTTP/1.1 200 OK
access-control-allow-origin: *
content-type: text/html; charset=utf-8
content-length: 14
date: Sun, 19 Mar 2023 12:33:20 GMT
x-envoy-upstream-service-time: 0
strict-transport-security: max-age=2592000; includeSubDomains
Via: 1.1 google
```
