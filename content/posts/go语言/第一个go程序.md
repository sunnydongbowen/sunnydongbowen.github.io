---
title: "第一个go程序"
date: 2023-04-23
tags : [                                    
     "Go基础语法",
 ]
categories : [                              
     "Go语言",
 ]
---

> 💡 第一个golang程序，hello world！看这个函数和print语句写法，有点c语言的影子


## 1. hello world

```go
// 导入了这个包，意味着这个go程序最终要被编译成可执行程序
package main

// 导入语句
import "fmt"

// 函数外面只能放一些标识符(变量、常量、函数、类型)的声明，不能放语句会报错，具体的语句只能放到函数里
// 程序的入口函数
func main() {
	fmt.Println("hello, world")
}
```

## 2. 运行

-   在Goland中，鼠标放在代码末尾行，右击，选择go run，`不是运行整个项目，是运行单个go程序`
-   或者是命令行执行`go run`

## 3. 编译

在终端窗口进行编译，进入src目录，输入 `go build`

![](/go基础/20230423100933.png)

会在src目录下产生一个src.exe的文件，这个文件以**项目名命名**，在**哪个目录下编译，可执行文件就会在哪个文件夹下**。如在src/bin目录下编译,可执行文件就会在bin下面了，**编译完成后执行exe或者二进制，速度会很快！**

```go
go build  ../src/test.go   # 这个会产生一个test.exe的文件，就不是项目名了
go build  -o xxx.exe  ../src/test.go  # windows下
 window下编译指定名称的话要加exe后缀，不然识别不出来是什么格式的
go build -o 进制转换器.exe  ../src/test_045_进制转换器.go  
```

## 4. 交叉编译

在windows下编译Linux可执行程序

-   powershell

```powershell
$ENV:CGO_ENABLED=0
$ENV:GOOS="linux"
$ENV:GOARCH="amd64"
```

-   cmd

```powershell
SET CGO_ENABLED=0  // 禁用CGO
SET GOOS=linux     // 目标平台是linux
SET GOARCH=amd64   // 目标处理器架构是amd64
```

输入`go build` 进行编译，可以得到Linux下的可执行文件，要注意**goland和git打开的是终端`powershell`**，要按照第一个输入命令，cmd打开是cmd命令行，需要按照第二个进行设置，得到文件后，传到Linux服务器上测试即可，注意**chmod +x 赋予执行权限**

```bash
[root@os14 ~]# chmod +x src
[root@os14 ~]# ./src
hello, world
```

通过这个例子，可以看到在编译方面，go确实比py方便很多。py虽然也可以借助第三方包`pyinstaller`打包得到可执行文件，但编译后会含有很多dll文件，编译步骤也相对复杂，而且没法跨平台编译，windows下更可能存在兼容性问题。这就是编译型语言的优势。在**goland里面可以设置cmd格式的终端**

![](/go基础/20230423101419.png)

编译在代码开发阶段还是比较少，代码开发完成要部署到服务器阶段，会用的比较频繁。


## 5. 补充交叉编译错误

一直处在学习阶段的我，很久没编译了，跟着B站老男孩视频学习到goroutine时，老师讲的说`GOMAXPROCS`在windows下跑不出来效果，我照着敲了也跑不出来，说是在mac和Linux上可以(具体原因在goroutine章节有介绍)，于是，就用到了交叉编译，可是编译完成后，遇到了下面错误

```go
cmd/go: unsupported GOOS/GOARCH pair linux /amd64
```

原来是我执行`SET GOOS=linux` 命令时，后面多了一个空格！网上有和我犯一样错误的人(后来发现都是复制粘贴惹的祸，我也是直接翻上面的笔记，直接粘过来的)。