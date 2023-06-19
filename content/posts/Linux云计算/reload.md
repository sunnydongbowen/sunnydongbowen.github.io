---
title: "reload命令"                         
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
修改配置文件，需要reload一下。执行下面命令reload全部
```bash
systemctl daemon-reload
```

nginx的reload
```bash
nginx -t
systemctl reload nginx
```