---
title: "Context基础篇"
date: 2023-04-02T10:43:34+08:00
tags : [                                    
     "Go进阶语法",
 ]
categories : [                              
     "Go语言",
 ]
 
author: "博文" 
---

在 Go http包的Server中，每一个`请求`在都有一个对应的 `goroutine` 去处理。请求处理函数通常会启动额外的 `goroutine` 用来访问后端服务，比如数据库和RPC服务。用来处理一个请求的 goroutine 通常需要访问一些与请求特定的数据，比如终端用户的身份认证信息、验证相关的token、请求的截止时间。 当一个请求被取消或超时时，所有用来处理该请求的 goroutine 都应该迅速退出，然后系统才能释放这些 goroutine 占用的资源。

## bool 型退出

```go
var wg sync.WaitGroup
var notify bool

func f() {
	defer wg.Done() //不加defer的话直接死锁了
	for {
		fmt.Println("bowen")
		time.Sleep(time.Millisecond * 500)
		if notify {
			break
		}
	}
}

func TestBoolExit(t *testing.T) {
	wg.Add(1)
	go f()
	time.Sleep(5 * time.Second)
	// 通知退出
	notify = true
	wg.Wait()
}
```

上面示例中，函数f()是一个死循环，每隔500ms打印一次，在主程序中，开启了一个goroutine去执行这个函数，并且延时5s，我们想让它在5s钟后退出，应该怎么做呢？上面程序设计了一个全局变量，5s后将全局变量设置为true，这个时候，子goroutine f() 检测到为true，就会退出死循环，从而函数执行完成，子goroutine成功结束.

## chan型退出

```Go
var exitChan = make(chan bool, 1)

func fchan() {
	defer wg.Done()
FORLOOP:
	for {
		fmt.Println("001")
		time.Sleep(time.Millisecond * 500)
		select {
		case <-exitChan:
			break FORLOOP
		default:
		}
	}
}
func TestChanExit(t *testing.T) {
	wg.Add(1)
	go fchan()
	time.Sleep(5 * time.Second)
	exitChan <- true
	wg.Wait()
```

这个程序同样的子goroutine是无限循环，除了上面的最常用的bool型退出外，这个示例中，定义了一个bool型的chan，并且在for循环中，使用select检测，5s钟后main程序中将true发送给chan，子goroutine检测到后，退出for循环，结束这个子goroutine。

{{< admonition type=info title="注意点"  >}}

- 在case里是没有break语句的！这里的break，是针对for循环的。之前在case里写了break，怎么都没有退出！

- default不能缺，否则会阻塞

  {{< /admonition >}}

## 引入context

上面两种解决方式是按照我们自己的想法来实现的。官方中其实对这个问题有更专业的解决方式。

Go1.7加入了一个新的`标准库context`，它定义了`Context类型`，专门用来简化 对于处理单个请求的多个 goroutine 之间与请求域的数据、取消信号、截止时间等相关操作，这些操作可能涉及多个 API 调用。

对服务器传入的请求应该创建上下文，而对服务器的传出调用应该接受上下文。它们之间的函数调用链必须传递上下文，或者可以使用`WithCancel`、`WithDeadline`、`WithTimeout`或`WithValue`创建的派生上下文。当一个上下文被取消时，它派生的所有上下文也被取消。

## context.Context是什么？

答案: `它是一个接口！`，既然是接口，那它又实现了哪些方法？该接口定义了需要实现的方法

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

## Context interface的四个方法

- `Deadline`方法需要返回当前`Context`被取消的时间，也就是完成工作的截止时间

- `Done`方法需要返回一个`Channel`，这个Channel会在当前工作完成或者上下文被取消之后关闭，多次调用`Done`方法会返回同一个Channel；示例

  ````go
  select {
  			case <-ctx.Done():
  				return
  			case dst <- n:
  				n++
  			}
  ````

- Err方法会返回当前Context结束的原因，它只会在Done返回的Channel被关闭时才会返回非空的值；
  - 如果当前Context被取消就会返回Canceled错误；
  - 如果当前Context超时就会返回DeadlineExceeded错误；

- Value方法会从Context中返回键对应的值，对于同一个上下文来说，多次调用Value 并传入相同的Key会返回相同的结果，该方法仅用于传递跨API和进程间跟请求域的数据；

## context包内置的两个函数

Background()主要用于main函数、初始化以及测试代码中，作为Context这个树结构的最顶层的Context，也就是根Context。

```go
ctx, cancel := context.WithCancel(context.Background())
```

`TODO()`，它目前还不知道具体的使用场景，如果不知道该使用什么Context的时候，可以使用这个。

`background`和`todo`本质上都是`emptyCtx`结构体类型，是一个不可取消，没有设置截止时间，没有携带任何值的Context。

## context包重要方法

### WithCancel()

WithCancel的函数签名如下：

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
```

WithCancel返回带有新Done通道的父节点的副本。当调用返回的cancel函数或当关闭父上下文的Done通道时，将关闭返回上下文的Done通道，无论先发生什么情况。取消此上下文将释放与其关联的资源，因此代码应该在此上下文中运行的操作完成后立即调用cancel。

```go
func con2(ctx context.Context) {
	defer wg.Done()
FORLOOP:
	for {
		fmt.Println("002")
		time.Sleep(time.Millisecond * 500)
		select {
		case <-ctx.Done():
			break FORLOOP
		default:
		}
	}
}

func fcon(ctx context.Context) {
	defer wg.Done()
	wg.Add(1)
	go con2(ctx)
FORLOOP:
	for {
		fmt.Println("001")
		time.Sleep(time.Millisecond * 500)
		select {
		case <-ctx.Done():
			break FORLOOP
		default:
		}
	}
}

func TestContext1(t *testing.T) {
	ctx, cancel := context.WithCancel(context.Background())
	wg.Add(1)
	go fcon(ctx)
	time.Sleep(time.Second * 5)
	cancel()
	wg.Wait()
}

```

分析一下上面的程序，很简单，首先在main 程序中创建context, 开启子goroutine，子goroutine是一个无限循环，和上面的引入示例一样，5s中通知退出。上面有两个函数，其中，在子的goroutine中又有一个子的goroutine，我们需要通知他们到了时间后退出。在当我们调用cancel()函数时，就开始通知子goroutine结束，而子goroutine收到通知后，会通知它的子goroutine结束。这一部分可以看源码，是采用无限循环实现的。

{{< admonition type=failure title="报错"  >}}

上面例子中，我遇到了一个报错，`panic: sync: negative WaitGroup counter`  比对我之前的代码，一样的。在最外面的函数中少了`wg.add(1)`, 加上后解决报错。这里要学会wg.add() ,wg.wait(),wg.done()这三个的位置！{{< /admonition >}}

{{< admonition type=info title="更形象的例子"  >}}

简单总结来说，`WithCancel()的作用就是到了时间后，通知子goroutine取消`，子goroutine通知它的子goroutine取消，一步步下去。下面我们看一个更形象的例子

{{< /admonition >}}

```go
func sun(ctx context.Context) error {
	defer wg.Done()
LOOP:
	for {
		fmt.Println("孙子goroutine: 我饿了")
		time.Sleep(500 * time.Millisecond)
		select {
		case <-ctx.Done():
			fmt.Println("孙子goroutine: 好的，我不叫了")
			break LOOP
		default:
		}
	}
	return ctx.Err()
}

func fa(ctx context.Context) error {
	defer wg.Done()
	wg.Add(1)
	go sun(ctx)
LOOP:
	for {
		fmt.Println("儿子goroutine: 我饿了")
		time.Sleep(time.Millisecond * 500)
		select {
		case <-ctx.Done():
			fmt.Println("儿子goroutine: 好的，我不叫了")
			time.Sleep(10 * time.Nanosecond)
			break LOOP
		default:
		}
	}
	return ctx.Err()
}

func TestGoroutie(t *testing.T) {
	ctx, cancel := context.WithCancel(context.Background())
	wg.Add(1)
	go fa(ctx)
	time.Sleep(5 * time.Second)
	fmt.Println("main:好了，别叫了，开饭了！")
	cancel()
	wg.Wait()
}
```

这个程序也是我去年学习goroutine为了理解，写的一个很有意思的例子。运行结果如下：

![](/并发编程/20230403162305.png)

{{< admonition type=note title="官网demo"  >}}再看一个官网的例子，这一部分很重要，举了三个例子便于理解,而这个例子的套路就和上面不一样了。{{< /admonition >}}

```go
func TestOfficial(t *testing.T) {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	for n := range gen(ctx) {
		fmt.Println(n)
		if n == 5 {
			break
		}
	}
}

func gen(ctx context.Context) <-chan int {
	dst := make(chan int)
	n := 1
	go func() {
		for {
			select {
			case <-ctx.Done():
				return
			case dst <- n:
				n++
			}
		}
	}()
	return dst
}
```

这段程序gen()一直产生数字，发送到chan中，将chan返回。main中从中取数字，直到取到了5，就退出for循环不再取了，这时就会走到cancel()函数，子goroutine收到通知后，就退出。

### WithDeadline()

WithDeadline的函数签名如下：

```go
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
```

返回父上下文的副本，并将deadline调整为不迟于d。如果父上下文的deadline已经早于d，则WithDeadline(parent, d)在语义上等同于父上下文。当截止日过期时，当调用返回的cancel函数时，或者当父上下文的Done通道关闭时，返回上下文的Done通道将被关闭，以最先发生的情况为准。取消此上下文将释放与其关联的资源，因此代码应该在此上下文中运行的操作完成后立即调用cancel

```go
func TestDeadline(t *testing.T) {
	d := time.Now().Add(1 * time.Second)
	ctx, cancel := context.WithDeadline(context.Background(), d)
	defer cancel()
	select {
	case <-time.After(3 * time.Second):
		fmt.Println("0001")
		//fmt.Println(ctx.Err())
	case <-ctx.Done():
		fmt.Println(ctx.Err())
	}
}
```

上面的代码中，定义了一个1s之后过期的deadline，我们调用`context.WithDeadline(context.Background(), d)`得到一个上下文（ctx）和一个取消函数（cancel），使用一个select让主程序陷入等待：等待1秒后打印或者等待ctx过期后退出。

在上面的示例代码中，因为ctx 1s后就会过期，所以`ctx.Done()`会先接收到context到期通知，并且会打印ctx.Err()的内容。

{{< admonition type=note title="简单理解"  >}}deadline相当于给了一个时间限制，如果时间到了，即便没接到通知，也会退出{{< /admonition >}}

![](/并发编程/20230403172207.png)

### WithTimeout()

WithTimeout的函数签名如下：

```go
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
```

`WithTimeout`返回`WithDeadline(parent, time.Now().Add(timeout))`。

取消此上下文将释放与其相关的资源，因此代码应该在此上下文中运行的操作完成后立即调用cancel，通常用于数据库或者网络连接的超时控制。具体示例如下：

```go
func TestWithout(t *testing.T) {
	ctx, cancel := context.WithTimeout(context.Background(), time.Millisecond*50)
	wg.Add(1)
	go worker(ctx)
	time.Sleep(time.Second * 5)
	cancel()
	wg.Wait()
	fmt.Println("over")
}

func worker(ctx context.Context) {
LOOP:
	for {
		fmt.Println("db connecting...")
		time.Sleep(time.Millisecond * 10)
		select {
		case <-ctx.Done():
			break LOOP
		default:
		}
	}
	fmt.Println("worker done")
	wg.Done()
}
```

来简单分析一下这个程序:

- 第一种情况，业务操作在设置的超时时间前完成了，这时`context.WithTimeout函数自己到了超时时间发出的退出通知` ，业务操作就结束了。这个不是手动调用cancel产生的。
- 第二种情况，业务操作超时了，就是在设置的时间外才完成，那么这个也是`context.WithTimeout函数自己到了超时时间发出的退出通知`  不是我主动调用的cancel()。

{{< admonition type=note title="注意点"  >}}

- cancel有个坑，如果超时完成前，就执行完成业务操作的话，还得手动调用cancel，否则就要等到超时时间才会释放资源。在实际编码中，如果明确知道成功了，可以提前手动调用，而不必等到超时时间到了。
- 一般都是在`context.WithTimeout`后面，下一句直接写defer cancel()保证任何情况都可以取消。{{< /admonition >}}

### WithValue()

WithValue函数能够将请求作用域的数据与 Context 对象建立关系。声明如下：

```go
func WithValue(parent Context, key, val interface{}) Context
```

`WithValue`返回父节点的副本，其中与key关联的值为val。

仅对API和进程间传递请求域的数据使用上下文值，而不是使用它来传递可选参数给函数。

所提供的键必须是可比较的，并且不应该是`string`类型或任何其他内置类型，以避免使用上下文在包之间发生冲突。`WithValue`的用户应该为键定义自己的类型。为了避免在分配给interface{}时进行分配，上下文键通常具有具体类型`struct{}`。或者，导出的上下文关键变量的静态类型应该是指针或接口。