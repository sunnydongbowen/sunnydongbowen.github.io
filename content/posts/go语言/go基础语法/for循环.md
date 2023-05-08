---
title: "for循环"
date: 2023-04-22
tags : [                                    
     "Go基础语法",
 ]
categories : [                              
     "Go语言",
 ]
---
> 在其他编程语言中，循环一般有for和while循环两种方式，但是在Go中，就只有一种，那就是for循环。

## 1. for 循环流程

```go
var sum int
for i := 0; i < 10; i++ {
    sum += i
}
println(sum)
```

![Untitled](/go基础/20230422113943.png)

从图中我们看到，经典 for 循环语句有四个组成部分（分别对应图中的①~④）。我们按顺序拆解一下这张图。

图中①对应的组成部分执行于循环体（③ ）之前，并且在整个 for 循环语句中仅会被执行一次，它也被称为循环前置语句。我们通常会在这个部分声明一些循环体（③ ）或循环控制条件（② ）会用到的自用变量，也称循环变量或迭代变量，比如这里声明的整型变量 i。与 if 语句中的自用变量一样，for 循环变量也采用短变量声明的形式，循环变量的作用域仅限于 for 语句隐式代码块范围内。

图中②对应的组成部分，是用来决定循环是否要继续进行下去的条件判断表达式。和 if 语句的一样，这个用于条件判断的表达式必须为布尔表达式，如果有多个判断条件，我们一样可以由逻辑操作符进行连接。当表达式的求值结果为 true 时，代码将进入循环体（③）继续执行，相反则循环直接结束，循环体（③）与组成部分④都不会被执行。

前面也多次提到了，图中③对应的组成部分是 for 循环语句的循环体。如果相关的判断条件表达式求值结构为 true 时，循环体就会被执行一次，这样的一次执行也被称为一次迭代（Iteration）。在上面例子中，循环体执行的动作是将这次迭代中变量 i 的值累加到变量 sum 中。

图中④对应的组成部分会在每次循环体迭代之后执行，也被称为循环后置语句。这个部分通常用于更新 for 循环语句组成部分①中声明的循环变量，比如在这个例子中，我们在这个组成部分对循环变量 i 进行加 1 操作。

Go 语言的 for 循环也在 C 语言的基础上有一些突破和创新。具体一点，Go 语言的 for 循环支持声明多循环变量，并且可以应用在循环体以及判断条件中，比如下面就是一个使用多循环变量的、稍复杂的例子：

```go
func main() {
	sum := 0
	for i, j, k := 0, 1, 2; (i < 20) && (j < 10) && (k < 30); i, j, k = i+1, j+1, k+5 {
		sum += (i + j + k)
		println(sum)
	}
}
```

## 2. 经典格式

`基本格式用的最多`

```go
// 输出0~9
for i := 0; i < 10; i++ {
		fmt.Println(i)
	}
```

### 2.1 格式变形1`省略了初始值`

```go
// 输出5·9
var i = 5
for ; i < 10; i++ {
	fmt.Println(i)
}
```

省略循环前置语句。比如下面例子中，我们就没有使用前置语句声明循环变量，而是直接使用了已声明的变量 i 充当循环变量的作用

### 2.2 变形2`把自变量放到了循环体内`

前置语句和后置语句都省略的写法

```go
i := 0
for ; i < 10; {
    println(i)
    i++
}
```

当循环前置与后置语句都省略掉，仅保留循环判断条件表达式时，我们可以省略经典 for 循环形式中的分号

```go
var i = 5
for i < 10 {
	fmt.Println(i)
	i++
}
```

## 2. 无限循环

`会不停的输出123`

```go
for {
			fmt.Println("123")
		}
```

当 for 循环语句的循环判断条件表达式的求值结果始终为 true 时，我们就可以将它省略掉了。它等价于

```go
for true {
   // 循环体代码
}
```

或者

```go
for ; ; {
   // 循环体代码
}
```

日常使用时，建议用它的最简形式，也就是for {...}，更加简单。后面两种知道即可。记住经典写法就好了，别给自己增加负担，那些骚操作。

## 3. for range

可以对比一下python，python中也有类似 for range这样的用法!

经典写法

```go
var sl = []int{1, 2, 3, 4, 5}
for i := 0; i < len(sl); i++ {
	fmt.Printf("sl[%d] = %d", i, sl[i])
}
```

变量 i 作为切片下标，逐一将切片中的元素读取了出来。不过，这样就有点麻烦了。其实，针对像切片这样的复合数据类型，还有 Go 原生的字符串类型（string），Go 语言提供了一个更方便的“语法糖”形式：for range。现在我们就来写一个等价于上面代码的 for range 循环：for i, v := range sl { fmt.Printf("sl[%d] = %d\n", i, v)

```go
for i, v := range sl {
	fmt.Printf("sl[%d] = %d", i, v)
}
```

我们看到，for range 循环形式与 for 语句经典形式差异较大，除了循环体保留了下来，其余组成部分都“不见”了。其实那几部分已经被融合到 for range 的语义中了。

具体来说，这里的 i 和 v 对应的是经典 for 语句形式中循环前置语句的循环变量，它们的初值分别为切片 sl 的第一个元素的下标值和元素值。并且，隐含在 for range 语义中的循环控制条件判断为：是否已经遍历完 sl 的所有元素，等价于i < len(sl)这个布尔表达式。另外，每次迭代后，for range 会取出切片 sl 的下一个元素的下标和值，分别赋值给循环变量 i 和 v，这与 for 经典形式下的循环后置语句执行的逻辑是相同的。

### 3.1 变形1

当我们不关心元素的值时，我们可以省略代表元素值的变量 v，只声明代表下标值的变量 i：

```go
for i := range sl {
  // ... 
}
```

### 3.2 变形2

如果我们不关心元素下标，只关心元素值，那么我们可以用空标识符替代代表下标值的变量 i。这里一定要注意，这个空标识符不能省略，否则就与上面的“变种一”形式一样了，Go 编译器将无法区分：

```go
for _, v := range sl {
  // ... 
}
```

### 3.3 变形3

到这里，你肯定要问：如果我们既不关心下标值，也不关心元素值，那是否能写成下面这样呢：

```go
for _, _ = range sl {
  // ... 
}
```

这种形式在语法上没有错误，就是看起来不太优雅。Go 核心团队早在Go 1.4 版本中就提供了一种优雅的等价形式，你后续直接使用这种形式就好了：

```go
for range sl {
  // ... 
}
```

不过一般很少用，我们既然进行了循环，肯定是有用的啊，怎么会一个值都不取，干循环呢？

好了，讲完了 for range 针对切片这种复合类型的各种形式后，我们再来看看 for range 应该如何用于对其他复合类型，或者是对 string 类型进行循环操作。for range 针对不同复合数据类型进行循环操作时，虽然语义是相同的，但它声明的循环变量的含义会有所不同，我们有必要逐一看一下。

## 4. for range遍历String类型😀

看下面例子

```go
var s = "中国人"
for i, v := range s {
    fmt.Printf("%d %s 0x%x\n", i, string(v), v)
}
```

运行这个例子，输出结果是这样的：

```go
0 中 0x4e2d
3 国 0x56fd 
6 人 0x4eba
```

我们看到：for range 对于 string 类型来说，每次循环得到的 v 值是一个 Unicode 字符码点，也就是 rune 类型值，而不是一个字节，返回的第一个值 i 为该 Unicode 字符码点的内存编码（UTF-8）的第一个字节在字符串内存序列中的位置。

可以看到，这个就和切片类型的不同了，首先下标i的不同，第二个，值v的不同。另外我要在这里再次提醒你，使用 for 经典形式与使用 for range 形式，对 string 类型进行循环操作的语义是不同的。

## 5. for range遍历map类型 😀

map 就是一个键值对（key-value）集合，最常见的对 map 的操作，就是通过 key 获取其对应的 value 值。但有些时候，我们也要对 map 这个集合进行遍历，这就需要 for 语句的支持了.

但在 Go 语言中，我们要对 map 进行循环操作，for range 是唯一的方法，for 经典循环形式是不支持对 map 类型变量的循环控制的。下面是通过 for range，对一个 map 类型变量进行循环操作的示例：

```go
var m = map[string]int{
		"Rob":  67,
		"Russ": 39,
		"John": 29,
	}

	for k, v := range m {
		fmt.Println(k, v)
	}
```

`out`

```go
John 29
Rob 67
Russ 39
```

通过输出结果我们看到：for range 对于 map 类型来说，每次循环，循环变量 k 和 v 分别会被赋值为 map 键值对集合中一个元素的 key 值和 value 值。而且，map 类型中没有下标的概念，通过 key 和 value 来循环操作 map 类型变量也就十分自然了。

## 6. for range 遍历channel 😀

除了可以针对 string、数组 / 切片，以及 map 类型变量进行循环操作控制之外，for range 还可以与 channel 类型配合工作。channel 是 Go 语言提供的并发设计的原语，它用于多个 Goroutine 之间的通信，我们在后面的课程中还会详细讲解 channel。当 channel 类型变量作为 for range 语句的迭代对象时，for range 会尝试从 channel 中读取数据，使用形式是这样的：

```go
var c = make(chan int)
for v := range c {
   // ... 
}
```

在这个例子中，for range 每次从 channel 中读取一个元素后，会把它赋值给循环变量 v，并进入循环体。当 channel 中没有数据可读的时候，for range 循环会阻塞在对 channel 的读操作上。直到 channel 关闭时，for range 循环才会结束，这也是 for range 循环与 channel 配合时隐含的循环判断条件。

详细的看后面的channel部分的介绍吧。

```go
func main() {
	//for range ,这个用的也比较多
	s := "Hello 博文"
	for i, v := range s {
		fmt.Printf("%d %c\n", i, v) //输出索引，和索引对应的值，c表示输出单个字符，d表示十进制整数
	}
}
```

## 6. break

和python中break用法相通，这里面就用到了for循环和if判断结合使用，经常这么用。

```go
func main() {
	// 当i = 5 时就跳出for循环
	for i := 1; i <= 10; i++ {
		if i == 5 {
			break // 跳出for循环
		}
		fmt.Println(i)
	}
	fmt.Println("over") // 输出到4就结束了，不会继续循环了
}
```

## 7. continue

和python中continue用法相通

```go
func main() {
	for i:=1;i<=10;i++{
//遇4跳过本轮，进入下一轮循环
		if i==4{
			continue
		}
		fmt.Println(i)
	}
	fmt.Println("循环结束")
}
```

## 8. for循环中使用哑元变量

> 如果不想定义变量或者用不到，可以使用匿名变量进行接收。这样也不需要捕捉err。

```go
func main() {
	s:="hello"
	for i,v :=range s{
		fmt.Printf("%d %c\n",i,v)
	}
	for _,v:=range s {
		fmt.Printf("%c\n",v)
	}
}
```

## 9. for循环中使用flag

这种写法在实际编码中也会经常用，设置一个标志flag，当符合一定条件时，flag置为true。

```go
var flag = false
func main() {
	for i:=0;i<1;i++{
		for j:='A';j<'Z';j++{
			if j=='C'{  //遇到C，flag置为true，跳出内层循环
				flag=true
				break
			}
			fmt.Printf("%d-%c\n",i,j)
        if flag==true{
        	break
             }
		}
	}
}
```

## 10. goto

第九个例子是flag为true时，`跳出`外循环，这一个是当flag为true时，`跳转到`指定语句执行

```go
var flag = false
func main() {
	for i:=0;i<1;i++{
		for j:='A';j<'Z';j++{
			if j=='C'{  //遇到C，flag置为true，跳出内层循环
				flag=true
				break
			}
			fmt.Printf("%d-%c\n",i,j)
        if flag==true{
        	goto  xx
             }
		}
	}
	xx:
		fmt.Println("over")
}
```