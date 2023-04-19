---
title: "Go module详细"
date: 2023-04-19
tags : [                                    
     "Go基础语法",
 ]
categories : [                              
     "Go语言",
 ]
---
<aside> 💡 极客时间专栏

</aside>

# 为当前项目添加一个依赖包

在一个项目的初始阶段，我们会经常为项目引入第三方包，并借助这些包完成特定功能。即便是项目进入了稳定阶段，随着项目的演进，我们偶尔还需要在代码中引入新的第三方包。

我们还是以上一篇中的 module-mode 项目为例。如果我们要为这个项目增加一个新依赖：[github.com/google/uuid](http://github.com/google/uuid%EF%BC%8C%E9%82%A3%E9%9C%80%E8%A6%81%E6%80%8E%E4%B9%88%E5%81%9A%E5%91%A2%EF%BC%9F)

```go
package main

import (
  "github.com/google/uuid"
  "github.com/sirupsen/logrus"
)

func main() {
  logrus.Println("hello, go module mode")
  logrus.Println(uuid.NewString())
}
```

新源码中，我们通过 import 语句导入了 [github.com/google/uuid](http://github.com/google/uuid%EF%BC%8C%E5%B9%B6%E5%9C%A8)     main 函数中调用了 uuid 包的函数 NewString。此时，如果我们直接构建这个 module，我们会得到一个错误提示：

```go
$go build
main.go:4:2: no required module provides package github.com/google/uuid; to add it:
  go get github.com/google/uuid
```

## go get

Go 编译器提示我们，go.mod 里的 require 段中，没有哪个 module 提供了 [github.com/google/uuid](http://github.com/google/uuid) 包，如果我们要增加这个依赖，可以手动执行 go get 命令。那我们就来按照提示手工执行一下这个命令：

```go
$go get github.com/google/uuid
go: downloading github.com/google/uuid v1.3.0
go get: added github.com/google/uuid v1.3.0
```

`大多数时候需要去翻墙才能下载`。除非设置了国内的proxy。你会发现，go get 命令将我们新增的依赖包下载到了本地 module 缓存里，并在 `go.mod` 文件的 require 段中新增了一行内容：

```go
require (
  github.com/google/uuid v1.3.0 //新增的依赖
  github.com/sirupsen/logrus v1.8.1
)
```

这新增的一行表明，我们当前项目依赖的是 uuid 的 v1.3.0 版本。

## go mod tidy

我们也可以使用 `go mod tidy` 命令，在执行构建前自动分析源码中的依赖变化，识别新增依赖项并下载它们：

```go
$go mod tidy
go: finding module for package github.com/google/uuid
go: found github.com/google/uuid in github.com/google/uuid v1.3.0
```

对于我们这个例子而言，手工执行 `go get` 新增依赖项，和执行 `go mod tidy` 自动分析和下载依赖项的最终效果，是等价的。但对于复杂的项目变更而言，逐一手工添加依赖项显然很没有效率，go mod tidy 是更佳的选择。

# 升级 / 降级依赖的版本

## 降级

我们先以对依赖的版本进行降级为例，分析一下。在实际开发工作中，如果我们认为 Go 命令自动帮我们确定的某个依赖的版本存在一些问题，比如，引入了不必要复杂性导致可靠性下降、性能回退等等，我们可以手工将它`降级`为之前发布的某个兼容版本。

那这个操作依赖于什么原理呢？“语义导入版本”机制。再来简单复习一下，Go Module 的版本号采用了语义版本规范，也就是版本号使用 vX.Y.Z 的格式。其中 X 是主版本号，Y 为次版本号 (minor)，Z 为补丁版本号 (patch)。主版本号相同的两个版本，较新的版本是兼容旧版本的。如果主版本号不同，那么两个版本是不兼容的。

有了语义版本号作为基础和前提，我们就可以从容地手工对依赖的版本进行升降级了，Go 命令也可以根据版本兼容性，自动选择出合适的依赖版本了。

我们还是以上面提到过的 logrus 为例，logrus 现在就存在着多个发布版本，我们可以通过下面命令来进行查询：

```go
$go list -m -versions github.com/sirupsen/logrus
github.com/sirupsen/logrus v0.1.0 v0.1.1 v0.2.0 v0.3.0 v0.4.0 v0.4.1 v0.5.0 v0.5.1 v0.6.0 v0.6.1 v0.6.2 v0.6.3 v0.6.4 v0.6.5 v0.6.6 v0.7.0 v0.7.1 v0.7.2 v0.7.3 v0.8.0 v0.8.1 v0.8.2 v0.8.3 v0.8.4 v0.8.5 v0.8.6 v0.8.7 v0.9.0 v0.10.0 v0.11.0 v0.11.1 v0.11.2 v0.11.3 v0.11.4 v0.11.5 v1.0.0 v1.0.1 v1.0.3 v1.0.4 v1.0.5 v1.0.6 v1.1.0 v1.1.1 v1.2.0 v1.3.0 v1.4.0 v1.4.1 v1.4.2 v1.5.0 v1.6.0 v1.7.0 v1.7.1 v1.8.0 v1.8.1
```

在这个例子中，基于初始状态执行的 `go mod tidy` 命令，帮我们选择了 logrus 的最新发布版本 v1.8.1。如果你觉得这个版本存在某些问题，想将 logrus 版本降至某个之前发布的兼容版本，比如 v1.7.0，那么我们可以在项目的 module 根目录下，执行`带有版本号的 go get` 命令：

```go
$go get github.com/sirupsen/logrus@v1.7.0
go: downloading github.com/sirupsen/logrus v1.7.0
go get: downgraded github.com/sirupsen/logrus v1.8.1 => v1.7.0
```

当然我们也可以使用万能命令 `go mod tidy` 来帮助我们降级，但前提是首先要用 go mod edit 命令，明确告知我们要依赖 v1.7.0 版本，而不是 v1.8.1，这个执行步骤是这样的：

```go
$go mod edit -require=github.com/sirupsen/logrus@v1.7.0
$go mod tidy       
go: downloading github.com/sirupsen/logrus v1.7.0
```

降级后，我们再假设 logrus v1.7.1 版本是一个安全补丁升级，修复了一个严重的安全漏洞，而且我们必须使用这个安全补丁版本，这就意味着我们需要将 logrus 依赖从 v1.7.0 升级到 v1.7.1。

## 升级

我们可以使用与降级同样的步骤来完成升级，这里我只列出了使用 go get 实现依赖版本升级的命令和输出结果，你自己动手试一下。

```go
$go get github.com/sirupsen/logrus@v1.7.1
go: downloading github.com/sirupsen/logrus v1.7.1
go get: upgraded github.com/sirupsen/logrus v1.7.0 => v1.7.1
```

但是你可能会发现一个问题，在前面的例子中，Go Module 的依赖的主版本号都是 1。根据我们上节课中学习的语义导入版本的规范，在 Go Module 构建模式下，当依赖的主版本号为 0 或 1 的时候，我们在 Go 源码中导入依赖包，不需要在包的导入路径上增加版本号，也就是：

```go
import github.com/user/repo/v0 等价于 import github.com/user/repo
import github.com/user/repo/v1 等价于 import github.com/user/repo
```

# 添加一个主版本号大于 1 的依赖

语义导入版本机制有一个原则：如果新旧版本的包使用相同的导入路径，那么新包与旧包是兼容的。也就是说，如果新旧两个包不兼容，那么我们就应该采用不同的导入路径

按照语义版本规范，如果我们要为项目引入主版本号大于 1 的依赖，比如 v2.0.0，那么由于这个版本与 v1、v0 开头的包版本都不兼容，我们在导入 v2.0.0 包时，不能再直接使用 [github.com/user/repo](http://github.com/user/repo%EF%BC%8C%E8%80%8C%E8%A6%81%E4%BD%BF%E7%94%A8%E5%83%8F%E4%B8%8B%E9%9D%A2%E4%BB%A3%E7%A0%81%E4%B8%AD%E9%82%A3%E6%A0%B7%E4%B8%8D%E5%90%8C%E7%9A%84%E5%8C%85%E5%AF%BC%E5%85%A5%E8%B7%AF%E5%BE%84%EF%BC%9A)

而要使用像下面代码中那样不同的包导入路径

```go
import github.com/user/repo/v2/xxx
```

也就是说，如果我们要为 Go 项目添加主版本号大于 1 的依赖，我们就需要使用“`语义导入版本`”机制，在声明它的导入路径的基础上，加上版本号信息。我们以“向 module-mode 项目添加 [github.com/go-redis/redis](http://github.com/go-redis/redis) 依赖包的 v7 版本”为例，看看添加步骤。

首先，我们在源码中，以空导入的方式导入 v7 版本的 [github.com/go-redis/redis](http://github.com/go-redis/redis) 包：

```go
package main

import (
  _ "github.com/go-redis/redis/v7" // “_”为空导入
  "github.com/google/uuid"
  "github.com/sirupsen/logrus"
)

func main() {
  logrus.Println("hello, go module mode")
  logrus.Println(uuid.NewString())
}
```

接下来的步骤就与添加兼容依赖一样，我们通过 go get 获取 redis 的 v7 版本：

```go
$go get github.com/go-redis/redis/v7
go: downloading github.com/go-redis/redis/v7 v7.4.1
go: downloading github.com/go-redis/redis v6.15.9+incompatible
go get: added github.com/go-redis/redis/v7 v7.4.1
```

我们可以看到，go get 为我们选择了 go-redis v7 版本下当前的最新版本 v7.4.1。

不过呢，这里说的是为项目添加一个主版本号大于 1 的依赖的步骤。有些时候，出于要使用依赖包最新功能特性等原因，我们可能需要将某个依赖的版本升级为其不兼容版本，也就是主版本号不同的版本，这又该怎么做呢？

# 升级依赖版本到一个不兼容版本

有些时候，出于要使用依赖包最新功能特性等原因，我们可能需要将某个依赖的版本升级为其不兼容版本，也就是主版本号不同的版本，这又该怎么做呢？

我们还以 go-redis/redis 这个依赖为例，将这个依赖从 v7 版本升级到最新的 v8 版本看看。

按照语义导入版本的原则，不同主版本的包的导入路径是不同的。所以，同样地，我们这里也需要先将代码中 redis 包导入路径中的版本号改为 v8

```go
import (
  _ "github.com/go-redis/redis/v8"
  "github.com/google/uuid"
  "github.com/sirupsen/logrus"
)
```

接下来，我们再通过 go get 来获取 v8 版本的依赖包

```go
$go get github.com/go-redis/redis/v8
go: downloading github.com/go-redis/redis/v8 v8.11.1
go: downloading github.com/dgryski/go-rendezvous v0.0.0-20200823014737-9f7001d12a5f
go: downloading github.com/cespare/xxhash/v2 v2.1.1
go get: added github.com/go-redis/redis/v8 v8.11.1
```

这样，我们就完成了向一个不兼容依赖版本的升级。是不是很简单啊！

# 移除一个依赖

我们还是看前面 go-redis/redis 示例，如果我们这个时候不需要再依赖 go-redis/redis 了，你会怎么做呢？

你可能会`删除掉代码`中对 redis 的空导入这一行，之后再利用 go build 命令成功地构建这个项目。

但你会发现，与添加一个依赖时 Go 命令给出友好提示不同，这次 go build 没有给出任何关于项目已经将 go-redis/redis 删除的提示，并且 go.mod 里 require 段中的 go-redis/redis/v8 的依赖依旧存在着。

我们再通过 go list 命令列出当前 module 的所有依赖，你也会发现 go-redis/redis/v8 仍出现在结果中：

```go
$go list -m all
github.com/bigwhite/module-mode
github.com/cespare/xxhash/v2 v2.1.1
github.com/davecgh/go-spew v1.1.1
... ...
github.com/go-redis/redis/v8 v8.11.1
... ...
gopkg.in/yaml.v2 v2.3.0
```

这是怎么回事呢？其实，要想彻底从项目中移除 go.mod 中的依赖项，仅从源码中删除对依赖项的导入语句还不够。这是因为如果源码满足成功构建的条件，go build 命令是不会“多管闲事”地清理 go.mod 中多余的依赖项的。

那正确的做法是怎样的呢？我们还得用 `go mod tidy` 命令，将这个依赖项彻底从 Go Module 构建上下文中清除掉。go mod tidy 会自动分析源码依赖，而且将不再使用的依赖从 go.mod 和 go.sum 中移除。

# Vendor

{不理解}，老技术了。现在除非是很老的项目采用这个，新项目大都是go module模式不要管它。

到这里，其实我们已经分析了 Go Module 依赖包管理的 5 个常见情况了，但其实还有一种特殊情况，需要我们借用 vendor 机制。

你可能会感到有点奇怪，为什么 Go Module 的维护，还有要用 vendor 的情况？

其实，vendor 机制虽然诞生于 GOPATH 构建模式主导的年代，但在 Go Module 构建模式下，它依旧被保留了下来，并且成为了 Go Module 构建机制的一个很好的补充。特别是在一些不方便访问外部网络，并且对 Go 应用构建性能敏感的环境，比如在一些内部的持续集成或持续交付环境（CI/CD）中，使用 vendor 机制可以实现与 Go Module 等价的构建。

和 GOPATH 构建模式不同，Go Module 构建模式下，我们再也无需手动维护 vendor 目录下的依赖包了，Go 提供了可以快速建立和更新 vendor 的命令，我们还是以前面的 module-mode 项目为例，通过下面命令为该项目建立 vendor：

```go
$go mod vendor
$tree -LF 2 vendor
vendor
├── github.com/
│   ├── google/
│   ├── magefile/
│   └── sirupsen/
├── golang.org/
│   └── x/
└── modules.txt
```

我们看到，go mod vendor 命令在 vendor 目录下，创建了一份这个项目的依赖包的副本，并且通过 vendor/modules.txt 记录了 vendor 下的 module 以及版本。

如果我们要基于 vendor 构建，而不是基于本地缓存的 Go Module 构建，我们需要在 go build 后面加上 -mod=vendor 参数。

在 Go 1.14 及以后版本中，如果 Go 项目的顶层目录下存在 vendor 目录，那么 go build 默认也会优先基于 vendor 构建，除非你给 go build 传入 -mod=mod 的参数。

# 总结

-   通过 `go get` 我们可以升级或降级某依赖的版本，如果升级或降级前后的版本不兼容，这里千万注意别忘了变化包导入路径中的版本号，这是 Go 语义导入版本机制的要求；
-   通过 `go mod tidy`，我们可以自动分析 Go 源码的依赖变更，包括依赖的新增、版本变更以及删除，并更新 go.mod 中的依赖信息。
-   通过 `go mod vendor`，我们依旧可以支持 vendor 机制，并且可以对 vendor 目录下缓存的依赖包进行自动管理。