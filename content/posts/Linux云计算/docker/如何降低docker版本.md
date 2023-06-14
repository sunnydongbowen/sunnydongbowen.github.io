---
title: "降低docker版本"                         
author: "博文"   
date: 2023-06-14    
comments: true  
tags : [                                    
     "docker",
 ]
categories : [                              
     "Linux云计算",
 ]
---
在安装openstack，执行自己写的自动化安装脚本后，在docker.service里配置了`etcd`后，发现docker起不来
```bash
[root@os19 docker]# yum list installed | grep docker
Repository epel is listed more than once in the configuration
containerd.io.x86_64                               1.6.6-3.1.el8                                 @docker-nightly
docker-buildx-plugin.x86_64                        0.8.2-1.el8                                   @docker-test 
docker-ce.x86_64                                   3:22.06.0~beta.0-1.el8                        @docker-test 
docker-ce-cli.x86_64                               1:22.06.0~beta.0-1.el8                        @docker-test 
docker-ce-rootless-extras.x86_64                   22.06.0~beta.0-1.el8                          @docker-test 
docker-compose-plugin.x86_64                       2.6.0-3.el8                                   @docker-stable
docker-scan-plugin.x86_64                          0.17.0-3.el8                                  @docker-stable
```

查看系统日志后发现新版本不支持etcd这个启动参数了

```bash
Jun  8 19:07:05 os11 dockerd[195152]: Host-discovery and overlay networks with external k/v stores are deprecated. The 'cluster-advertise', 'cluster-store', and 'cluster-store-opt' options have been removed
Jun  8 19:07:05 os11 systemd[1]: docker.service: Main process exited, code=exited, status=1/FAILURE
Jun  8 19:07:05 os11 systemd[1]: docker.service: Failed with result 'exit-code'.
Jun  8 19:07:05 os11 systemd[1]: Failed to start Docker Application Container Engine.
Jun  8 19:07:07 os11 systemd[1]: docker.service: Service RestartSec=2s expired, scheduling restart.
```

查看正常机器的版本

```bash
[root@os55 ~]# yum list installed | grep docker
containerd.io.x86_64                              1.4.3-3.1.el8                                 @docker-nightly
docker-ce.x86_64                                  3:20.10.3-3.el8                               @docker-stable
docker-ce-cli.x86_64                              1:20.10.3-3.el8                               @docker-stable
docker-ce-rootless-extras.x86_64                  20.10.3-3.el8                                 @docker-stable
[root@os55 ~]# rpm -qa|grep docker
docker-ce-cli-20.10.3-3.el8.x86_64
docker-ce-20.10.3-3.el8.x86_64
docker-ce-rootless-extras-20.10.3-3.el8.x86_64
```

降低docker版本

```bash
rpm -e docker-buildx-plugin-0:0.8.2-1.el8.x86_64 # 要卸载，不然降低失败
yum downgrade --setopt=obsoletes=0 -y docker-ce-20.10.3-3.el8.x86_64  docker-ce-cli-20.10.3-3.el8.x86_64  containerd.io-1.4.3-3.1.el8
```

[参考链接](https://blog.csdn.net/jiongshang3743/article/details/118998571)