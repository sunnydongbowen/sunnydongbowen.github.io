---
title: "gin~middleware"
date: 2023-03-31T10:09:12+08:00
tags : [                                    
     "Go Web",
 ]
categories : [                              
     "Go语言",
 ]
 
author: "博文"  
---

Gin框架允许开发者在处理请求的过程中，加入用户自己的钩子（Hook）函数。这个钩子函数就叫中间件，中间件适合处理一些公共的业务逻辑，比如**登录认证、权限校验、数据分页、记录日志、耗时统计** 等。

# 定义中间件

中间件用的还是比较多的，比如，我们创建路由引擎时 `r := gin.Default()`  就默认了两个中间件。

```go

// Default returns an Engine instance with the Logger and Recovery middleware already attached.
func Default() *Engine {
	debugPrintWARNINGDefault()
	engine := New()
	engine.Use(Logger(), Recovery())
	return engine
}
```

{{< admonition  title="示例"  >}}
中间件函数示例
{{< /admonition >}}

```go
func StatCosts(format string) gin.HandlerFunc {
	return func(c *gin.Context) {
		start := time.Now()
		//闭包的用法，内层函数使用外层函数的变量
		log.Println(start.Format(format))
		c.Set("name", "bowen")
		// 调用该请求的剩余处理程序
		c.Next()
		//计算耗时
		cost := time.Since(start)
		log.Println(cost)
	}
}
```

这是一个统计执行时间的中间件，带有格式化时间的参数。我们为它写一个测试程序,来看一下注册中间件的过程

# 注册中间件

## 全局中间件

```go
// 新建一个没有任何默认中间件的路由
r := gin.New()
// 注册一个全局中间件
r.Use(StatCosts("2006-01-02 15:04:05"))
r.GET("/test", func(c *gin.Context) {
	name := c.MustGet("name").(string)
	log.Println(name)
	c.JSON(http.StatusOK, gin.H{
		"message": "hello world",
	})
})
r.Run()
```

输入请求，`http://127.0.0.1:8080/test`

![/中间件/中间件.png](/中间件/中间件.png)

可以看到，按照我们传的参数改了时间格式，拿到了中间件里面set的数据，执行了路由函数，打印时间。

{{< admonition type=tip title="注意点"  >}}
全局注册，不管请求哪个url路径，都会执行这个中间件
{{< /admonition >}}

## 为某个路由注册

{{< admonition  title="局部路由注册"  >}}
局部中间件，我们可以针对不同的url，选择是否需要中间件，及执行哪一个中间件
{{< /admonition >}}

```go
func TestLocal(t *testing.T) {
	r := gin.Default()
	r.GET("/test", StatCosts("2006-01-02 15:04:05"), func(c *gin.Context) {
		name := c.MustGet("name").(string)
		log.Println(name)
		c.JSON(http.StatusOK, gin.H{
			"message": "hello world",
		})
	})

	r.GET("/xyz", StatCosts("2006-01-02 15:04:05"), func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"message": "hello xyz",
		})
	})
	r.Run()
}
```

![/中间件/中间件.png](/中间件/20230401172130.png)

## 某个路由不用执行中间件

{{< admonition type=tip title="某个路由不执行中间件"  >}}
比如100个路由，但有一个不需要或者我不想让它去执行中间件，给它一些特殊对待，可以选择跳过 ，只需要在中间件代码里去判断这个路由跳过即可。{{< /admonition >}}

```go
func StatCost() gin.HandlerFunc {
	return func(c *gin.Context) {
		if c.Request.URL.Path == "/xyz" {
			c.Next()
			//c.Abort()
			return
			// abort()终端当前的handlerfunc
		}
		start := time.Now()
		// 闭包的用法，内层函数使用外层函数的变量 format string,可以传参数
		//log.Println(start.Format(format))

		c.Set("name", "bowen")
		// 调用该请求的剩余处理程序
		c.Next()
		//计算耗时
		cost := time.Since(start)
		log.Println(cost)
	}
}
```

我们启动服务端，再次请求，就不会执行下面的统计执行时间的过程了。

## 路由组注册中间件

{{< admonition  title="路由组注册中间件"  >}}
当我们有共同前缀的路径需要注册中间件时，可以使用路由组注册中间件的方式{{< /admonition >}}

> 路由组注册中间件方式1

```go
shopGroup := r.Group("/shop", StatCost())
{
    shopGroup.GET("/index", func(c *gin.Context) {...})
    ...
}
```

> 路由组注册中间件方式2

```go
shopGroup := r.Group("/shop")
shopGroup.Use(StatCost())
{
    shopGroup.GET("/index", func(c *gin.Context) {...})
    ...
}
```

# iris中间件引用

{{< admonition type=info title="iris中间件引用"  >}}
这是群里在讨论时发出来的iris框架中间件的代码，iris在国内公司用的不多，放到这里，是与gin的中间件做对比。{{< /admonition >}}

![/中间件/中间件.png](/中间件/20230401201245.png)

# 小结

中间件是很重要的一部分，后面会结合实际例子去做，把gin里面学过基础demo结合和gorm，zap等，当这些看似简单的知识点组合起来的时候，就是一个简单的项目了。再复杂的项目，都是由一个个小的知识点积累起来的。框架本身没什么难的，是手写框架！可以帮我们更深的理解。

- 单个知识点，理解小demo
- 组合其他知识点做综合一点的项目
- 手写中间件框架，理解设计原理

