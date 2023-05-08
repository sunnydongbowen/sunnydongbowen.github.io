---
title: "slice基础篇"
date: 2023-04-19
tags : [                                    
     "Go基础语法",
 ]
categories : [                              
     "Go语言",
 ]
---

{{< admonition type=info title=""  >}}

 在很多地方我都无意有意间去对比这两门语言, 其实每一门语言都有它的应用领域，没有必要看网上那些争论。这里我做一个应用场景的对比

{{< /admonition >}}

## 1. 定义

切片（Slice）是一个拥有相同类型元素的可变长度的序列。它是基于`数组`类型做的一层封装。它非常灵活，支持自动扩容(不管定义的容量是多少，都会自动扩容，只是扩容的次数频不频繁的区别)。

切片是一个**引用类型**，它的内部结构包含`地址`、`长度`和`容量`。切片一般用于快速地操作一块数据集合。

```go
var sint []int  // 定义一个int类型的切片
var sstr []string  // 定义一个string类型切片
fmt.Println(sint, sstr)
fmt.Println(sint == nil, sstr == nil)//nil ,其实没有分配内存的，初始化才能使用！数组有默认值！
```

`out`
```go
[] []
true true
```

从上面可以看到，切片和数组直观上的不同
-   切片定义的时候是没有长度的，而数组必须有长度，且是数组类型的一部分
-   切片没有默认值，而数组是有默认值的
-   切片可以使用`make()`初始化，数组不可以

## 2. 切片初始化

```go
sint = []int{1, 2, 3}
sstr = []string{"江宁", "雨花台", "鼓楼", "栖霞", "建邺"}
fmt.Println(sint)
fmt.Println(sstr)
```

`out`
```go
[1 2 3]
[江宁 雨花台 鼓楼 栖霞 建邺]
```

## 3. 初识len()和cap()

*`切片的容量是底层数组从切片的第一个元素指向s数组的最后元素的这个长度。和长度是不一样的`*

```go
fmt.Println(len(sint), cap(sint))
fmt.Println(len(sstr), cap(sstr))
```

`out`

```go
3 3
5 5
```

## 4. 由数组得到切片

>和python的是类似的，注意左闭右开

```go
aint := [...]int{1, 3, 5, 7, 9, 11, 13, 15}
s1 := aint[0:4] // 闭右开，类似py里的列表
s2 := aint[1:6]
s3 := aint[:4]
s4 := aint[3:]
s5 := aint[:]
fmt.Printf("s1=%v\n", s1)
fmt.Printf("s2=%v\n", s2)
fmt.Printf("s3=%v\n", s3)
fmt.Printf("s4=%v\n", s4)
fmt.Printf("s5=%v\n", s5)
```

`out`

```go
s1=[1 3 5 7]
s2=[3 5 7 9 11]
s3=[1 3 5 7]
s4=[7 9 11 13 15]
s5=[1 3 5 7 9 11 13 15]
```

## 5. 切片的长度和容量
>切片的容量是底层数组从切片的第一个元素指向数组的最后元素的长度。和长度是不一样的

```go
// 接上一段代码
fmt.Printf("len(s1)=%d, cap(s1)=%d\n", len(s1), cap(s1))
fmt.Printf("len(s4)=%d, cap(s4)=%d\n", len(s4), cap(s4))
fmt.Printf("len(s5)=%d, cap(s5)=%d\n", len(s5), cap(s5))
fmt.Printf("len(s3)=%d, cap(s3)=%d\n", len(s3), cap(s3))
```

`out`

```go
len(s1)=4, cap(s1)=8
len(s4)=5, cap(s4)=5
len(s5)=8, cap(s5)=8
len(s3)=4, cap(s3)=8
```

## 6. 切片再切片

这种情况下，切片的容量计算方式也是一样的。
```go
s6 := s2[0:3]
fmt.Printf("s6=%v\n", s6)
fmt.Printf("len(s6)=%d,cap(s6)=%d\n", len(s6), cap(s6))
```
`out`
```go
s6=[3 5 7]
len(s6)=3,cap(s6)=7
```

## 7. 切片是引用类型

> **切片是引用类型，都指向了底层的一个数组，底层变了，它也变**
```go
// 接上面代码
aint[2] = 1300
fmt.Printf("s6=%v\n", s6)
```

`out`
```go
s6=[3 1300 7]
```

> 可以看到，如果改了底层数组，其他的由底层数组得到的slice对于的数值都也会改变。



## 8. **使用make()函数构造切片**🎈

```go
make([]T, size, cap)
```

其中：
-   T:切片的元素类型
-   size:切片中元素的数量
-   cap:切片的容量

```go
s1 := make([]int, 5, 10)  // 5是长度，10是容量
s2 := make([]int, 0, 10)  // 底层数组的长度是10
fmt.Printf("s1=%v,len(s1)=%d,cap(s1)=%d\n", s1, len(s1), cap(s1))
fmt.Printf("s2=%v,len(s2)=%d,cap(s2)=%d\n", s2, len(s2), cap(s2))
```

`out`

```go
s1=[0 0 0 0 0],len(s1)=5,cap(s1)=10
s2=[],len(s2)=0,cap(s2)=10
```

{{< admonition type=warning  title="注意"  >}} 

- make初始化的时候，第二个参数建议是0。否则前面会有默认值影响的。
- make 初始化，是分配内存的，不为nil，go语言毕竟偏向底层，指针，长度，容量，后面会用的多！
{{< /admonition >}}

## 9. 共享底层数组

下面的代码中演示了拷贝前后两个变量共享底层数组，**对一个切片的修改会影响另一个切片的内容**，这点需要特别注意。

```go
s3 := []int{1, 3, 5}
s4 := s3  // 切片的赋值,s2和s4都指向了同一个底层数组，也就是同一个内存地址，本身是不存放数据的
fmt.Println(s3, s4)
s3[0] = 100
fmt.Println(s3, s4)
```

`out`

```go
[1 3 5] [1 3 5]
[100 3 5] [100 3 5]
```

## 10. 遍历
切片的遍历和数组遍历是一样的，都可以通过`索引`或是`for range`的方式进行遍历
{{< admonition type=note  title="索引遍历"  >}} 
```go
for i := 0; i < len(s3); i++ {
		fmt.Println(s3[i])

	}
```

{{< /admonition >}}


{{< admonition type=tip  title="for range遍历"  >}} 
```go
for i := 0; i < len(s3); i++ {
		fmt.Println(s3[i])

	}
```

{{< /admonition >}}

## 11. append()函数扩容切片 ✨

Go语言的内建函数`append()`可以为切片动态添加元素。 可以一次添加一个元素，可以添加多个元素，也可以添加另一个切片中的元素（后面加…）
{{< admonition type=example  title="扩容切片示例"  >}}

```go
s1 := []string{"江宁", "鼓楼", "雨花台"}
fmt.Printf("s1=%v,len(s1)=%d,cap(s1)=%d\n", s1, len(s1), cap(s1))
// 调用append函数必须使用原来的切片变量接收返回值
s1 = append(s1, "建邺")
fmt.Printf("s1=%v,len(s1)=%d,cap(s1)=%d\n", s1, len(s1), cap(s1))
//添加多个
s1 = append(s1, "栖霞", "连云港")
fmt.Printf("s1=%v,len(s1)=%d,cap(s1)=%d\n", s1, len(s1), cap(s1))
//添加切片
ss := []string{"北京", "上海", "广州"}
s1 = append(s1, ss...)
fmt.Printf("s1=%v,len(s1)=%d,cap(s2)=%d\n", s1, len(s1), cap(s1))
```

`out`

```go
s1=[江宁 鼓楼 雨花台],len(s1)=3,cap(s1)=3
s1=[江宁 鼓楼 雨花台 建邺],len(s1)=4,cap(s1)=6                             
s1=[江宁 鼓楼 雨花台 建邺 栖霞 连云港],len(s1)=6,cap(s1)=6 
```
注意点

- 使用原来的切片变量接收值
- append是切片的时候，需要用`slicename...`, 而不是直接写切片名称
- append 追加元素,原来的底层数组放不下的时候，go底层会把底层数组换一个。
 
 {{< /admonition >}}


## 12. append扩容策略

{{< admonition type=note  title="继续扩容"  >}}

上面的cap可以看到，s1的容量第一次追加后扩容到6，但后面再追加就没有变。再在此基础上，增加1个元素,容量扩容到12

```go
/*
1、如果小于1024，2倍扩容
2、如果大于1024，1.25扩容
3、如果申请的容量大于原来的2倍，直接扩容到新申请的容量
4、具体存储的值类型不同，扩容的策略也不同
*/
s1 = append(s1, "bowen")
fmt.Printf("s1=%v,len(s1)=%d cap(s1)=%d\\n", s1, len(s1), cap(s1))
```

`out`

```go
s1=[江宁 鼓楼 雨花台 建邺 栖霞 连云港 北京 上海 广州 bowen],len(s1)=10 cap(s1)=12
```

 {{< /admonition >}}


{{< admonition type=note  title="切片扩容源码实现"  >}}
`容量扩容策略，根据不同的数据类型策略不一样。可以看源码，*$GOROOT/*src/runtime/slice.go`其中扩容相关代码如下

```go
newcap := old.cap
doublecap := newcap + newcap
if cap > doublecap {
	newcap = cap
} else {
	if old.len < 1024 {
		newcap = doublecap
	} else {
		// Check 0 < newcap to detect overflow
		// and prevent an infinite loop.
		for 0 < newcap && newcap < cap {
			newcap += newcap / 4
		}
		// Set newcap to the requested cap when
		// the newcap calculation overflowed.
		if newcap <= 0 {
			newcap = cap
		}
	}
}
```

从上面的代码可以看出以下内容：

-   首先判断，如果新申请容量（cap）大于2倍的旧容量（old.cap），最终容量（newcap）就是新申请的容量（cap）。
-   否则判断，如果旧切片的长度小于1024，则最终容量(newcap)就是旧容量(old.cap)的两倍，即（newcap=doublecap），
-   否则判断，如果旧切片长度大于等于1024，则最终容量（newcap）从旧容量（old.cap）开始循环增加原来的1/4，即（newcap=old.cap,for {newcap += newcap/4}）直到最终容量（newcap）大于等于新申请的容量(cap)，即（newcap >= cap）
-   如果最终容量（cap）计算值溢出，则最终容量（cap）就是新申请容量（cap）。

>需要注意的是，切片扩容还会根据切片中元素的类型不同而做不同的处理，比如`int`和`string`类型的处理方式就不一样。

 {{< /admonition >}}

## 13. copy函数🗯️

Go语言内建的`copy()`函数可以迅速地将一个切片的数据复制到另外一个切片空间中,注意是新开辟的空间，和第9小节区的共享底层数组分开来

```go
s1 := []int{1, 3, 5}
s2 := s1 // 共享底层数组

var s3 = make([]int, 3, 3)
copy(s3, s1) // 重新开辟空间
fmt.Println(s1, s2, s3)
s1[1] = 100
fmt.Println(s1, s2, s3)
```

`out`

```go
[1 3 5] [1 3 5] [1 3 5]
[1 100 5] [1 100 5] [1 3 5]
```

## 14. 刪除元素🗨️

Go语言中并没有删除切片元素的专用方法，我们可以使用切片本身的特性来删除元素。 代码如下：
{{< admonition type=example  title="删除"  >}}

```go
// 接上面代碼
s1 = append(s1[:1], s1[2:]...)  // s1[2:] 换成s2[2:],也是可以成功删除的
fmt.Println(s1)  // 1,5
```

可以看到，s1第二个元素被删除了。
>总结一下就是：要从切片a中删除索引为`index`的元素，操作方法是`a = append(a[:index], a[index+1:]...)`，注意不要写反，写反就不是删除，而是增加了。

```go
x1 := []int{1, 3, 5}
fmt.Printf("%T\n", x1)

s1 := x1[:]
fmt.Println(s1, len(s1), cap(s1))
// 指向的内存地址是一样的
fmt.Printf("%p\n", &s1[0])
fmt.Printf("%p\n", &x1[0])
// 删除s1后，
s1 = append(s1[:1], s1[2:]...)
fmt.Println(s1, len(s1), cap(s1))
fmt.Println(x1) // 修改了底层数组[1,5,5] 把2覆盖1了。3没变
fmt.Printf("%p\n", &s1[0])

s1[1] = 100
fmt.Println(x1)  //[1,100,5]
```
`out`
```go
[]int
[1 3 5] 3 3  
0xc00000e120 
0xc00000e120 
[1 5] 2 3    
[1 5 5]      
0xc00000e120 
[1 100 5]
```
可以看到一个很奇怪的现象，当我们删除了s1[1]后，s1正常删除，但是x1却不是，这就是删除操作给底层数组带来的影响，底层数组的[2]还是5没有动，但是[1]元素被删除后，底层数组的[1]被[2]覆盖了。
 {{< /admonition >}}
## 15. 切片面试题
{{< admonition type=example  title="题目1"  >}}

写出下面程序运行结果

```go
var s = make([]int, 5, 10)
fmt.Println(s)  // 这一句是调试加的
for i := 0; i < 10; i++ {
	s = append(s, i)
}
fmt.Println(s)
fmt.Println(len(s), cap(s))
```

`out`

```go
[0 0 0 0 0]
[0 0 0 0 0 0 1 2 3 4 5 6 7 8 9] 
15 20
```

这题确实是个坑，以为结果会是这样`[0 1 2 3 4 5 6 7 8 9]` ,然而却是`[0 0 0 0 0 0 1 2 3 4 5 6 7 8 9]`，是因为一开始初始化的时候就有默认值了。
 {{< /admonition >}}

{{< admonition type=example  title="题目2"  >}}

写出下面程序运行结果

```go
func main() {
	var s = make([]int, 1)
	s = append(s, 1) 
	fmt.Println(s)
}
```

`out`

```go
[0 1]
```

这题也是个坑！
 {{< /admonition >}}
 
{{< admonition type=example  title="题目3"  >}}

```go
func main() {
	var s []int
	s = append(s, 1)
	fmt.Println(s)
}
```

`out`

```go
[1]
```

这题考的是与上面的区别，使用`append()`可以初始化切片
 {{< /admonition >}}

## 16. sort.Ints(s[:])排序

```go
import (
	"fmt"
	"sort"
)
func main() {
	var s = []int{3, 1, 7, 11, 8}
	sort.Ints(s[:])  // 是没有返回值的,直接改的就是切片，sort.Ints(s)也是可以的
	fmt.Println(s)   // [1 3 7 8 11]
}
```

## 17. 删除切片元素影响底层数组🦉

```go
func main() {
	var a = [5]int{1, 3, 5, 7, 9}
	s := a[:]
	s = append(s[:1], s[2:]...)
	fmt.Println(s)  // [1 5 7 9]
	fmt.Println(a)  // [1 5 7 9 9]
}
```

{{< admonition type=question  title="思考一下为什么"  open=false  >}}
这段代码的功能是将数组 a 的第 2 个元素删除，即将下标为 1 的元素 3 从切片 s 中删除，最后输出切片 s 和数组 a。

首先，通过 `s := a[:]` 将数组 a 转换为切片 s，从而可以对 s 进行修改。接着，使用 `append()` 函数将 s 中从下标为 1 的元素开始后面的元素向前移动一个位置，即将 s 中下标为 2 的元素 5 移动到下标为 1 的位置，下标为 3 的元素 7 移动到下标为 2 的位置，同时删除了原来的下标为 1 的元素 3，从而实现了删除操作。最后，输出修改后的切片 s 和数组 a，结果为 [1 5 7 9] 和 [1 5 7 9 9]，其中数组 a 最后一个元素被重复了，**这是由于切片 s 指向了数组 a 的一部分，因此在对切片 s 进行修改时也会对数组 a 产生影响**。
 {{< /admonition >}}