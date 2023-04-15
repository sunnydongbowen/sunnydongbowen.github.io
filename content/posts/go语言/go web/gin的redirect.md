---
title: "gin~redirect"
date: 2023-03-31T10:09:30+08:00
tags : [                                    
     "Go Web",
 ]
categories : [                              
     "Go语言",
 ]
author: "博文"  
---

> ✏️ 重定向主要分为两种，http重定向和路由重定向，在日常web开发中经常用到的，要掌握。

# HTTP重定向

```go
r := gin.Default()
r.GET("/hello", func(c *gin.Context) {
c.JSON(200, gin.H{
	"message": "hello gin",
   })
})

r.GET("/abc", func(c *gin.Context) {
	c.Redirect(http.StatusMovedPermanently, "http://www.baidu.com")
   })
r.Run()
```

输入` http://127.0.0.1:8080/abc` , 会帮我们重定向到百度

# 路由重定向

路由重定向，使用`HandleContext`，这个例子注意**浏览器缓存**，可以用postman测试

```go
r.GET("/hello", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "hello gin",
		})
	})

r.GET("/abc", func(c *gin.Context) {
    c.Request.URL.Path = "/hello"
    r.HandleContext(c)
})
r.Run()
```

![](/gin路由/20230331113140.png)