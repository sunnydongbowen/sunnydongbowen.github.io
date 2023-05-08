---
title: "panic"
date: 2023-04-26
tags : [                                    
     "Go基础语法",
 ]
categories : [                              
     "Go语言",
 ]
---
# 1. 认识panic

## 1.1 引入panic

Go语言中在（Go1.12）是没有异常机制，但是使用`panic/recover`模式来处理错误。 `panic`可以在任何地方引发，但`recover`只有在`defer`调用的函数中有效。 首先来看一个例子：
{{< admonition type=note title="panic示例"  >}} 
```go
func A() {
	fmt.Println("a...")
}

func B() {
	panic("panic in B")
}

func C() {
	fmt.Println("C...")

}
func main() {
	A()
	B()
	C()
}
```

`out`

```go
a...
panic: panic in B                             
                                              
goroutine 1 [running]:                        
main.B(...)                                   
        E:/go/gotest/src/practice.go:12       
main.main()                                   
        E:/go/gotest/src/practice.go:21 +0x66
```

程序运行期间`funcB`中引发了`panic`导致程序崩溃，异常退出了。C就没有执行，这个时候我们就可以通过`recover`将程序恢复回来，继续往后执行。

{{< /admonition >}}


{{< admonition type=example title="panic与recover示例"  >}} 
```go
func A() {
	fmt.Println("a...")
}
func B() {
	defer func() { // defer 必须在可能引发panic的语句之前使用
		err := recover() // 尽量少用，不推荐使用，recover必须在defer里使用！
		fmt.Println("返回的错误为：", err)
		println("释放数据库连接")
	}()
	panic("panic in B") //　程序异常退出
	fmt.Println("b")    // 这句不会被执行了
}

func C() {
	fmt.Println("C...")

}
func main() {
	A()
	B()
	C()
}
```

`out`

```go
a...
返回的错误为： panic in B 
释放数据库连接            
C...
```

**注意：**

1.  `recover()`必须搭配`defer`使用。
2.  `defer`一定要在可能引发`panic`的语句之前定义。

从上面例子可以看出，

-   如果不使用recover，那么直接下面的函数也不会调用了，主程序直接panic。使用了后才可以走下去。
-   即便使用recover，在当前的函数中，panic后面的语句也不会再执行了。

{{< /admonition >}}

## 1.2 panic和errors区别

{{< admonition type=tip title="panic与recover示例"  >}} 

```go
func main() {
	var a int
	a = 10
	if a < 20 {
		panic("不能玩手机哦")
	}
	fmt.Println("over")
}
```

运行结果如下：

```go
panic: 不能玩手机哦                             
                                          
goroutine 1 [running]:                          
main.main()                                     
        E:/go/gotest/src/test_painc2.go:9 +0x27
```

可以看到，程序在没有使用recover的情况下，出现了panic。

我们修改一下，加入recover后

```go
func pa() {
	var a int
	a = 10
	if a < 20 {
		defer func() {
			err := recover()
			fmt.Println(err)
		}()
		panic("不能玩手机哦")
	}
	fmt.Println("over-funcpa")  // 不会打印了
}

func main() {
	pa()
	fmt.Println("over~main")
}
```

`out`

```go
不能玩手机哦
over
```

{{< /admonition >}}


{{< admonition type=note title="errors示例"  >}} 

```go
func main() {
	var a int
	a = 10
	if a < 18 {
		fmt.Println(errors.New("不能玩手机哦"))
	}
	fmt.Println("over")
}
```

`out`

```go
不能玩手机哦
over
```

可以看到，使用errors的时候，仅仅是把error抛了出来，并不影响程序的运行。在实际使用中，我们尽可能避免使用panic，**遇事不决用error，error 肯定不会错**，让调用者去决定要不要 panic。panic 是不可挽回，error 还可以挽回。(这是从中间件的角度出发的)

Go中这个panic和errors其实都“类似”其他语言里的exception，但并不是一回事。

{{< /admonition >}}

# 2. 细致分析panic执行过程

panic 指的是 Go 程序在运行时出现的一个异常情况。如果异常出现了，但没有被捕获并恢复，Go 程序的执行就会被终止，**即便出现异常的位置不在主 Goroutine 中也会这样**。在 Go 中，panic 主要有两类来源，一类是来自 Go 运行时，另一类则是 Go 开发人员通过 panic 函数主动触发的。无论是哪种，一旦 panic 被触发，后续 Go 程序的执行过程都是一样的，这个过程被 Go 语言称为 panicking。

[Go 官方文档](https://go.dev/blog/defer-panic-and-recover)以手工调用 panic 函数触发 panic 为例，对 panicking 这个过程进行了诠释：当函数 F 调用 panic 函数时，函数 F 的执行将停止。不过，**函数 F 中已进行求值的 deferred 函数都会得到正常执行**，执行完这些 deferred 函数后，函数 F 才会把控制权返还给其调用者。对于函数 F 的调用者而言，函数 F 之后的行为就如同调用者调用的函数是 panic 一样，该panicking过程将继续在栈上进行下去，直到当前 Goroutine 中的所有函数都返回为止，然后 Go 程序将崩溃退出。我们用一个例子来更直观地解释一下 panicking 这个过程：

{{< admonition type=example title="分析panic过程1"  >}} 

```go
func foo() {
    println("call foo")
    bar()
    println("exit foo")
}

func bar() {
    println("call bar")
    panic("panic occurs in bar")
    zoo()
    println("exit bar")
}

func zoo() {
    println("call zoo")
    println("exit zoo")
}

func main() {
    println("call main")
    foo()
    println("exit main")
}
```

上面这个例子中，从 Go 应用入口开始，函数的调用次序依次为main -> foo -> bar -> zoo。在 bar 函数中，我们调用 panic 函数手动触发了 panic。我们执行这个程序的输出结果是这样的：

```go
call main
call foo
call bar
panic: panic occurs in bar
```

我们再根据前面对 panicking 过程的诠释，理解一下这个例子。
{{< /admonition >}}

{{< admonition type=example title="分析panic过程2"  >}} 

我们继续修改这个代码，只修改bar这个函数

```go
func bar() {
    defer func() {
        if e := recover(); e != nil {
            fmt.Println("recover the panic:", e)
        }
    }()

    println("call bar")
    panic("panic occurs in bar")
    zoo()
    println("exit bar")
}
```

在更新版的 bar 函数中，我们在一个 defer 匿名函数中调用 recover 函数对 panic 进行了捕捉。recover 是 Go 内置的专门用于恢复 panic 的函数，它必须被放在一个 defer 函数中才能生效。如果 recover 捕捉到 panic，它就会返回以 panic 的具体内容为错误上下文信息的错误值。如果没有 panic 发生，那么 recover 将返回 nil。而且，如果 panic 被 recover 捕捉到，panic 引发的 panicking 过程就会停止论 bar 函数正常执行结束，还是因 panic 异常终止，在那之前设置成功的 defer 函数都会得到执行。我们执行更新后的程序，得到如下结果：

```go
call main
call foo
call bar
recover the panic: panic occurs in bar
exit foo
exit main
```

在更新后的代码中，当 bar 函数调用 panic 函数触发异常后，bar 函数的执行就会被中断。但这一次，在代码执行流回到 bar 函数调用者之前，bar 函数中的、在 panic 之前就已经被设置成功的 derfer 函数就会被执行。这个匿名函数会调用 recover 把刚刚触发的 panic 恢复，这样，panic 还没等沿着函数栈向上走，就被消除了。所以，这个时候，从 foo 函数的视角来看，bar 函数与正常返回没有什么差别。foo 函数依旧继续向下执行，直至 main 函数成功返回。这样，这个程序的 panic“危机”就解除了。

{{< /admonition >}}

# 3. 什么时候要panic？

**能不用panic就不用，使用error即可**。

## 3.1 评估程序对 panic 的忍受度

不同应用对异常引起的程序崩溃退出的忍受度是不一样的。比如，一个单次运行于控制台窗口中的命令行交互类程序（CLI），和一个常驻内存的后端 HTTP 服务器程序，对异常崩溃的忍受度就是不同的。

**前者即便因异常崩溃，对用户来说也仅仅是再重新运行一次而已**。但后者一旦崩溃，就很可能导致整个网站停止服务。所以，针对各种应用对 panic 忍受度的差异，我们采取的应对 panic 的策略也应该有不同。像后端 HTTP 服务器程序这样的任务关键系统，我们就需要在特定位置捕捉并恢复 panic，以保证服务器整体的健壮度。在这方面，**Go 标准库中的 http server 就是一个典型的代表**。

Go 标准库提供的 http server 采用的是，每个客户端连接都使用一个单独的 Goroutine 进行处理的并发处理模型。也就是说，客户端一旦与 http server 连接成功，http server 就会为这个连接新创建一个 Goroutine，并在这 Goroutine 中执行对应连接（conn）的 serve 方法，来处理这条连接上的客户端请求。

前面提到了 panic 的“危害”时，我们说过，无论在哪个 Goroutine 中发生未被恢复的 panic，整个程序都将崩溃退出。所以，为了保证处理某一个客户端连接的 Goroutine 出现 panic 时，不影响到 http server 主 Goroutine 的运行，Go 标准库在 serve 方法中加入了对 panic 的捕捉与恢复，下面是 serve 方法的部分代码片段：

```go
// $GOROOT/src/net/http/server.go
// Serve a new connection.
func (c *conn) serve(ctx context.Context) {
    c.remoteAddr = c.rwc.RemoteAddr().String()
    ctx = context.WithValue(ctx, LocalAddrContextKey, c.rwc.LocalAddr())
    defer func() {
        if err := recover(); err != nil && err != ErrAbortHandler {
            const size = 64 << 10
            buf := make([]byte, size)
            buf = buf[:runtime.Stack(buf, false)]
            c.server.logf("http: panic serving %v: %v\n%s", c.remoteAddr, err, buf)
        }
        if !c.hijacked() {
            c.close()
            c.setState(c.rwc, StateClosed, runHooks)
        }
    }()
    ... ...
}
```

可以看到，serve 方法在一开始处就设置了 defer 函数，并在该函数中捕捉并恢复了可能出现的 panic。这样，即便处理某个客户端连接的 Goroutine 出现 panic，处理其他连接 Goroutine 以及 http server 自身都不会受到影响。这种**局部不要影响整体的异常处理策略**，在很多并发程序中都有应用。并且，捕捉和恢复 panic 的位置通常都在子 Goroutine 的起始处，这样设置可以捕捉到后面代码中可能出现的所有 panic，就像 serve 方法中那样。

## 3.2 提示潜在 bug

有了对 panic 忍受度的评估，panic 是不是也没有那么“恐怖”了呢？而且，我们甚至可以借助 panic 来帮助我们快速找到潜在 bug。

Go 语言标准库中并没有提供断言之类的辅助函数，但我们可以使用 panic，部分模拟断言对潜在 bug 的提示功能。比如，下面就是标准库encoding/json包使用 panic 指示潜在 bug 的一个例子：

```go
// $GOROOT/src/encoding/json/decode.go
... ...
//当一些本不该发生的事情导致我们结束处理时，phasePanicMsg将被用作panic消息
//它可以指示JSON解码器中的bug，或者
//在解码器执行时还有其他代码正在修改数据切片。

const phasePanicMsg = "JSON decoder out of sync - data changing underfoot?"

func (d *decodeState) init(data []byte) *decodeState {
    d.data = data
    d.off = 0
    d.savedError = nil
    if d.errorContext != nil {
        d.errorContext.Struct = nil
        // Reuse the allocated space for the FieldStack slice.
        d.errorContext.FieldStack = d.errorContext.FieldStack[:0]
    }
    return d
}

func (d *decodeState) valueQuoted() interface{} {
    switch d.opcode {
    default:
        panic(phasePanicMsg)

    case scanBeginArray, scanBeginObject:
        d.skip()
        d.scanNext()

    case scanBeginLiteral:
        v := d.literalInterface()
        switch v.(type) {
        case nil, string:
            return v
        }
    }
    return unquotedValue{}
}
```

我们看到，在valueQuoted这个方法中，如果程序执行流进入了 default 分支，那这个方法就会引发 panic，这个 panic 会提示开发人员：这里很可能是一个 bug。

同样，在 json 包的 encode.go 中也有使用 panic 做潜在 bug 提示的例子：

```go
// $GOROOT/src/encoding/json/encode.go

func (w *reflectWithString) resolve() error {
    ... ...
    switch w.k.Kind() {
    case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64:
        w.ks = strconv.FormatInt(w.k.Int(), 10)
        return nil
    case reflect.Uint, reflect.Uint8, reflect.Uint16, reflect.Uint32, reflect.Uint64, reflect.Uintptr:
        w.ks = strconv.FormatUint(w.k.Uint(), 10)
        return nil
    }
    panic("unexpected map key type")
}
```

这段代码中，resolve方法的最后一行代码就相当于一个“代码逻辑不会走到这里”的断言。一旦触发“断言”，这很可能就是一个潜在 bug。我们也看到，去掉这行代码并不会对resolve方法的逻辑造成任何影响，但真正出现问题时，开发人员就缺少了“断言”潜在 bug 提醒的辅助支持了。**在 Go 标准库中，大多数 panic 的使用都是充当类似断言的作用的**。

## 3.3 不要混淆异常与错误

这里不要把panic当成Java或python里的异常来理解，更不要把panic当成异常来使用。

上层代码，也就是 API 调用者根本不会去逐一了解 API 是否会引发panic，也没有义务去处理引发的 panic。一旦你在编写的 API 中，像checked exception那样使用 panic 作为正常错误处理的手段，把引发的panic当作错误，那么你就会给你的 API 使用者带去大麻烦！因此，在 Go 中，作为 API 函数的作者，你一定不要将 panic 当作错误返回给 API 调用者。

# 4. 总结

-   在实际编码中，尽量不要使用panic，而是使用error
-   panic 与defer和recover配合使用，要注意顺序