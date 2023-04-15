---
title: "http"
date: 2023-04-02T10:43:05+08:00
tags : [                                    
     "Go Web",
 ]
categories : [                              
     "Go语言",
 ]
author: "博文" 
---

>  ✏️ 阅读汪明Go并发编程实战第11章做的笔记

# net/http简介

Go语言中对web服务器端的编程提供了很好的支持，标准库中的`net/http包`提供了很多Web服务相关的`API`，可以帮助开发人员快速构建web服务。这个包是很重要的包，虽然我们在开发web项目中，大都使用框架，但go的gin，beego，iris等框架还是基于这个库封装的。

下面简单介绍一下能提供web服务的listenAndServe函数，它在go源码中的net/http/server.go中定义

```go
// ListenAndServe listens on the TCP network address addr and then calls
// Serve with handler to handle requests on incoming connections.
// Accepted connections are configured to enable TCP keep-alives.
//
// The handler is typically nil, in which case the DefaultServeMux is used.
//
// ListenAndServe always returns a non-nil error.
func ListenAndServe(addr string, handler Handler) error {
	server := &Server{Addr: addr, Handler: handler}
	return server.ListenAndServe()
}
```

该函数用于在特定的TCP网络地址addr(第一个参数)上进行监听，然后调用服务器端处理流程，handler(第二个参数)来处理传入的连接请求。handler通常为nil，这意味着调用默认的` DefaultServeMux `进行处理，默认的 `DefaultServeMux` 会自动注册用户定义的客户端逻辑处理程序。

# 示例

```go
type Data struct {
	Msg string
}

func handleInex(w http.ResponseWriter, r *http.Request) {
	t, _ := template.ParseFiles("index.html")
	data := Data{"hello http"}
	t.Execute(w, data)
}

func handleText(w http.ResponseWriter, r *http.Request) {
	if r.Method == "GET" {
		vars := r.URL.Query()
		key, ok := vars["key"]
		if ok {
			msg := "hello get " + key[0]
			w.Write([]byte(msg))
		} else {
			w.Write([]byte("hello world"))
		}
	}
	if r.Method == "POST" {
		r.ParseForm()
		key := r.Form.Get("name")
		msg := "hello post" + key
		w.Write([]byte(msg))
	}
}

func TestHttp(t *testing.T) {
	http.HandleFunc("/", handleText)
	http.HandleFunc("/index", handleIndex)
	err := http.ListenAndServe("127.0.0.1:8080", nil)
	if err != nil {
		fmt.Println(err)
	}
}

```

{{< admonition type=info title="运行结果"  >}}
我们启动这个程序，用postman请求这两个路径，分别看一下结果{{< /admonition >}}

当请求`http://127.0.0.1:8080/`且是`get请求`时，如果没有找到`key`这个参数，会返回给客户端

![](/http/20230402115502.png)

当请求`http://127.0.0.1:8080/?key=博文`且是`get请求` 时，找到了这个参数，会返回给客户端

![](/http/20230402115834.png)

当请求是`http://127.0.0.1:8080/?name=bowen` 且是`post请求` ,且能找到name参数时，会返回客户端

![](/http/20230402120316.png)

当请求是`http://127.0.0.1:8080/?key=bowen` 且是`post请求` ,且能找不到name参数时，会返回客户端

![](/http/20230402120521.png)

当请求`http://127.0.0.1:8080/index`  时，给我们返回了自己定义的页面

![](/http/20230402120823.png)

# 分析

{{< admonition type=info title="分析"  >}}
上面可以看到，结果是按照程序中设定的运行的，下面分析一下这个看似简单的程序{{< /admonition >}}

##  http.ListenAndServe("", nil)

```
http.ListenAndServe("127.0.0.1:8080", nil)
```

首先导入`net/http包`，这个包中有提供web服务的核心API,`http.ListenAndServe("127.0.0.1:8080", nil)` 语句在地址127.0.0.1:8080上监听请求，要注意端口号不要忘记写了，我第一次运行的时候没有写端口号，一下子运行完成，也没有报错。第二个参数为nil，意味着调用默认的 DefaultServeMux 进行处理。

```go
// HandleFunc registers the handler function for the given pattern
// in the DefaultServeMux.
// The documentation for ServeMux explains how patterns are matched.
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}
```

## http.HandleFunc("/", func(w , r)

```Go
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request)
```

这个语句定义一个路由请求处理程序，`http.HandleFunc` 接收两个参数：

第一个参数是HTTP请求的目标路径`“/”`,该参数可以是字符串，也可以是字符串形式的正则表达式。`“/”`表示处理`http://127.0.0.1:8080/`  下的请求

  第二个是具体的`回调方法`，响应客户端请求。`http.HandleFunc` 具有路由URL功能，可以根据路由定义来进行不同的逻辑处理，`http.Request`  代表客户端请求，可以获取很多信息，比如客户端请求的方法(GET或POST等)以及客户端请求的参数等。

## r.URL.Query()

如果是GET请求，可以通过r.URL.Query()  方法获取客户端Query参数，它是一个Values类型，本质上是一个`map[string][]string` ，因此可以用[‘参数名’]来获取具体的参数的值。例如我们正常url输入，匹配到了程序中的if语句，会get到key的值。如果获取的Query参数没有错误，则将返回一个[]string参数类型的值，获取值后拼接后返回一个字符串。

## http.Request(参数)

是一个结构体，可以点开看一下里面的参数。其中Form存储了POST，GET，PUT参数，在使用之前需要调用ParseForm方法。PostForm存储了POST和PUT参数，使用之前也需要调用ParseForm方法。MultipartForm传的是表单post参数，在使用前需要调用ParseMultipartForm方法。这个点开源码看一下

## http.ResponseWriter(参数)

{{< admonition type=info title="http.ResponseWriter(参数)"  >}}
是一个接口，定义如下,更具体的可以点开源码看一下说明，说明写的极其详细:{{< /admonition >}}

```go
type ResponseWriter interface {
	Header() Header
	Write([]byte) (int, error)
	WriteHeader(statusCode int)
}
```

`w.Write([]byte(msg))` 语句返回数据到客户端，由于msg是string类型，因此需要强制类型转换。

上面程序还是很经典的，要熟练掌握。

## 模板渲染

{{< admonition type=info title="模板渲染"  >}}
上面的示例中，涉及到了一个简单的html页面，内容如下：{{< /admonition >}}

```html
<body>
<h1>Hello Go</h1>
<h1>{{.Msg}}</h1>
</body>
```

其中`{{.Msg}}`  是一个模板内容, handle函数如下：

```go
func handleIndex(w http.ResponseWriter, r *http.Request) {
	t, _ := template.ParseFiles("http/index.html")
	data := Data{"hello http"} 
	t.Execute(w, data)
}
```

上面的程序中，首先导入html/template包，支持html模板的解析，函数handleIndex负责读取和解析index.html文件并返回客户端。

`data := Data{"hello http"}` 实例化结构体data

`t.Execute(w, data)`  对读取的html模板进行解析， 并将字段Msg值替换到{{.Msg}}中

注意`template.ParseFiles()`，可以解析多个文件，如果不同目录有相同的文件名，那么剩下的生效的只有最后一个。`template.ParseFiles("http/index.html"，"http1/index.html")`，虽然在实际编程中一般都是确定的，但也要警惕这个坑。

# 小结

定义handler

```go
func handleText(w http.ResponseWriter, r *http.Request){}
func handleIndex(w http.ResponseWriter, r *http.Request){}
```

使用

```go
func main() {
  // 调用方式
	http.HandleFunc("/", handleText)
	http.HandleFunc("/index", handleIndex)
	// 一定要写端口号，不然程序一下子就运行完了
	err := http.ListenAndServe("127.0.0.1:8080", nil)
	if err != nil {
		fmt.Println(err)
	}
}
```

可以看到，基本使用步骤

- 定义handle函数
- http.HandleFunc()
- 启动服务端