---
title: "Go module初步"
date: 2023-04-19
tags : [                                    
     "Go基础语法",
 ]
categories : [                              
     "Go语言",
 ]
---

## 1. 引入go module

```go
package main

import (
  "github.com/valyala/fasthttp"
  "go.uber.org/zap"
)

var logger *zap.Logger

func init() {
  logger, _ = zap.NewProduction()
}

func fastHTTPHandler(ctx *fasthttp.RequestCtx) {
  logger.Info("hello, go module", zap.ByteString("uri", ctx.RequestURI()))
}

func main() {
  fasthttp.ListenAndServe(":8081", fastHTTPHandler)
}
```

这个示例创建了一个在 8081 端口监听的 http 服务，当我们向它发起请求后，这个服务会在终端标准输出上输出一段访问日志。

你会看到，和“hello，world“相比，这个示例显然要复杂许多。但不用担心，你现在大可不必知道每行代码的功用，你只需要我们在这个稍微有点复杂的示例中引入了两个第三方依赖库，zap 和 fasthttp 就可以了。我们尝试一下使用编译“hello，world”的方法来编译“hellomodule”中的 main.go 源文件，go 编译器的输出结果是这样的：
```go
$go build main.go main.go:4:2: no required module provides package github.com/valyala/fasthttp: go.mod file not found in current directory or any parent directory; see 'go help modules' main.go:5:2: no required module provides package go.uber.org/zap: go.mod file not found in current directory or any parent directory; see 'go help modules'
```



##  2. go module

Go module 构建模式是在 Go 1.11 版本正式引入的，为的是彻底解决 Go 项目复杂版本依赖的问题，在 Go 1.16 版本中，Go module 已经成为了 Go 默认的包依赖管理机制和 Go 源码构建机制。

Go Module 的核心是一个名为 go.mod 的文件，在这个文件中存储了这个 module 对第三方依赖的全部信息。接下来，我们就通过下面命令为“hello，module”这个示例程序添加 go.mod 文件：
```go
$go mod init github.com/bigwhite/hellomodule
go: creating new go.mod: module github.com/bigwhite/hellomodule
go: to add module requirements and sums:go mod tidy
```

你会看到，go mod init 命令的执行结果是在当前目录下生成了一个 go.mod 文件：
```go
$cat go.mod
module github.com/bigwhite/hellomodule
go 1.16

```

其实，一个 module 就是一个包的集合，这些包和 module 一起打版本、发布和分发。go.mod 所在的目录被我们称为它声明的 module 的根目录。

不过呢，这个时候的 go.mod 文件内容还比较简单，第一行内容是用于声明 module 路径（module path）的。而且，module 隐含了一个命名空间的概念，module 下每个包的导入路径都是由 module path 和包所在子目录的名字结合在一起构成。

比如，如果 hellomodule 下有子目录 pkg/pkg1，那么 pkg1 下面的包的导入路径就是由 module path（[github.com/bigwhite/hellomodule）和包所在子目录的名字（pkg/pkg1）结合而成，也就是](http://github.com/bigwhite/hellomodule%EF%BC%89%E5%92%8C%E5%8C%85%E6%89%80%E5%9C%A8%E5%AD%90%E7%9B%AE%E5%BD%95%E7%9A%84%E5%90%8D%E5%AD%97%EF%BC%88pkg/pkg1%EF%BC%89%E7%BB%93%E5%90%88%E8%80%8C%E6%88%90%EF%BC%8C%E4%B9%9F%E5%B0%B1%E6%98%AF) [github.com/bigwhite/hellomodule/pkg/pkg1。](http://github.com/bigwhite/hellomodule/pkg/pkg1%E3%80%82)

另外，go.mod 的最后一行是一个 Go 版本指示符，用于表示这个 module 是在某个特定的 Go 版本的 module 语义的基础上编写的。

这个再重新构建，在构建之前，需要go mod tidy 下载这两个第三方库，再重新构建就好了。

完成后，go.mod 已经记录了 hellomodule 直接依赖的包的信息。不仅如此，hellomodule 目录下还多了一个名为 go.sum 的文件，这个文件记录了 hellomodule 的直接依赖和间接依赖包的相关版本的 hash 值，用来校验本地包的真实性。在构建的时候，如果本地依赖包的 hash 值与 go.sum 文件中记录的不一致，就会被拒绝构建。

有了 go.mod 以及 hellomodule 依赖的包版本信息后，我们再来执行构建：

```go
$go build main.go
$ls
go.mod    go.sum    main*    main.go
```

这次我们成功构建出了可执行文件 main，运行这个文件，新开一个终端窗口，在新窗口中使用 curl 命令访问该 http 服务：curl localhost:8081/foo/bar，我们就会看到服务端输出如下日志：

```go
$./main
{"level":"info","ts":1626614126.9899719,"caller":"hellomodule/main.go:15","msg":"hello, go module","uri":"/foo/bar"}
```

这下，我们的“ hellomodule”程序可算创建成功了。我们也看到使用 Go Module 的构建模式，go build 完全可以承担其构建规模较大、依赖复杂的 Go 项目的重任。还有更多关于 Go Module 的内容，这个后面有说。下面是我随便打开的我本地的项目目录所展示的内容：

![](/go基础/20230419113746.png)

go.sum内容

![](/go基础/20230419113927.png)



## 3. 几个不错的问题
{{< admonition type=question title="问题1"    >}} 
>如何import自己在本地创建的module，在这个module还没有发布到GitHub的情况下？

作者回复: 非常好的问题。go module机制在您提到的工作场景下目前的体验做的还不够好。在Go 1.17版本及之前版本的解决方法是使用go mod的replace指示符(directive)。假如你的module a要import的module b将发布到github.com/user/repo中，那么你可以手动在module的go.mod中的require块中手工加上一条

require [github.com/user/repo](http://github.com/user/repo) v1.0.0

注意v1.0.0这个版本号是一个临时的版本号。然后在module a的go.mod中使用replace将上面对module b的require替换为本地的module b:

replace [github.com/user/repo](http://github.com/user/repo) v1.0.0 => module b本地路径

这样go命令就会使用你本地正在开发、尚未提交github的module b了。

这个问题，不是我想那样，当成包引入。知道吗。但并不难理解。
{{< /admonition >}}

{{< admonition type=question title="问题2"    >}} 
>如果 **路径和包名**不一样， `path: apath`, `package: apack`   那么使用的时候是这样吗？` import "apath"`  ` apack.Print()`

这个是对的！但一般不这样用，一般都是一致的。import的是路径，而package 声明的是包名，二者是可以不一样的。
{{< /admonition >}}

{{< admonition type=question title="问题3"    >}} 
>go 引用了其他包的话，是将引用的包都编译进去。我用 ldd 才看几个 go 编译出来的二进制程序都是没有动态链接库的使用。但是，在看其他几个 go 编译出来的二进制程序时（比如 containerd、ctr，它们都是用 go 编写的），又有引用动态链接库，这个是为什么？

go默认是开启CGO_ENABLED的，即`CGO_ENABLED=1`。但编译出来的二进制程序究竟有无动态链接，取决于你的程序使用了什么包。如果就是一个hello，world，那么你编译出来的将是一个纯静态程序。

如果你依赖了网络包或一些系统包，比如用http包编写了一个web server，那么你编译出来的二进制程序又会是一个包含动态链接的程序。

原因就在于目前的go标准库中，某些功能具有两份实现，一份是c语言实现的，一份是go语言实现的。在cgo_enable开启的情况下，go链接器会链接c语言的版本，于是就有了依赖动态链接库的情况。如果你将cgo_enabled置为0，你再重新编译链接，那么go链接器会使用go版本的实现，这样你将得到一个没有动态链接的纯静态二进制程序。
{{< /admonition >}}



{{< admonition type=question title="问题4"    >}} 
1.  go module和go package有什么区别和联系？
2.  go run只是go build和执行可执行文件这俩步的一个快捷方式吗？还是有一些特殊的处理

回答：
1.  关于go module后面有详细讲解，这里简单说一下。go module是一个逻辑概念，它由一个或一组package组成，这组package的版本由module的版本决定。这组packet的导入路径也基于module path。实际我们在包中导入的包都是module下的某个包，我们在go.mod中的依赖则用module及module版本表述。
2.  go run 就是build+run。
{{< /admonition >}}

{{< admonition type=question title="问题5"    >}} 
>go module管理，有了这种显式的依赖，是不是连makefile都不需要了，感觉go build可以实现类似make file的功能？

不用makefile等第三方构建辅助工具是Go核心团队的目标。
但说实话，在实际生产项目中，项目不仅仅只是构建(go build)与跑测试(go test)操作，很可能还有制作镜像，上传镜像等，这些操作go命令是“鞭长莫及”的，这时候，我们还是需要依赖make等第三方的项目管理工具。
我个人在生产项目中就一直在使用make工具。
{{< /admonition >}}