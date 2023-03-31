---
title: "gin路由"
date: 2023-03-30
tags : [                                    
     "Go Web",
 ]
categories : [                              
     "Go语言",
 ]

---



# 普通路由

```Go
r.GET("/index", func(c *gin.Context) {...})
r.GET("/login", func(c *gin.Context) {...})
r.POST("/login", func(c *gin.Context) {...})
```

上面列出的，是普通路由，是最常用的。

# Any

有一个比较特殊的路由，Any，只要输入匹配的路径，不管是什么方法,`GET`  , `POST` ，`PUT` `, DELETE` 等，都会走到这个逻辑里，例如

```go
r.Any("/AnyMethod", func(c *gin.Context) {
    c.JSON(200, gin.H{
    "message": "这里返回any页面",
    })
})
```

![路由](/gin路由/20230330214905.png)

# Noroute

也比较特殊，用于匹配不到时返回的信息,例如

```go
r.NoRoute(func(c *gin.Context) {
		c.String(http.StatusNotFound, "没有该页面")
	})
```

也可自定义返回页面或者是信息

```go
r.NoRoute(func(c *gin.Context) {
	c.JSON(200, gin.H{
	     "message": "没有找到对应页面",
		})
  })
```

为没有配置处理函数的路由添加处理程序，默认情况下它返回404代码，下面的代码为没有匹配到路由的请求都返回`views/404.html`页面。

```go
r.NoRoute(func(c *gin.Context) {
		c.HTML(http.StatusNotFound, "views/404.html", nil)
	})
```

可以对比一下两者的区别，在正式项目里，为了美观，可以直接使用自己公司定义的not found html文件。自己定义文字还不如用原生的显示。

# 路由组

我们可以将拥有**共同URL前缀**的路由划分为一个路由组。习惯性一对{}包裹同组的路由，只是为了看着清晰，你用不用{}包裹功能上没什么区别。事实上不如不要{},看起来很烦！而且格式容易出错。

```go
r := gin.Default()
	userGroup := r.Group("/user")
	// 不用{}也可
	{
		userGroup.GET("/index", func(c *gin.Context) {
			c.JSON(200, gin.H{
				"message": "这里user的index页面",
			})
		})
		userGroup.GET("/login", func(c *gin.Context) {
			c.JSON(200, gin.H{
				"message": "这里是user的login页面",
			})
		})
	}

	shopGroup := r.Group("/shop")
	shopGroup.GET("/index", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"mesage": "这是shop的index页面",
		})
	})
	shopGroup.GET("/cart", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "这是shop的cart页面",
		})
	})
	r.Run()
```

运行，请求`http://127.0.0.1:8080/user/index`, 会返回

```go
{
    "message": "这里user的index页面"
}
```

```go
[GIN] 2023/03/31 - 09:41:47 | 200 |       958.9µs |       127.0.0.1 | GET      "/user/index"
```

可以看控制台打出的日志，即便是本地调试，还是很快的，达到了微秒级别。

路由组也是支持嵌套的，嵌套路由组,没啥意义，而且又容易弄混，知道就好，选择性忽略。例如：

```Go
shopGroup := r.Group("/shop")
	{
		shopGroup.GET("/index", func(c *gin.Context) {...})
		shopGroup.GET("/cart", func(c *gin.Context) {...})
		shopGroup.POST("/checkout", func(c *gin.Context) {...})
		// 嵌套路由组
		xx := shopGroup.Group("xx")
		xx.GET("/oo", func(c *gin.Context) {...})
	}
```

通常我们将路由分组用在划分业务逻辑或划分API版本时。

# 路由原理

Gin框架中的路由使用的是[httprouter](https://github.com/julienschmidt/httprouter)这个库。其基本原理就是构造一个路由地址的前缀树。

这个就要仔细研究大明老师的课程了！怎么去设计路由树！这个还是很重要的！

