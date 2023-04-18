---
title: "channel进阶篇"
date: 2023-04-15
tags : [                                    
     "Go进阶语法",
 ]
categories : [                              
     "Go语言",
 ]
---

{{< admonition type=info title="并发编程"  >}}

 这是我学习极客时间tony白专栏时摘录的笔记，如有侵权，请联系我删除。在之前已经写过一篇关于chan的笔记了。chan 很重要！要仔细研究！

{{< /admonition >}}

# channel简介

 Go 语言实现了基于 CSP（Communicating Sequential Processes）理论的并发方案。

Go 语言的 CSP 模型的实现包含两个主要组成部分：一个是 `Goroutine`，它是 Go 应用并发设计的基本构建与执行单元；另一个就是 `channel`，它在并发模型中扮演着重要的角色。channel 既可以用来实现 Goroutine 间的通信，还可以实现 Goroutine 间的同步。它就好比 Go 并发设计这门“武功”的秘籍口诀，可以说，学会在 Go 并发设计时灵活运用 channel，才能说真正掌握了 Go 并发设计的真谛。
Go 对并发的原生支持可不是仅仅停留在口号上的，Go 在语法层面将并发原语 channel 作为一等公民对待。
这意味着我们可以像使用普通变量那样使用 channel，比如，定义 channel 类型变量、给 channel 变量赋值、将 **channel 作为参数传递给函数 / 方法、将 channel 作为返回值从函数 / 方法中返回，甚至将 channel 发送到其他 channel 中**。这就大大简化了 channel 原语的使用，提升了我们开发者在做并发设计和实现时的体验。

# channel操作
## 创建channel

和切片、结构体、map 等一样，channel 也是一种复合数据类型。也就是说，我们在声明一个 channel 类型变量时，必须给出其具体的元素类型，比如下面的代码这样：

```go
var ch chan int
```

这句代码里，我们声明了一个元素为 int 类型的 channel 类型变量 ch。

如果 channel 类型变量在声明时没有被赋予初值，那么它的默认值为 `nil`。并且，和其他复合数据类型支持使用复合类型字面值作为变量初始值不同，为 channel 类型变量赋初值的唯一方法就是使用 `make` 这个 Go 预定义的函数，比如下面代码：

```go
ch1 := make(chan int)   
ch2 := make(chan int, 5)
```

这里，我们声明了两个元素类型为 int 的 channel 类型变量 ch1 和 ch2，并给这两个变量赋了初值。但我们看到，两个变量的赋初值操作使用的 make 调用的形式有所不同。

第一行我们通过make(chan T)创建的、元素类型为 T 的 channel 类型，是无缓冲 channel，而第二行中通过带有 capacity 参数的make(chan T, capacity)创建的元素类型为 T、缓冲区长度为 capacity 的 channel 类型，是`带缓冲 channel`。

这两种类型的变量关于发送（send）与接收（receive）的特性是不同的，我们接下来就基于这两种类型的 channel，看看 channel 类型变量如何进行发送和接收数据元素。

## 发送与接收

Go 提供了<-操作符用于对 channel 类型变量进行发送与接收操作：
```go
ch1 <- 13 // 将整型字面值13发送到无缓冲channel类型变量ch1中 
n := <- ch1 // 从无缓冲channel类型变量ch1中接收一个整型值存储到整型变量n中
ch2 <- 17 // 将整型字面值17发送到带缓冲channel类型变量ch2中
m := <- ch2 // 从带缓冲channel类型变量ch2中接收一个整型值存储到整型变量m中
```

{{< admonition type=tip  title="提醒"  >}}
这里我要提醒你一句，在理解 channel 的发送与接收操作时，你一定要始终牢记：**channel 是用于 Goroutine 间通信的，所以绝大多数对 channel 的读写都被分别放在了不同的 Goroutine 中**
{{< /admonition >}}

## 无缓冲channel
现在，我们先来看看无缓冲 channel 类型变量（如 ch1）的发送与接收。

由于无缓冲 channel 的运行时层实现不带有缓冲区，所以 Goroutine 对无缓冲 channel 的接收和发送操作是同步的。也就是说，对同一个无缓冲 channel，只有对它进行接收操作的 Goroutine 和对它进行发送操作的 Goroutine 都存在的情况下，通信才能得以进行，否则单方面的操作会让对应的 Goroutine 陷入挂起状态，比如下面示例代码：
{{< admonition type=failure title="错误示例"  >}} 

```go
ch1 := make(chan int) 
ch1 <- 13 // fatal error: all goroutines are asleep - deadlock! 
n := <-ch1 
println(n)
```

在这个示例中，我们创建了一个无缓冲的 channel 类型变量 ch1，对 ch1 的读写都放在了一个 Goroutine 中。

运行这个示例，我们就会得到 fatal error，提示我们所有 Goroutine 都处于休眠状态，程序处于死锁状态。要想解除这种错误状态，**我们只需要将接收操作，或者发送操作放到另外一个 Goroutine 中就可以了，比如下面代码**：

{{< /admonition >}}

{{< admonition type=tip  title="正确写法"  >}} 
```go
func main() {
	ch1 := make(chan int) 
	go func() { 
		ch1 <- 13 // 将发送操作放入一个新goroutine中执行 
		}() 
	n := <-ch1 
	println(n) 
}
```
由此，我们可以得出结论：**对无缓冲 channel 类型的发送与接收操作，一定要放在两个不同的 Goroutine 中进行，否则会导致 deadlock**。
{{< /admonition >}}

## 有缓冲channel
接下来，我们再来看看带缓冲 channel 的发送与接收操作。

和无缓冲 channel 相反，带缓冲 channel 的运行时层实现带有缓冲区，因此，对带缓冲 channel 的发送操作在缓冲区未满、接收操作在缓冲区非空的情况下是**异步**的（发送或接收不需要阻塞等待）。

也就是说，对一个带缓冲 channel 来说，在缓冲区未满的情况下，对它进行发送操作的 Goroutine 并不会阻塞挂起；在缓冲区有数据的情况下，对它进行接收操作的 Goroutine 也不会阻塞挂起。

但当缓冲区满了的情况下，对它进行发送操作的 Goroutine 就会阻塞挂起；当缓冲区为空的情况下，对它进行接收操作的 Goroutine 也会阻塞挂起。

如果光看文字还不是很好理解，你可以再看看下面几个关于带缓冲 channel 的操作的例子：

```go
ch2 := make(chan int, 1) 
n := <-ch2 // 由于此时ch2的缓冲区中无数据，因此对其进行接收操作将导致goroutine挂起 
ch3 := make(chan int, 1) 
ch3 <- 17 // 向ch3发送一个整型数17
ch3 <- 27 // 由于此时ch3中缓冲区已满，再向ch3发送数据也将导致goroutine挂起
```

也正是因为带缓冲 channel 与无缓冲 channel 在发送与接收行为上的差异，在具体使用上，它们有各自的“用武之地”，这个我们等会再细说，现在我们先继续把 channel 的基本语法讲完。
## 限制只发送或只接收

使用操作符<-，我们还可以声明只发送 channel 类型（send-only）和只接收 channel 类型（recv-only），我们接着看下面这个例子
```go
ch1 := make(chan<- int, 1) // 只发送channel类型
ch2 := make(<-chan int, 1) // 只接收channel类型

<-ch1       // invalid operation: <-ch1 (receive from send-only type chan<- int)
ch2 <- 13   // invalid operation: ch2 <- 13 (send to receive-only type <-chan int)
```

> 不要弄混符号，看chan是在箭头左边还是右边，左边是**send-only**，右边是**receive-only**

你可以从这个例子中看到，试图从一个只发送 channel 类型变量中接收数据，或者向一个只接收 channel 类型发送数据，都会导致编译错误。通常只发送 channel 类型和只接收 channel 类型，会被用作函数的参数类型或返回值，用于限制对 channel 内的操作，或者是明确可对 channel 进行的操作的类型，比如下面这个例子：
{{< admonition type=note title="生产者消费者模型"  >}}
```go
func produce(ch chan<- int) {  
   for i := 0; i < 10; i++ {  
      ch <- i + 1  
      time.Sleep(time.Second)  
   }  
   close(ch)  
   defer wg.Done()  
}  
  
func consume(ch <-chan int) {  
   for v := range ch {  
      println(v)  
   }  
   defer wg.Done()  
}  

func TestProducer(t *testing.T) {  
   ch := make(chan int, 5)  
   wg.Add(2)  
  
   go produce(ch)  
   go consume(ch)  
   wg.Wait()  
  
}
```
在这个例子中，我们启动了两个 Goroutine，分别代表生产者（produce）与消费者（consume）。生产者只能向 channel 中发送数据，我们使用chan<- int作为 produce 函数的参数类型；消费者只能从 channel 中接收数据，我们使用<-chan int作为 consume 函数的参数类型。

在消费者函数 consume 中，我们使用了 for range 循环语句来从 channel 中接收数据，for range 会阻塞在对 channel 的接收操作上，直到 channel 中有数据可接收或 channel 被关闭循环，才会继续向下执行。channel 被关闭后，for range 循环也就结束了。

> 这是一个很典型的例子，在并发编程基础篇里也有写过类似的，在学习数据库操作时，也有利用这个写过批量插入数据库的demo，很有意思，虽然gorm里提供了批量插入数据的API。

{{< /admonition >}}

## 关闭channel
在上面的例子中，produce 函数在发送完数据后，调用 Go 内置的 close 函数关闭了 channel。channel 关闭后，所有等待从这个 channel 接收数据的操作都将返回。

这里我们继续看一下采用不同接收语法形式的语句，在 channel 被关闭后的返回值的情况：
```go
n := <- ch      // 当ch被关闭后，n将被赋值为ch元素类型的零值
m, ok := <-ch   // 当ch被关闭后，m将被赋值为ch元素类型的零值, ok值为false
for v := range ch { // 当ch被关闭后，for range循环结束
    ... ...
}
```
我们看到，通过“comma, ok”惯用法或 for range 语句，我们可以准确地判定 channel 是否被关闭。而单纯采用n := <-ch形式的语句，我们就无法判定从 ch 返回的元素类型零值，究竟是不是因为 channel 被关闭后才返回的。
{{< admonition type=note title="在发送端关闭channel"  >}}
另外，从前面 produce 的示例程序中，我们也可以看到，`channel 是在 produce 函数中被关闭的`，这也是 channel 的一个`使用惯例`，`那就是发送端负责关闭 channel`。

这里为什么要在**发送端关闭 channel **呢？

这是因为发送端没有像接受端那样的、可以安全判断 channel 是否被关闭了的方法。同时，一旦向一个已经关闭的 channel 执行发送操作，这个操作就会引发 panic，比如下面这个示例：
```go
ch := make(chan int, 5)
close(ch)
ch <- 13 // panic: send on closed channel
```

{{< /admonition >}}

# select
当涉及同时对多个 channel 进行操作时，我们会结合 Go 为 CSP 并发模型提供的另外一个原语 select，一起使用。通过 select，我们可以同时在多个 channel 上进行发送 / 接收操作：
```go
select {
case x := <-ch1:     // 从channel ch1接收数据
  ... ...

case y, ok := <-ch2: // 从channel ch2接收数据，并根据ok值判断ch2是否已经关闭
  ... ...

case ch3 <- z:       // 将z值发送到channel ch3中:
  ... ...

default:             // 当上面case中的channel通信均无法实施时，执行该默认分支
}
```

当 select 语句中没有 default 分支，而且所有 case 中的 channel 操作都阻塞了的时候，整个 select 语句都将被阻塞，直到某一个 case 上的 channel 变成可发送，或者某个 case 上的 channel 变成可接收，select 语句才可以继续进行下去。关于 select 语句的妙用，我们在后面还会细讲，这里我们先简单了解它的基本语法

看到这里你应该能感受到，channel 和 select 两种原语的操作都十分简单，它们都遵循了 Go 语言“追求简单”的设计哲学，但它们却为 Go 并发程序带来了**强大的表达能力**。学习了这些基础用法后，接下来我们再深一层，看看 Go 并发原语 channel 的一些惯用法。同样地，这里我们也分成**无缓冲 channel 、带缓冲 channel** 两种情况来分析。


# 无缓冲 channel 的惯用法

## 用作信号传递

无缓冲 channel 兼具通信和同步特性，在并发程序中应用颇为广泛。现在我们来看看几个无缓冲 channel 的典型应用：
{{< admonition type=note title="用作信号传递"  >}}
```go
type signal struct {  
}  
  
func worker() {  
   println("worker is working")  
   time.Sleep(1 * time.Second)  
}  
  
func spawn(f func()) <-chan signal {  
   c := make(chan signal)  
   go func() {  
      println("worker start to work...")  
      f()  
      c <- signal{}  
   }()  
   return c  
}  
  
func TestSignal(t *testing.T) {  
   println("main start a worker.....")  
   c := spawn(worker)  
   <-c  
   fmt.Println("worker work done!")  
  
}
```
在这个例子中，spawn 函数返回的 channel，被用于承载新 Goroutine 退出的“通知信号”，这个信号专门用作通知 main goroutine。main goroutine 在调用 spawn 函数后一直阻塞在对这个“通知信号”的接收动作上。

我们来运行一下这个例子：
```
main start a worker.....
worker start to work...
worker is working
worker work done!
```
{{< /admonition >}}

{{< admonition type=note title="1对n信号通知"  >}}
有些时候，无缓冲 channel 还被用来实现 1 对 n 的信号通知机制。这样的信号通知机制，常被用于协调多个 Goroutine 一起工作，比如下面的例子：
```go
func worker1(i int) {  
   fmt.Printf("worker %d: is working...\n", i)  
   time.Sleep(1 * time.Second)  
   fmt.Printf("worker %d: works done\n", i)  
}  
  
func spawnGroup(f func(i int), num int, grouSignal <-chan signal) <-chan signal {  
  
   c := make(chan signal)  
   // Num个goroutine  
   for i := 0; i < num; i++ {  
      wg.Add(1)  
      go func(i int) {  
         <-grouSignal //阻塞点  
         fmt.Printf("worker %d: start to work...\n", i)  
         f(i)  
         wg.Done()  
      }(i + 1)  
   }  
  
   go func() {  
      wg.Wait()  
      c <- signal{}  
   }()  
   return c  
}  
  
func TestSignal2(t *testing.T) {  
   fmt.Println("start a group of workers...")  
   groupSignal := make(chan signal)  
   c := spawnGroup(worker1, 5, groupSignal)  
   time.Sleep(5 * time.Second)  
   fmt.Println("the group of workers start to work...")  
   close(groupSignal) // 关闭了  
   <-c  
   fmt.Println("the group of workers work done!")  
}
```
这个例子中，main goroutine 创建了一组 5 个 worker goroutine，这些 Goroutine 启动后会阻塞在名为 groupSignal 的无缓冲 channel 上。main goroutine 通过close(groupSignal)向所有 worker goroutine 广播“开始工作”的信号，收到 groupSignal 后，所有 worker goroutine 会“同时”开始工作，就像起跑线上的运动员听到了裁判员发出的起跑信号枪声。
这个例子的运行结果如下：
```
start a group of workers...
the group of workers start to work...
worker 1: start to work...
worker 3: start to work...
worker 1: is working...
worker 2: start to work...
worker 2: is working...
worker 5: start to work...
worker 5: is working...
worker 3: is working...
worker 4: start to work...
worker 4: is working...
worker 4: works done
worker 5: works done
worker 3: works done
worker 2: works done
worker 1: works done
the group of workers work done!
--- PASS: TestSignal2 (6.02s)
```
>我们可以看到，关闭一个无缓冲 channel 会让所有阻塞在这个 channel 上的接收操作返回，从而实现了一种 1 对 n 的“广播”机制。

{{< /admonition >}}

## 用于替代锁机制

{{< admonition type=note title="基于“共享内存”+“互斥锁”的 Goroutine 安全的计数器的实现"  >}}
```go
type counter struct {  
   sync.Mutex  
   i int  
}  
  
var cter counter  
  
// 加锁解锁，i+1  
func Increase() int {  
   cter.Lock()  
   defer cter.Unlock()  
   cter.i++  
   return cter.i  
}  
  
func TestNoBuffer(t *testing.T) {  
   for i := 0; i < 10; i++ {  
      wg.Add(1)  
      go func(i int) {  
         v := Increase()  
         fmt.Printf("goroutine-%d: current counter value is %d\n", i, v)  
         wg.Done()  
      }(i)  
   }  
   wg.Wait()  
}

```

运行结果
```
=== RUN   TestNoBuffer
goroutine-9: current counter value is 1
goroutine-0: current counter value is 2
goroutine-1: current counter value is 3
goroutine-2: current counter value is 4
goroutine-3: current counter value is 5
goroutine-4: current counter value is 6
goroutine-5: current counter value is 7
goroutine-6: current counter value is 8
goroutine-7: current counter value is 9
goroutine-8: current counter value is 10
--- PASS: TestNoBuffer (0.00s)
PASS
```

{{< /admonition >}}

{{< admonition type=tip title="扩展"  >}}
把程序修改一下
```go
 go func() {
            v := Increase()
            fmt.Printf("goroutine-%d: current counter value is %d\n", i, v)
            wg.Done()
        }()
```
输出结果
```
goroutine-2: current counter value is 1
goroutine-9: current counter value is 3
goroutine-9: current counter value is 4
goroutine-9: current counter value is 5
goroutine-9: current counter value is 6
goroutine-9: current counter value is 7
goroutine-9: current counter value is 8
goroutine-10: current counter value is 9
goroutine-5: current counter value is 2
goroutine-10: current counter value is 10
```
> 得到上面结果，是不是和在[并发编程基础篇](https://sunnydongbowen.github.io/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E5%9F%BA%E7%A1%80%E7%AF%87/#22-goroutine%E4%B8%8E%E9%97%AD%E5%8C%85)很类似，思考一下，为什么？

{{< /admonition >}}

{{< admonition type=note title="无缓冲 channel 替代锁"  >}}
在上面示例中，我们使用了一个带有互斥锁保护的全局变量作为计数器，所有要操作计数器的 Goroutine 共享这个全局变量，并在互斥锁的同步下对计数器进行自增操作。
接下来我们再看更符合 Go 设计惯例的实现，也就是使用`无缓冲 channel 替代锁`后的实现：
```go
type counterchan struct {  
   c chan int  
   i int  
}  
  
func NewCounter() *counterchan {  
   cter := &counterchan{  
   // 这是无缓冲的chan
      c: make(chan int),  
   }  
   go func() {  
   // 无限循环，会阻塞在这里
      for {  
         cter.i++  // i+1
         cter.c <- cter.i  //发送给chan，这是必须有人接收，否则会阻塞在这
      }    
   }()  
   // 返回chan
   return cter  
}  
  
// 返回channel里的数值
func (cter *counterchan) Increase() int {  
   return <-cter.c  
}  
  
func TestChanMutex(t *testing.T) { 
   // 
   cter := NewCounter()  
  
   for i := 0; i < 10; i++ {  
      wg.Add(1)  
      go func(i int) {  
         v := cter.Increase() // 这里开始接收。
         fmt.Printf("goroutine-%d: current counter value is %d\n", i, v)  
         wg.Done()  
      }(i)  
   }  
   wg.Wait()  
}
```

- 创建NewCounter() 函数，里面的无限循环会阻塞在`cter.c <- cter.i `   发。
- 直到下面的goroutine调用`Increase() `函数才会执行，收
- 通过一发一收，进行控制，利用的就是无缓冲channel的同步阻塞特性，这样就不会产生冲突
这个实现中，我们将计数器操作全部交给一个独立的 Goroutine 去处理，并通过无缓冲 channel 的同步阻塞特性，实现了计数器的控制。这样其他 Goroutine 通过 Increase 函数试图增加计数器值的动作，**实质上就转化为了一次无缓冲 channel 的接收动作**。
这种并发设计逻辑更符合 Go 语言所倡导的“不要通过共享内存来通信，而是通过通信来共享内存”的原则。
{{< /admonition >}}


# 带缓冲 channel 的惯用法
带缓冲的 channel 与无缓冲的 channel 的最大不同之处，就在于它的异步性。也就是说，对一个带缓冲 channel，在缓冲区未满的情况下，对它进行发送操作的 Goroutine 不会阻塞挂起；在缓冲区有数据的情况下，对它进行接收操作的 Goroutine 也不会阻塞挂起。

这种特性让带缓冲的 channel 有着与无缓冲 channel 不同的应用场合。接下来我们一个个来分析。
## 消息队列




# 5. 并发安全和锁

有时候我们的代码中可能会存在多个 goroutine 同时操作一个资源（临界区）的情况，这种情况下就会发生竞态问题（数据竞态）。这就好比现实生活中十字路口被各个方向的汽车竞争，还有火车上的卫生间被车厢里的人竞争。我们用下面的代码演示一个数据竞争的示例。

```go
var (
	x int64
)

func add() {
	for i := 0; i < 5000; i++ {
		x = x + 1
	}
	wg.Done()
}

func TestMutex(t *testing.T) {
	wg.Add(2)
	go add()
	go add()
	wg.Wait()
	fmt.Println(x)
}
```

我们将上面的代码编译后执行，不出意外每次执行都会输出诸如9537、5865、6527等不同的结果。这是为什么呢？(当然如果电脑性能足够好，也是会输出10000的，我试了几次)。

在上面的示例代码片中，我们开启了两个 goroutine 分别执行 add 函数，这两个 goroutine 在访问和修改全局的`x`变量时就会存在数据竞争，某个 goroutine 中对全局变量`x`的修改可能会覆盖掉另一个 goroutine 中的操作，所以导致最后的结果与预期不符。

{{< admonition type=failure title="报错"  >}}

上面的demo是我去年刚学习的时候敲的，我看了notion记录的是wg.add(1),这是一个很明显的错误。在这次整理博客的过程中，也发现自己之前敲的代码一些错误，边整理博客边复习。看看在这条路上，我能走多远。

{{< /admonition >}}

## 5.1 互斥锁

互斥锁是一种常用的控制共享资源访问的方法，它能够保证同一时间只有一个 goroutine 可以访问共享资源。Go 语言中使用`sync`包中提供的`Mutex`类型来实现互斥锁。`sync.Mutex`提供了两个方法供我们使用。

| 方法名                   | 功能       |
| ------------------------ | ---------- |
| func (m *Mutex) Lock()   | 获取互斥锁 |
| func (m *Mutex) Unlock() | 释放互斥锁 |

{{< admonition type=tip  title="互斥锁"  >}}

```go
m  sync.Mutex     // 互斥锁
m.Lock()          //  修改x前假锁
m.Unlock()       // 改完解锁
```

{{< /admonition >}}

```go
func addWithLock() {
	for i := 0; i < 10000; i++ {
		m.Lock()
		x = x + 1
		m.Unlock()
	}
	//fmt.Println(x)
	wg.Done()

}
func TestMutex(t *testing.T) {
	wg.Add(2)
	go addWithLock()
	go addWithLock()
	wg.Wait()
	fmt.Println(x) // 此时
}
```

使用互斥锁能够保证同一时间有且只有一个 goroutine 进入临界区，其他的 goroutine 则在等待锁；当互斥锁释放后，等待的 goroutine 才可以获取锁进入临界区，多个 goroutine 同时等待一个锁时，唤醒的策略是随机的。

这样，最后的输出是不会变动的，可以分析一下，比如第一个线程执行到600，锁住了，释放完，变成601。这时不管第一个还是第二个goroutine拿到601，接着锁住，再加1，再释放，交给下一个goroutine(可能是当前goroutine，也可能是另外一个)，这样，虽然是在两个goroutine里面进行，但是总的和是不变的， 如果不加锁，两个goroutine是可以同时操作x的。

## 5.2 读写互斥锁

互斥锁是完全互斥的，但是实际上有很多场景是`读多写少`的，当我们并发的去读取一个资源而不涉及资源修改的时候是没有必要加互斥锁的，这种场景下使用读写锁是更好的一种选择。读写锁在 Go 语言中使用`sync`包中的`RWMutex`类型。

读写锁分为两种：读锁和写锁。当一个 goroutine 获取到读锁之后，其他的 goroutine 如果是获取读锁会继续获得锁，如果是获取写锁就会等待；而当一个 goroutine 获取写锁之后，其他的 goroutine 无论是获取读锁还是写锁都会等待。

| 方法名                              | 作用                           |
| ----------------------------------- | ------------------------------ |
| func (rw *RWMutex) Lock()           | 获取写锁                       |
| func (rw *RWMutex) Unlock()         | 释放写锁                       |
| func (rw *RWMutex) RLock()          | 获取读锁                       |
| func (rw *RWMutex) RUnlock()        | 释放读锁                       |
| func (rw *RWMutex) RLocker() Locker | 返回一个实现Locker接口的读写锁 |

```go
var (
	mutex   sync.Mutex
	rwMutex sync.RWMutex
)

// 使用互斥锁进行的写操作
func writeWithLock() {
	mutex.Lock()
	x = x + 1
	time.Sleep(10 * time.Millisecond)
	mutex.Unlock()
	wg.Done()
}

// 使用互斥锁的读操作
func readWithLock() {
	mutex.Lock()
	time.Sleep(time.Millisecond)
	mutex.Unlock()
	wg.Done()
}

// 使用读写互斥锁的写操作
func writeWithRwLock() {
	rwMutex.Lock()
	x = x + 1
	time.Sleep(10 * time.Millisecond)
	rwMutex.Unlock()
	wg.Done()
}

// 使用读写互斥锁的读操作
func readWithRwLock() {
	rwMutex.RLock()
	time.Sleep(time.Millisecond)
	rwMutex.RUnlock()
	wg.Done()
}

func do(wf, rf func(), wc, rc int) {
	start := time.Now()
	// wc个并发写操作
	for i := 0; i < wc; i++ {
		wg.Add(1)
		go wf()
	}

	// rc个并发读操作
	for i := 0; i < rc; i++ {
		wg.Add(1)
		go rf()
	}
	wg.Wait()
	cost := time.Since(start)
	fmt.Printf("x:%v const:%v\n", x, cost)
}

func TestRW(t *testing.T) {
	do(writeWithRwLock, readWithRwLock, 10, 100)
}

func TestWithoutRw(t *testing.T) {
	do(writeWithLock, readWithLock, 10, 100)
}
```

输出

```
=== RUN   TestRW
x:10 const:167.9521ms
--- PASS: TestRW (0.17s)
=== RUN   TestWithoutRw
x:20 const:1.7118294s
--- PASS: TestWithoutRw (1.71s)
PASS
```

{{< admonition type=tip title="分析"  >}}

- 从最终的执行结果可以看出，使用读写互斥锁在读多写少的场景下能够极大地提高程序的性能
- 不过需要注意的是如果`一个程序中的读操作和写操作数量级差别不大，那么读写互斥锁的优势就发挥不出来`
- 后面可以参考benchmark进行测试
- 这个程序加延时只是为了看的更明显，可以把延时取消的，执行的更快！

{{< /admonition >}}