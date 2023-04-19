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