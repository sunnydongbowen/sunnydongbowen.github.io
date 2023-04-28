---
title: "值recevier和指针型recevier"
date: 2023-04-27
tags : [                                    
     "Go基础语法",
 ]
categories : [                              
     "Go语言",
 ]
---
> 💡 极课时间专栏做的笔记，这篇文章的 内容还是有一定深度的，实际使用稍微注意下即可

## 1. 分析值recevier和指针型receiver

Go 语言中**方法本质上就是函数，Go 方法实质上是以方法的 receiver 参数作为第一个参数的普通函数”**

receiver 参数类型对 Go 方法的影响，直接来看下面例子中的两个 Go 方法，以及它们等价转换后的函数：

```go
func (t T) M1() <=> F1(t T)
func (t *T) M2() <=> F2(t *T)
```

这个例子中有方法 M1 和 M2。M1 方法是 receiver 参数类型为 T 的一类方法的代表，而 M2 方法则代表了 receiver 参数类型为 *T 的另一类。下面我们分别来看看不同的 receiver 参数类型对 M1 和 M2 的影响。

{{< admonition type=note title="值类型recevier"  >}}
首先，当 receiver 参数的类型为 T 时：当我们选择以 T 作为 receiver 参数类型时，M1 方法等价转换为F1(t T)。我们知道，Go 函数的参数采用的是值拷贝传递，也就是说，F1 函数体中的 t 是 T 类型实例的一个副本。这样，我们在 F1 函数的实现中对参数 t 做任何修改，都只会影响副本，而不会影响到原 T 类型实例。

据此我们可以得出结论：当我们的方法 M1 采用类型为 T 的 receiver 参数时，代表 T 类型实例的 receiver 参数以值传递方式传递到 M1 方法体中的，实际上是 T 类型实例的副本，M1 方法体中对副本的任何修改操作，都不会影响到原 T 类型实例。
 {{< /admonition >}}
 
 {{< admonition type=note title="指针类型recevier"  >}}
第二，当 receiver 参数的类型为 *T 时：当我们选择以 *T 作为 receiver 参数类型时，M2 方法等价转换为F2(t *T)。

同上面分析，我们传递给 F2 函数的 t 是 T 类型实例的地址，这样 F2 函数体中对参数 t 做的任何修改，都会反映到原 T 类型实例上。

据此我们也可以得出结论：当我们的方法 M2 采用类型为 *T 的 receiver 参数时，代表 *T 类型实例的 receiver 参数以值传递方式传递到 M2 方法体中的，实际上是 T 类型实例的地址，M2 方法体通过该地址可以对原 T 类型实例进行任何修改操作。
 {{< /admonition >}}

 {{< admonition type=example title="代码示例"  >}}
```go
type T struct {
	a int
}

func (t T) M1() {
	t.a = 10
}

func (t *T) M2() {
	t.a = 11
}
func main() {
	var t T
	t = T{5}
	fmt.Println(t.a) // 5

	t.M1()
	fmt.Println(t.a) // 5

	t.M2()
	fmt.Println(t.a) //11
}
```

 {{< /admonition >}}
 
## 2. 选择recevier的原则

>  第一个原则 : 看要不要修改

如果 Go 方法要把对 receiver 参数代表的类型实例的修改，反映到原类型实例上，那么我们应该选择 *T 作为 receiver 参数的类型。

```go
type T struct {
	a int
}

func (t T) M1() {
	t.a = 10
}

func (t *T) M2() {
	t.a = 11
}
func main() {
	var t T
	t = T{5}
	fmt.Println(t.a) // 5

	t.M1()
	fmt.Println(t.a) // 5

	t.M2()
	fmt.Println(t.a) //11

	var t2 = &T{3}
	fmt.Println(t2.a) //3

	t2.M1()
	fmt.Println(t2.a) //3

	t2.M2()
	fmt.Println(t2.a) //11

}
```

我们先来看看类型为 T 的实例 t1。我们看到它不仅可以调用 receiver 参数类型为 T 的方法 M1，它还可以直接调用 receiver 参数类型为 *T 的方法 M2，并且调用完 M2 方法后，t1.a 的值被修改为 11 了。

其实，T 类型的实例 t1 之所以可以调用 receiver 参数类型为 *T 的方法 M2，都是 Go 编译器在背后自动进行转换的结果。或者说，t1.M2() 这种用法是 Go 提供的“语法糖”：Go 判断 t1 的类型为 T，也就是与方法 M2 的 receiver 参数类型 *T 不一致后，会自动将t1.M2()转换为(&t1).M2()。

同理，类型为 *T 的实例 t2，它不仅可以调用 receiver 参数类型为 *T 的方法 M2，还可以调用 receiver 参数类型为 T 的方法 M1，这同样是因为 Go 编译器在背后做了转换。也就是，Go 判断 t2 的类型为 *T，与方法 M1 的 receiver 参数类型 T 不一致，就会自动将t2.M1()转换为(*t2).M1()。

通过这个实例，我们知道了这样一个结论：无论是 T 类型实例，还是 *T 类型实例，都既可以调用 receiver 为 T 类型的方法，也可以调用 receiver 为 *T 类型的方法。这样，我们在为方法选择 receiver 参数的类型的时候，就不需要担心这个方法不能被与 receiver 参数类型不一致的类型实例调用了。

上面啰哩啰嗦说了那么多，其实就是想说，**调用者是值类型的还是指针型的不需要管！**

> 第二个原则 : 还要看size大小

前面我们第一个原则说的是，当我们要在方法中对 receiver 参数代表的类型实例进行修改，那我们要为 receiver 参数选择 *T 类型，但是如果我们不需要在方法中对类型实例进行修改呢？这个时候我们是为 receiver 参数选择 T 类型还是 *T 类型呢？这也得分情况。

一般情况下，我们通常会为 receiver 参数选择 T 类型，因为这样可以缩窄外部修改类型实例内部状态的“接触面”，也就是尽量少暴露可以修改类型内部状态的方法。

不过也有一个例外需要你特别注意。考虑到 Go 方法调用时，receiver 参数是以值拷贝的形式传入方法中的。那么，如果 receiver 参数类型的 size 较大，以值拷贝形式传入就会导致较大的性能开销，这时我们选择 *T 作为 receiver 类型可能更好些。

## 3. 方法集合

我们先通过一个示例，直观了解一下为什么要有方法集合，它主要用来解决什么问题：

```go
type Interface interface {
    M1()
    M2()
}

type T struct{}

func (t T) M1()  {}
func (t *T) M2() {}

func main() {
    var t T
    var pt *T
    var i Interface

    i = pt
    i = t // cannot use t (type T) as type Interface in assignment: T does not implement Interface (M2 method has pointer receiver)
}
```

在这个例子中，我们定义了一个接口类型 Interface 以及一个自定义类型 T。Interface 接口类型包含了两个方法 M1 和 M2，代码中还定义了基类型为 T 的两个方法 M1 和 M2，但它们的 receiver 参数类型不同，一个为 T，另一个为 *T。在 main 函数中，我们分别将 T 类型实例 t 和 *T 类型实例 pt 赋值给 Interface 类型变量 i。

运行一下这个示例程序，我们在i = t这一行会得到 Go 编译器的错误提示，Go 编译器提示我们：T 没有实现 Interface 类型方法列表中的 M2，因此类型 T 的实例 t 不能赋值给 Interface 变量。

可是，为什么呀？为什么 *T 类型的 pt 可以被正常赋值给 Interface 类型变量 i，而 T 类型的 t 就不行呢？如果说 T 类型是因为只实现了 M1 方法，未实现 M2 方法而不满足 Interface 类型的要求，那么 *T 类型也只是实现了 M2 方法，并没有实现 M1 方法啊？

有些事情并不是表面看起来这个样子的。了解方法集合后，这个问题就迎刃而解了。同时，方法集合也是用来判断一个类型是否实现了某接口类型的唯一手段，可以说，“方法集合决定了接口实现”。

Go 中任何一个类型都有属于自己的方法集合，或者说方法集合是 Go 类型的一个“属性”。但不是所有类型都有自己的方法呀，比如 int 类型就没有。所以，`对于没有定义方法的 Go 类型，我们称其拥有空方法集合`。

接口类型相对特殊，它只会列出代表接口的方法列表，不会具体定义某个方法，它的方法集合就是它的方法列表中的所有方法，我们可以一目了然地看到。因此，我们下面重点讲解的是非接口类型的方法集合。

为了方便查看一个非接口类型的方法集合，我这里提供了一个函数 dumpMethodSet，用于输出一个非接口类型的方法集合：

```go
func dumpMethodSet(i interface{}) {
    dynTyp := reflect.TypeOf(i)

    if dynTyp == nil {
        fmt.Printf("there is no dynamic type\n")
        return
    }

    n := dynTyp.NumMethod()
    if n == 0 {
        fmt.Printf("%s's method set is empty!\n", dynTyp)
        return
    }

    fmt.Printf("%s's method set:\n", dynTyp)
    for j := 0; j < n; j++ {
        fmt.Println("-", dynTyp.Method(j).Name)
    }
    fmt.Printf("\n")
}
```

下面我们利用这个函数，输出一下 Go 原生类型以及自定义类型的方法集合，看下面代码：

```go
type T struct{}

func (T) M1() {}
func (T) M2() {}

func (*T) M3() {}
func (*T) M4() {}

func main() {
    var n int
    dumpMethodSet(n)
    dumpMethodSet(&n)

    var t T
    dumpMethodSet(t)
    dumpMethodSet(&t)
}
```

运行这段代码，我们得到如下结果：

```go
int's method set is empty!
*int's method set is empty!
main.T's method set:
- M1
- M2

*main.T's method set:
- M1
- M2
- M3
- M4
```

我们看到以 int、*int 为代表的 Go 原生类型由于没有定义方法，所以它们的方法集合都是空的。自定义类型 T 定义了方法 M1 和 M2，因此它的方法集合包含了 M1 和 M2，也符合我们预期。但 *T 的方法集合中除了预期的 M3 和 M4 之外，居然还包含了类型 T 的方法 M1 和 M2！

不过，这里程序的输出并没有错误。这是因为，Go 语言规定，*T 类型的方法集合包含所有以 *T 为 receiver 参数类型的方法，以及所有以 T 为 receiver 参数类型的方法。这就是这个示例中为何 *T 类型的方法集合包含四个方法的原因。这个时候，你是不是也找到了前面那个示例中为何i = pt没有报编译错误的原因了呢？我们同样可以使用 dumpMethodSet 工具函数，输出一下那个例子中 pt 与 t 各自所属类型的方法集合：

```go
type Interface interface {
    M1()
    M2()
}

type T struct{}

func (t T) M1()  {}
func (t *T) M2() {}

func main() {
    var t T
    var pt *T
    dumpMethodSet(t)
    dumpMethodSet(pt)
}
```

运行上述代码，我们得到如下结果：

```bash
main.T's method set:
- M1

*main.T's method set:
- M1
- M2
```

通过这个输出结果，我们可以一目了然地看到 T、*T 各自的方法集合。

我们看到，T 类型的方法集合中只包含 M1，没有 Interface 类型方法集合中的 M2 方法，这就是 Go 编译器认为变量 t 不能赋值给 Interface 类型变量的原因。

在输出的结果中，我们还看到 *T 类型的方法集合除了包含它自身定义的 M2 方法外，还包含了 T 类型定义的 M1 方法，*T 的方法集合与 Interface 接口类型的方法集合是一样的，因此 pt 可以被赋值给 Interface 接口类型的变量 i。

到这里，我们已经知道了所谓的方法集合决定接口实现的含义就是：`如果某类型 T 的方法集合与某接口类型的方法集合相同，或者类型 T 的方法集合是接口类型 I 方法集合的超集，那么我们就说这个类型 T 实现了接口 I。或者说，方法集合这个概念在 Go 语言中的主要用途，就是用来判断某个类型是否实现了某个接口。`

## 4. 第三个原则

这个原则的选择依据就是 T 类型是否需要实现某个接口，也就是是否存在将 T 类型的变量赋值给某接口类型变量的情况。

-   如果 T 类型需要实现某个接口，那我们就要使用 T 作为 receiver 参数的类型，来满足接口类型方法集合中的所有方法。
-   如果 T 不需要实现某一接口，但 *T 需要实现该接口，那么根据方法集合概念，*T 的方法集合是包含 T 的方法集合的，这样我们在确定 Go 方法的 receiver 的类型时，参考原则一和原则二就可以了。
-   如果说前面的两个原则更多聚焦于类型内部，从单个方法的实现层面考虑，那么这第三个原则则是更多从全局的设计层面考虑，聚焦于这个类型与接口类型间的耦合关系。

## 5. 总结

用指针recevier，用指针recevier，用指针recevier！重要的事情说三遍！

