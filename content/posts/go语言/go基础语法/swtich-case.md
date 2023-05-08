---
title: "swtich-case"
date: 2023-04-22
tags : [                                    
     "Go基础语法",
 ]
categories : [                              
     "Go语言",
 ]
---
# 1.switch初体验

## 1.1 常规用法
> 这是最常用的方式

```go
func main() {
	i :=1
	switch i {
	case 1:
		fmt.Println("你好")
	case 2:
		fmt.Println("hello")
	case 3:
		fmt.Println("大家好")
	default:
		fmt.Println("Sorry")
	}
```

## 1.2 简写
>     在特定场景下会用到这种写法

```go
switch a:=9;a {
case 1,3,5,7,9:
	fmt.Println("我是奇数")
case 2,4,6,8,10:
	fmt.Println("我是偶数")
}
```

_`swtich 也有判断大小，或者是==,<,>等`_

# 3. 认识switch语句 😀

通过上面例子，认识了swtich语句。现在通过下面例子来直观地感受一下 switch 语句的优点。在一些执行分支较多的场景下，使用 switch 分支控制语句可以让代码更简洁，可读性更好。

比如下面例子中的 readByExt 函数会根据传入的文件扩展名输出不同的日志，它使用了 if 语句进行分支控制：

```go
func readByExt(ext string) {
    if ext == "json" {
        println("read json file")
    } else if ext == "jpg" || ext == "jpeg" || ext == "png" || ext == "gif" {
        println("read image file")
    } else if ext == "txt" || ext == "md" {
        println("read text file")
    } else if ext == "yml" || ext == "yaml" {
        println("read yaml file")
    } else if ext == "ini" {
        println("read ini file")
    } else {
        println("unsupported file extension:", ext)
    }
}
```

如果用 switch 改写上述例子代码，我们可以这样来写：

```go
func readByExtBySwitch(ext string) {
    switch ext {
    case "json":
        println("read json file")
    case "jpg", "jpeg", "png", "gif":
        println("read image file")
    case "txt", "md":
        println("read text file")
    case "yml", "yaml":
        println("read yaml file")
    case "ini":
        println("read ini file")
    default:
        println("unsupported file extension:", ext)
    }
}
```

从代码呈现的角度来看，针对这个例子，使用 switch 语句的实现要比 if 语句的实现更加简洁紧凑。

简单来说，readByExtBySwitch 函数就是将输入参数 ext 与每个 case 语句后面的表达式做比较，如果相等，就执行这个 case 语句后面的分支，然后函数返回。

```go
func case1() int {
    println("eval case1 expr")
    return 1
}

func case2_1() int {
    println("eval case2_1 expr")
    return 0 
}
func case2_2() int {
    println("eval case2_2 expr")
    return 2 
}

func case3() int {
    println("eval case3 expr")
    return 3
}

func switchexpr() int {
    println("eval switch expr")
    return 2
}

func main() {
    switch switchexpr() {
    case case1():
        println("exec case1")
    case case2_1(), case2_2():
        println("exec case2")
    case case3():
        println("exec case3")
    default:
        println("exec default")
    }
}
```

这段代码的输出结果如下：

```go
eval switch expr 
eval case1 expr 
eval case2_1 expr 
eval case2_2 expr 
exec case2
```

如果要分析执行的顺序,最好在debug一下，看一下执行过程，这里不做废篇章做陈述了，debug能很清楚的看到执行顺序，变量的变化。

![Untitled](/go基础/20230422122512.png)

这里其实有一个优化小技巧，考虑到 switch 语句是按照 case 出现的先后顺序对 case 表达式进行求值的，那么如果我们将`匹配成功概率高的 case 表达式排在前面，就会有助于提升 switch 语句执行效率`。这点对于 case 后面是表达式列表的语句同样有效，我们可以将匹配概率最高的表达式放在表达式列表的最左侧。

无论 default 分支出现在什么位置，它都只会在所有 case 都没有匹配上的情况下才会被执行的。

# 4. switch 语句的灵活性

## 4.1 switch 语句表达式类型多样

Go 语言中，只要类型支持比较操作，都可以作为 switch 语句中的表达式类型。比如整型、布尔类型、字符串类型、复数类型、元素类型都是可比较类型的数组类型，甚至字段类型都是可比较类型的结构体类型，也可以。下面就是一个使用自定义结构体类型作为 switch 表达式类型的例子：

```go
type person struct {
    name string
    age  int
}

func main() {
    p := person{"tom", 13}
    switch p {
    case person{"tony", 33}:
        println("match tony")
    case person{"tom", 13}:
        println("match tom")
    case person{"lucy", 23}:
        println("match lucy")
    default:
        println("no match")
    }
}
```

不过，实际开发过程中，以结构体类型为 switch 表达式类型的情况并不常见，这里举这个例子仅是为了说明 Go switch 语句对各种类型支持的广泛性。

## 4.2 支持省略后面的表达式

当 switch 表达式的类型为布尔类型时，如果求值结果始终为 true，那么我们甚至可以省略 switch 后面的表达式，比如下面例子

```go
// 带有initStmt语句的switch语句
switch initStmt; {
    case bool_expr1:
    case bool_expr2:
    ... ...
}

// 没有initStmt语句的switch语句
switch {
    case bool_expr1:
    case bool_expr2:
    ... ...
}
```

## 4.3 支持声明临时变量

这一点和if，for循环类似，就不做陈述了

## 4.4 支持表达式列表

```go
func checkWorkday(a int) {
    switch a {
    case 1, 2, 3, 4, 5:
        println("it is a work day")
    case 6, 7:
        println("it is a weekend day")
    default:
        println("are you live on earth")
    }
}
```

# 5. falthrough

Go 语言中的swtich取消了默认执行下一个 case 代码逻辑的“非常规”语义，每个 case 对应的分支代码执行完后就结束 switch 语句。如果在少数场景下，你需要执行下一个 case 的代码逻辑，你可以显式使用 Go 提供的关键字 fallthrough 来实现，这也是 Go“显式”设计哲学的一个体现。下面就是一个使用 fallthrough 的 switch 语句的例子，我们简单来看一下：

```go
func case1() int {
    println("eval case1 expr")
    return 1
}

func case2() int {
    println("eval case2 expr")
    return 2
}

func switchexpr() int {
    println("eval switch expr")
    return 1
}

func main() {
    switch switchexpr() {
    case case1():
        println("exec case1")
        fallthrough
    case case2():
        println("exec case2")
        fallthrough
    default:
        println("exec default")
    }
}
```

执行一下这个示例程序，我们得到这样的结果：

```go
eval switch expr
eval case1 expr
exec case1
exec case2
exec default
```

这里有一个注意点，由于 fallthrough 的存在，Go 不会对 case2 的表达式做求值操作，而会直接执行 case2 对应的代码分支。而且，在这里 case2 中的代码分支也显式使用了 fallthrough，于是最后一个代码分支，也就是 default 分支对应的代码也被执行了。

这里有一个注意点，由于 fallthrough 的存在，Go 不会对 case2 的表达式做求值操作，而会直接执行 case2 对应的代码分支。而且，在这里 case2 中的代码分支也显式使用了 fallthrough，于是最后一个代码分支，也就是 default 分支对应的代码也被执行了。

# 6. type switch 😀

```go
func main() {
	var x interface{} = 13
	switch x.(type) {
	case nil:
		println("x is nil")
	case int:
		println("the type of x is int")
	case string:
		println("the type of x is string")
	case bool:
		println("the type of x is bool ")
	default:
		println("don't support the type")
	}
}
```

`out`

```go
the type of x is int
```

switch 关键字后面跟着的表达式为`x.(type)`，这种表达式形式是 switch 语句专有的，而且也只能在 switch 语句中使用。这个表达式中的 x 必须是一个接口类型变量，表达式的求值结果是这个接口类型变量对应的动态类型。

什么是一个接口类型的动态类型呢？我们简单解释一下。以上面的代码var x interface{} = 13为例，x 是一个接口类型变量，它的静态类型为interface{}，如果我们将整型值 13 赋值给 x，x 这个接口变量的动态类型就为 int。关于接口类型变量的动态类型，后面详细介绍。

接着，case 关键字后面接的就不是普通意义上的表达式了，而是一个个具体的类型。这样，Go 就能使用变量 x 的动态类型与各个 case 中的类型进行匹配，之后的逻辑就都是一样的了。

```go
func main() {
    var x interface{} = 13
    switch v := x.(type) {
    case nil:
        println("v is nil")
    case int:
        println("the type of v is int, v =", v)
    case string:
        println("the type of v is string, v =", v)
    case bool:
        println("the type of v is bool, v =", v)
    default:
        println("don't support the type")
    }
}
```

这里我们将 switch 后面的表达式由x.(type)换成了v := x.(type)。对于后者，你千万不要认为变量 v 存储的是类型信息，其实 v 存储的是变量 x 的动态类型对应的值信息，这样我们在接下来的 case 执行路径中就可以使用变量 v 中的值信息了。

可以发现，在前面的 type switch 演示示例中，我们一直使用 interface{}这种接口类型的变量，Go 中所有类型都实现了 interface{}类型，所以 case 后面可以是任意类型信息。

但如果在 switch 后面使用了某个特定的接口类型 I，那么 case 后面就只能使用实现了接口类型 I 的类型了，否则 Go 编译器会报错。你可以看看这个例子：

```go
type I interface {
      M()
  }
  
  type T struct {
  }
  
 func (T) M() {
 }
 
 func main() {
     var t T
     var i I = t
     switch i.(type) {
     case T:
         println("it is type T")
     case int:
         println("it is type int")
     case string:
         println("it is type string")
     }
 }
```

在这个例子中，我们在 type switch 中使用了自定义的接口类型 I。那么，理论上所有 case 后面的类型都只能是实现了接口 I 的类型。但在这段代码中，只有类型 T 实现了接口类型 I，Go 原生类型 int 与 string 都没有实现接口 I，于是在编译上述代码时，编译器会报出如下错误信息：

```go
19:2: impossible type switch case: i (type I) cannot have dynamic type int (missing M method)
21:2: impossible type switch case: i (type I) cannot have dynamic type string (missing M method)
```

# 7. 逃不出循环的break

```go
func main() {
    var sl = []int{5, 19, 6, 3, 8, 12}
    var firstEven int = -1

    // find first even number of the interger slice
    for i := 0; i < len(sl); i++ {
        switch sl[i] % 2 {
        case 0:
            firstEven = sl[i]
            break
        case 1:
            // do nothing
        }        
    }         
    println(firstEven) 
}
```

我们运行一下这个修改后的程序，得到结果为 12。这个输出的值与我们的预期的好像不太一样。这段代码中，切片中的第一个偶数是 6，而输出的结果却成了切片的最后一个偶数 12。为什么会出现这种结果呢？

这就是 Go 中 break 语句与 switch 分支结合使用会出现一个“小坑”。和我们习惯的 C 家族语言中的 break 不同，Go 语言规范中明确规定，不带 label 的 break 语句中断执行并跳出的，是同一函数内 break 语句所在的最内层的 `for、switch 或 select`。所以，上面这个例子的 break 语句实际上只跳出了 switch 语句，并没有跳出外层的 for 循环，这也就是程序未按我们预期执行的原因。解决方法，加LOOP

```go
func main() {
    var sl = []int{5, 19, 6, 3, 8, 12}
    var firstEven int = -1

    // find first even number of the interger slice
loop:
    for i := 0; i < len(sl); i++ {
        switch sl[i] % 2 {
        case 0:
            firstEven = sl[i]
            break loop
        case 1:
            // do nothing
        }
    }
    println(firstEven) // 6
}
```

在改进后的例子中，我们定义了一个 label：loop，这个 label 附在 for 循环的外面，指代 for 循环的执行。当代码执行到“break loop”时，程序将停止 label loop 所指代的 for 循环的执行。之前在写select的时候，也遇到过一次这样的坑！这是习惯了其他语言的break用法。