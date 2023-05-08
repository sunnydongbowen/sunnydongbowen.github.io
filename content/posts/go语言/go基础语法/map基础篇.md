---
title: "map基础篇"
date: 2023-04-21
tags : [                                    
     "Go基础语法",
 ]
categories : [                              
     "Go语言",
 ]
---
## 1. map定义

Go语言中提供的映射关系容器为`map`，其内部使用`散列表（hash）`实现。map是一种无序的基于`key-value`的数据结构，Go语言中的**map是引用类型，必须初始化才能使用**。定义格式如下

Go语言中 `map`的定义语法如下：

```go
map[KeyType]ValueType
```

其中，

-   KeyType:表示键的类型。
-   ValueType:表示键对应的值的类型。

map类型的变量默认初始值为nil，需要使用make()函数来分配内存。语法为：

```go
make(map[KeyType]ValueType, [cap])
```

其中cap表示map的`容量`，该参数虽然不是必须的，但是我们应该在初始化map的时候就为其指定一个合适的容量。


## 2. map初始化

必须要初始化，否则是无法使用的。

```go
var info map[string](int)  //定义
fmt.Println(info == nil)
info = make(map[string]int, 10) // make初始化map
```
## 3. 赋值

>  和py中相同的

```go
// 接上面代码
info["小明"] = 10
info["小张"] = 5
fmt.Println(info)
fmt.Println(info["小明"])
```

`out`

```go
map[小张:5 小明:10] 
10
```


## 4. 判断key是否存在

```go
// 接上面代码
value, ok := info["小张"] //map chan 都可以两个返回
if !ok {
	fmt.Println("查无此人")
} else {
	fmt.Println(value)  //5
}
```

## 5. 遍历map

-   遍历`key`，`value`

```go
// 接上面代码
for k, v := range info { 
		fmt.Println(k, v)
	}
```

-   只遍历`key`

```go
for k := range info {
		fmt.Println(k)
	}
```

-   只遍历`value`

```go
for _, v := range info {
		fmt.Println(v)
	}
```

## 6. 删除map元素

使用`delete()`内建函数从map中删除一组键值对，`delete()`函数的格式如下：

```go
delete(map, key)
```

其中，

-   map:表示要删除键值对的map
-   key:表示要删除的键值对的键

```go
delete(info, "小明")
fmt.Println(info)
```


## 7. map类型的切片

它本质上是个切片，只是切片元素类型是map类型，就好像Py里的列表，列表元素是字典一样的。写到这里感觉还有点别扭的，没有py里面那么直观了当和方便，不过既然选择了Go，就要接受它的这种写法，熟练了就好了。

`out`

```go
var s1 = make([]map[int]string, 10, 10)
s1[0] = make(map[int]string, 1) //初始化
s1[0][0] = "bowen001"
fmt.Println(s1)
```

```go
[map[0:bowen001] map[] map[] map[] map[] map[] map[] map[] map[] map[]]
```

## 8. value为切片的map

它本质上是map，只是这个map的value类型是切片，类似于Py的字典的v值是列表类型一样。

```go
var m1 = make(map[string][]int, 10)
m1["南京"] = []int{1, 2, 3} //初始化
fmt.Println(m1)
```

`out`

```go
map[南京:[1 2 3]]
```

map类型在实际编程中用的还是很多的，尤其是遇到Json这种格式的。但从使用上来说没有Py那么灵活了。后面用到了再体会吧, 有个专门的第三方库可以很好的处理Json格式的数据。


## 9. 对map按照key排序
```go
func TestSortTest(t *testing.T) {  
   rand.Seed(time.Now().UnixNano())  
  
   var scoremap = make(map[string]int, 100)  
   for i := 0; i < 100; i++ {  
      key := fmt.Sprintf("stu%003d", i)  
      value := rand.Intn(100)  
      scoremap[key] = value  
   }  
   //fmt.Println(scoremap)  
  
   //for k, v := range scoremap {   // fmt.Println(k, v)   //}  
   var keys = make([]string, 0, 200)  
   for key := range scoremap {  
      keys = append(keys, key)  
   }  
   sort.Strings(keys)  
   for _, key := range keys {  
      fmt.Println(key, scoremap[key])  
   }  
   fmt.Println(scoremap)  
}
```

map是无序的，有时候我想要对map进行排序，且按照key排序，上面的思路是把map的key取出来，然后对key进行排序，排序后再根据key去遍历原来的scoremap，就可以有序输出了。

{{< admonition type=note title="注意点"  >}}
在 Go 中，使用 `for range` 循环迭代数组、切片、字符串和映射时是有序的。这意味着在迭代期间，它们的元素将按顺序访问。但需要注意的是，在并发情况下，无法保证多个协程同时访问相同的数据结构的顺序。

以下是一个简单的示例代码，演示了在 Go 中使用 `for range` 循环迭代一个数组的顺序：
```go
package main

import "fmt"

func main() {
    arr := []int{1, 2, 3, 4, 5}
    for i, v := range arr {
        fmt.Printf("index: %d, value: %d\n", i, v)
    }
}

```

>但是，for range循环遍历map却是无序的！即便是在上面的示例中，我按照有序的方式把数据存进了map中，但for range取出的时候却是随机的。

 {{< /admonition >}}