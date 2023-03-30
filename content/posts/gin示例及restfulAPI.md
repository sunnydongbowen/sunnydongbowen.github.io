---
title: "gin示例及restfulAPI"                         
author: "博文"      
comments: true 
 
tags : [                                    
     "Go Web",
 ]
categories : [                              
     "Go语言",
 ]
date: 2023-03-30
---

> ✏️ 观看[李文周博客](https://www.liwenzhou.com/posts/Go/gin/#autoid-0-0-0)及七米视频作的笔记，这是看的我自己notion笔记又把代码敲了一遍。

# Gin框架简介

Go世界里最流行的Web框架，[Github](https://github.com/gin-gonic/gin)上有`32K+`star。 基于[httprouter](https://github.com/julienschmidt/httprouter)开发的Web框架。 [中文文档](https://gin-gonic.com/zh-cn/docs/) 齐全，简单易用的轻量级框架。类似py里的`flask`、`FastAPI`，不完善,。不像那种大而全的框架(`Beego`,`Django`，`Spring`)，大而全的框架用起来方便，不用选。框架越完善，受限制越多，不灵活。Beego有点类似全家桶的框架。

至于beego，可以不学，它的定位，是类似Spring，Django这样的全家桶，Beego的综合性很强，里面有缓存，有orm等等，beego全包了。目前官网已经不怎么更新了，这个项目是大明维护的，看起来丰富的组件，实际上束缚了开发，用起来没有gin小巧而灵活。把gin熟练能够开发项目就可以，不需要学习那么多web框架。

gin优点是**简单**，**轻量**，**好用**，**快速**，用到什么再去集成什么。gin结合gorm，viper，zap库等等，很灵活。在Go的领域，还是很有名气的，它的源码甚至比Beego更复杂一些。

gin目前是我正儿八经学过的第一个web框架，确实很好用！不是硬性要求用别的框架，一直用gin即可。py里的web框架真的是多。Django，fastapi，flask, 还有很多！目前做web类的，我还是偏向go去做。gin不错的！看原理实现, 其他别学了; gin用的溜就已经很可以了！研究好了甚至可以给它提pr之类的！看看gin社区！

下载并安装`Gin`  记得`go mod tidy`一下。

```go
go get -u github.com/gin-gonic/gin
```

# 第一个demo

```go
func TestGindemo1(t *testing.T) {
	// 创建默认的路由引擎
	r := gin.Default()
	r.GET("/hello", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "Hello,Gin",
		})
	})
	//启动，默认是本地8080启动
	r.Run()
}
```

![image-20230330175350775](/gin示例及restfulAPI/20230330175350775.png)

运行，可以看到上面结果。同时还支持返回字符串,例如

```go
r := gin.Default()
	r.GET("/hello", func(c *gin.Context) {
		c.String(200, "hello, gin")
	})
	r.Run()
```

注意不管返回字符串还是Json数据，状态码和func函数是要的。

# RESTful API

REST与技术无关，代表的是一种软件架构风格，REST是Representational State Transfer的简称，中文翻译为“表征状态转移”或“表现层状态转化”。推荐阅读[阮一峰 理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful.html)

简单来说，REST的含义就是客户端与Web服务器之间进行交互的时候，使用HTTP协议中的4个请求方法代表不同的动作。

- `GET`用来获取资源
- `POST`用来新建资源
- `PUT`用来更新资源
- `DELETE`用来删除资源。

只要API程序遵循了REST风格，那就可以称其为`RESTful API`。目前在前后端分离的架构中，前后端基本都是通过`RESTful API`来进行交互。

例如，我们现在要编写一个管理书籍的系统，我们可以查询对一本书进行查询、创建、更新和删除等操作，在编写程序的时候就要设计客户端浏览器与我们Web服务端交互的方式和路径。按照经验我们通常会设计成如下模式(**没错，我以前做云平台开发的时候刚接触也是这么写的，还被说了**)：

| 请求方法 | url          | 含义         |
| -------- | ------------ | ------------ |
| GET      | /book        | 查询书籍信息 |
| POST     | /create_book | 创建书籍记录 |
| POST     | /update_book | 更新书籍信息 |
| POST     | /delete_book | 删除书籍信息 |

同样的需求我们按照RESTful API设计如下：

| 请求方法 | URL   | 含义         |
| -------- | ----- | ------------ |
| GET      | /book | 查询书籍信息 |
| POST     | /book | 创建书籍记录 |
| PUT      | /book | 更新书籍信息 |
| DELETE   | /book | 删除书籍信息 |

Gin框架支持开发RESTful API的开发

```go
func TestRestful(t *testing.T) {
	r := gin.Default()
	r.GET("/book", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "GET",
		})
	})

	r.POST("/book", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "POST",
		})
	})

	r.PUT("/book", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "PUT",
		})
	})

	r.DELETE("/book", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "DELETE",
		})
	})
	r.Run()
}

```

上面代码就是url虽然一样，但是根据请求的不同，执行了不同的操作。

![示例图片](/gin示例及restfulAPI/20230330203931.png)

# 补充

学习Gin框架时，可以点开源码，看一下原理，是怎么对net/http包进行封装的。核心多写多练习。后面会结合其他第三方库进行一些简单的开发。

另外补充第一个数据结构，这个是经常用的数据结构，要记得

```go
map[string]any
```

gin.H 是一个自定义类型

```go
// H is a shortcut for map[string]interface{}
type H map[string]any
```

看一下它的使用方式，这个就是类似`map[string]any{"code": 1,"msg":  "查询数据失败",}`

```go
func TestH(t *testing.T) {
	m := map[string]any{"message": "GET", "Code": 200}
	fmt.Println(m)
}
```

```Go
c.JSON(200, gin.H{
			"code": 1,
			"msg":  "查询数据失败",
		})
		return
```

关于基础的东西后面边复习边整理，例如基础数据类型和slice，map，chan等复合数据类型。