---
title: "gin~session"
date: 2023-03-31T10:12:43+08:00
tags : [                                    
     "Go Web",
 ]
categories : [                              
     "Go语言",
 ]
author: "博文"  
---

> ✏️ 这是大明老师训练营的demo，我当时研究过，现在又自己敲了一变，差一点把自己绕进去

# demo

```Go
func main() {
	r := gin.Default()

	store := cookie.NewStore([]byte("world")) //
	r.Use(sessions.Sessions("Mysession", store))

	r.GET("/abc", func(c *gin.Context) {
		session := sessions.Default(c)
		//panic("test")
		fmt.Println(session.Get("hello")) // world ，如果是第一次请求，这里打的是nil
		//fmt.Println(session.Get("bowen"))
		if session.Get("hello") != "world" {
			session.Set("hello", "world")
			session.Save()
		} else {
			session.Set("hello", "bowen")
			session.Save()
		}
		c.JSON(200, gin.H{"message": session.Get("hello")})
	})
	r.Run(":8081")
}
```

# 分析

首先，当我们第一次请求，确保是纯第一次(就是没有cookie)，打印的肯定是nil，不等于world，会执行第一个分支，`session.Set("hello", "world")`  如下结果：

![image-20230331201128453](/gin路由/20230331201128453.png)

```go
{
    "message": "world"
}
```

第二次请求，hello对应的是world `{"hello":"world"}`, 会进入第二个分支，可以看到，打印的是`world`，接下来就置为`bowen`, 结果如下:

![image-20230331201128453](/gin路由/20230331201948.png)

```go
{
    "message": "bowen"
}
```

接下来的请求，会来回改变session，就是看到的上面结果来回切换。

# 困惑

困惑我的是，服务端停掉了，也没有采用持久化存储，那为何我重启后，再次请求，我看到第一次就打出了bowen这样的数据，百思不得其解，觉得是环境不干净导致的，于是我编译成Linux程序仍在了服务器上跑，发现一样的现象，第一次请求得到是nil。后面就不是了。

在群里讨论，这是因为数据存储到了客户端，虽然服务端没有做任何存储，但上一次请求的时候，客户端却把数据存下来了，不管是用postman，还是浏览器，都会把这个cookie存下来。然后发请求的时候就带上了。所以服务端可以拿到key对应的value。

![image-20230331201128453](/gin路由/20230331202721.png)

清空cookie，再次请求，得到就是我理解的结果。