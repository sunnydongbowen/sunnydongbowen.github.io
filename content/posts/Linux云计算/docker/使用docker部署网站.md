---
title: "使用docker部署网站"                         
author: "博文"   
date: 2023-05-10       
comments: true  
tags : [                                    
     "docker",
 ]
categories : [                              
     "Linux云计算",
 ]
---

## 编写dockerfile

```bash
FROM centos:7.8.2003
RUN curl -o /etc/yum.repos.d/CentOS-Base,repo https://mirrors.aliyun,com/repo/Centos-7.repo;
RUN curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo;
RUN yum makecache fast;
RUN yum installython3-devel python3-pip -y
RUN pip3 install -i https://pypi.douban.com/simple flask
COPY chaoge_flask.py /opt
WORKDIR /opt
EXPOSE 8080
CMD ["python3","chaoge_flask.py"]
```

![Untitled](/docker/20230510105957.png)

## 构建镜像

```bash
docker build -t 'yuchao163/my_flask_web'
```

## 基于镜像run容器

```bash
docker run -d --name my_flask web 1 -p 90:8089 yuchao163/my_flask web
```

## 修改应用


1.  修改宿主机代码及dockerfile，重新构建
2.  直接在容器内修改

```bash
docker exec -it 容器id bash
```

## 小结

其实19年在YJ的时候，我就已经自己搞过这些东西了，当时没用dockerfile，只是自己pull容器，然后把项目部署到容器里，容器访问。当时我的水平和认知不像现在，没有系统化的学习这一块。现在随着工作经验的增加，做的事情越多，理解的就越深入，**学习的也越系统。好好珍惜吧。自己走到现在很不容易。**

