---
title: "Go刷算法"
date: 2023-04-28
tags : [                                    
     "算法",
 ]
categories : [                              
     "Go语言",
 ]
---
>💡 这是牛客网华为的机试题，大都是关于字符串的简单题，刷一刷还是有助于提高自己能力的。

## 1. 字符反转

{{< admonition type=note title="字符反转"  >}}
[字符串反转试题链接](https://www.nowcoder.com/practice/e45e078701ab4e4cb49393ae30f1bb04?tpId=37&tqId=21235&rp=1&ru=/exam/oj/ta&qru=/exam/oj/ta&sourceUrl=%2Fexam%2Foj%2Fta%3FtpId%3D37&difficulty=undefined&judgeStatus=undefined&tags=&title=)

接受一个只包含小写字母的字符串，然后输出该字符串反转后的字符串。（字符串长度不超过1000）

**输入描述**：输入一行，为一个只包含小写字母的字符串。

**输出描述**：输出该字符串反转后的字符串。

**示例1**  输入：abcd   输出：dcba
 
 
{{< /admonition >}}


{{< admonition type=example title="代码示例1"  open=false  >}}
```go
func main() {
    var s string
    fmt.Scanln(&s)
    for i:=len(s)-1;i>=0;i--{
        fmt.Print(string(s[i]))
    }
}
```

这是算是简单题了，但是有不少**注意点**：
- 用Scanln获取输入，这里由于是单字符串，所以Scan()方法也可，注意参数传的是&s
- 注意for循环是倒序的, 条件是i>=0, 需要细心一点
- 注意输出的时候**不要换行**，
- 同时要**用string()方法转一下**

 {{< /admonition >}}

{{< admonition type=example title="代码示例2"   open=false >}}
```go
func main() {
    scanner := bufio.NewScanner(os.Stdin)
    scanner.Scan()
    input := scanner.Text()
    for i:=len(input)-1;i>=0;i--{
        fmt.Print(string(input[i]))
    }
}
```

> 这段代码与上面的就是获取输入的方式不同，这两种获取输入的方式都要掌握！**为了统一，后面尽量用bufio里的scanner做题。**

{{< /admonition >}}

{{< admonition type=example title="python实现"   open=false >}}
```python
print(input()[::-1])
```

> 一句话的事情，太简洁了！

{{< /admonition >}}

## 2. 句子逆序

{{< admonition type=note title="句子逆序"  >}}
[句子逆序试题链接](https://www.nowcoder.com/practice/48b3cb4e3c694d9da5526e6255bb73c3?tpId=37&tqId=21235&rp=1&ru=%2Fexam%2Foj%2Fta&qru=%2Fexam%2Foj%2Fta&sourceUrl=%2Fexam%2Foj%2Fta%3FtpId%3D37&difficulty=undefined&judgeStatus=undefined&tags=&title=)

将一个英文语句以单词为单位逆序排放。例如“I am a boy”，逆序排放后为“boy a am I”

所有单词之间用一个空格隔开，语句中除了英文字母外，不再包含其他字符

数据范围：输入的字符串长度满足  1≤n≤1000   

**注意本题有多组输入**

**输入描述**：输入一个英文语句，每个单词用空格隔开。保证输入只包含空格和字母。

**输出描述**：得到逆序的句子

**示例1**   输入：I am a boy  输出：boy a am I

**示例2**  输入：nowcoder     输出：nowcoder

 {{< /admonition >}}

{{< admonition type=example title="代码示例"  open=false  >}}
```go
func main() {
    scanner := bufio.NewScanner(os.Stdin)
    scanner.Scan()
    input := scanner.Text()
    wordlist := strings.Fields(input)
    for i := len(wordlist)-1; i >= 0; i-- {
        fmt.Printf("%s ", wordlist[i])
    }
}
```

这是上上面类似，也有不少**注意点**：
- 前面三行是标准的获取输入，这个没什么好说的，记下即可。
- 利用strings.Fields转成切片了。i love you ->[i love you]
- 注意for循环是倒序的，-1不要忘记了！**条件是i>=0**
- 注意输出的时候**不要换行**，, 不要弄错了。
- 这里输出的时候用了printf输出的，就不需要去转了。如果是print输出也是可以的。
```go
fmt.Print( string(wordlist[i])+" ")
```
这样测试也能通过。
 {{< /admonition >}}


## 3. 字符集合

{{< admonition type=note title="字符集合"  >}}
[字符集合试题链接](https://www.nowcoder.com/questionTerminal/784efd40ed8e465a84821c8f3970b7b5)

输入一个字符串，求出该字符串包含的字符集合，按照字母输入的顺序输出。

数据范围：输入的字符串长度满足 1≤n≤100，且只包含大小写字母，区分大小写。  

**注意本题有多组输入**

**输入描述**：每组数据输入一个字符串，字符串最大长度为100，且只包含字母，不可能为空串，区分大小写。

**输出描述**：每组数据一行，按字符串原有的字符顺序，输出字符集合，即重复出现并靠后的字母不输出。

**示例1**   输入：abcqweracb  输出：abcqwer

**示例2**  输入：aaa     输出：a

 {{< /admonition >}}

{{< admonition type=example title="代码示例1"  open=false  >}}
```go
func main() {
    for {
        var s string
        _, err := fmt.Scanln(&s)
        if err != nil {
            break
        }
        //核心
        chars := make(map[rune]bool)
        for _, ch := range s {
            if !chars[ch] {  // 没有
                chars[ch] = true
                fmt.Printf("%c", ch)
            }
        }
        fmt.Println()
    }
}
```

这是有点复杂，不单单是逻辑上要注意，还有它是包含换行的，给的示例中是没有的，但是人家测试数据是有这样的数据。所以这里用了无限循环一行一行去读的。
- 核心逻辑用的是一个make(map[rune]bool)，这样的map，如果这个key对应的value是false（默认值），说明没有出现过，就将它的value设置胃true. 下一次遇到这个value是true，说明已经有了，不会走到if语句里。
- 需要注意的是，这里是**存在多行输入**，用了err判断，如果不为nil，就说明可能读完了，就退出循环。这是我当时没注意到的点。
- 无限循环，其实是每一行一个循环，操作完打印换行。**换行也是一个注意点**。

 {{< /admonition >}}

{{< admonition type=example title="代码示例2"  open=false  >}}
下面和上面类似，只是我换了一个输入方式，也通过测试了。
```go
func main() {
    scanner := bufio.NewScanner(os.Stdin)
    for {
        isStop := scanner.Scan()
        if !isStop {
            break
        }
        input := scanner.Text()
        chars := make(map[rune]bool)
        for _, ch := range input {
            if !chars[ch] {
                chars[ch] = true
                fmt.Printf("%c", ch)
            }
        }
        fmt.Println()
    }
}
```

 {{< /admonition >}}

{{< admonition type=example title="python实现"  open=false  >}}
```python
str1= input()
str2= input()
for i in str2:
    str1=str1.replace(i,"")
print(str1)
```
{{< /admonition >}}

## 4. 统计大写字母个数

{{< admonition type=note title="统计大写字母个数"  >}}
[句子逆序试题链接](https://www.nowcoder.com/practice/434414efe5ea48e5b06ebf2b35434a9c?tpId=37&rp=1&ru=%2Fexam%2Foj%2Fta&qru=%2Fexam%2Foj%2Fta&sourceUrl=%2Fexam%2Foj%2Fta%3FtpId%3D37&difficulty=&judgeStatus=&tags=&title=%E5%A4%A7%E5%86%99&gioEnter=menu)

找出给定字符串中大写字符(即'A'-'Z')的个数。数据范围：字符串长度：1≤∣s∣≤250  ，字符串中可能包含空格或其他字符。
进阶：时间复杂度：O(n)  ，空间复杂度：O(n) 

输入描述：对于每组样例，输入一行，代表待统计的字符串
 
输出描述：输出一个整数，代表字符串中大写字母的个数

{{< /admonition >}}

{{< admonition type=example title="代码示例"    >}}
```go
func main() {
    scanner := bufio.NewScanner(os.Stdin)
    scanner.Scan()
    input:=scanner.Text()
    res := 0
    for _, s := range input {
        if string(s) >= "A" && string(s)<= "Z" {
            res++
        }
    }
    fmt.Print(res)
}
```

这题算是简单题了，没有上一题那么坑，上一题主要包含了多行输入，多了一个无限循环。大致的思路就是
- 利用**bufio.NewScanner**获取输入，做完前面几道，这个应该很熟悉了吧！
- 循环遍历，注意用的是**for range循环**
- if判断的时候注意转一下格式
{{< /admonition >}}



## 5. 删除公共字符
{{< admonition type=note title="需求描述"  >}}
[试题链接](https://www.nowcoder.com/practice/f0db4c36573d459cae44ac90b90c6212?tpId=182&tqId=34789&ru=/exam/oj)

输入两个字符串，从第一字符串中删除第二个字符串中所有的字符。例如，输入”They are students.”和”aeiou”，则删除之后的第一个字符串变成”Thy r stdnts.”
输入描述：每个测试输入包含2个字符串
 
输出描述：输出删除后的字符串

示例1  输入：
They are students. 
aeiou
输出：Thy r stdnts.
{{< /admonition >}}

{{< admonition type=example title="代码示例"    >}}
```go
func main() {
    reader := bufio.NewReader(os.Stdin)
    str1, _ := reader.ReadString('\n')
    str2, _ := reader.ReadString('\n')

    // 去除字符串中的空格和换行符
    str1 = strings.TrimSpace(str1)
    str2 = strings.TrimSpace(str2)

    // 将str2中出现的字符从str1中删除
    for _, c := range str2 {
        str1 = strings.ReplaceAll(str1, string(c), "")
    }
    fmt.Println(str1)
 }
```

这题获取输入的方式又变了，很灵活吧！因为这题明确了两个字符串。
- 利用**bufio.NewReader**获取输入，这个要记下来。
- 去掉字符串左右两边的空格，注意这个方法只去两边的。
- 循环遍历str2，注意用的是**for range循环**，然后使用ReplaceAll方法，用空格替换掉。
{{< /admonition >}}

## 6. 替换字符

{{< admonition type=note title="需求描述"  >}}
[试题链接ACM模式](https://www.nowcoder.com/questionTerminal/c010f5a2cfad400183982dee1550815f)              [试题链接(核心模式)](https://www.nowcoder.com/practice/0e26e5551f2b489b9f58bc83aa4b6c68?tpId=13&rp=1&ru=%2Fexam%2Foj%2Fta&qru=%2Fexam%2Foj%2Fta&sourceUrl=%2Fexam%2Foj%2Fta%3FtpId%3D37&difficulty=&judgeStatus=&tags=&title=%E7%A9%BA%E6%A0%BC&gioEnter=menu)

请实现一个函数，将一个字符串s中的每个空格替换成“%20”。
例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。

输入："We Are Happy"   返回值："We%20Are%20Happy"
输入：" "                         返回值： "%20"
{{< /admonition >}}

{{< admonition type=example title="ACM模式代码示例"    >}}
```go
func main() {
    reader := bufio.NewReader(os.Stdin)
    var res string
    s, _ := reader.ReadString('\n')
    for i := 0; i < len(s); i++ {
        if s[i] == ' ' {
            res += "%100"
        } else {
            res += string(s[i])
        }
    }
    fmt.Print(res)  
}
```

这题沿用了上一题的输入方式，还是比较简单的。
- 利用**bufio.NewReader**获取输入，这个要记下来；
- 核心逻辑，依然是遍历，是就替换，不是就拼接；
- 拼接的时候用的是res，不是s。也要用string()转一下的
{{< /admonition >}}

{{< admonition type=example title="核心代码示例"    >}}
```go
/**
 * 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
 * @param s string字符串
 * @return string字符串
 */
func replaceSpace(s string) string {
    var res string
    for i := 0; i < len(s); i++ {
        if s[i] == ' ' {
            res += "%20"
        } else {
            res += string(s[i])
        }
    }
    return res
}
```

>  这题是核心模式的代码，是不需要写main函数的，人家上面也写的很清楚了。与ACM模式需要自己写输入输出的代码不同，核心模式只需要按照规定写返回值即可，**核心模式更简单**，因为输入输出人家都帮我们规定好了。只要记住核心实现逻辑，去写就可以了。不要去纠结这些东西。

{{< /admonition >}}

## 7. 大数加法

{{< admonition type=note title="需求描述"  >}}
[试题链接ACM模式](https://www.nowcoder.com/questionTerminal/1ac1af77536b4917aedaac4746eeb808)              [试题链接(核心模式)](https://www.nowcoder.com/practice/11ae12e8c6fe48f883cad618c2e81475?tpId=196&tqId=37176&ru=/exam/oj)

以字符串的形式读入两个数字，再以字符串的形式输出两个数字的和。

输入：输入两行，表示两个数字a和b，-109 <= a , b <= 109  ，用双引号括起。
输出：输出a+b的值，用双引号括起。
{{< /admonition >}}

{{< admonition type=example title="ACM模式"    >}}
```python
import sys
m = int(input()[1:-1])
n = int(input()[1:-1])
print('"'+str(m+n)+'"')
```

> 这题用python三行代码搞定了。我看了其他语言，一大堆我看都不想看。pythony依旧是如此强大！

{{< /admonition >}}

{{< admonition type=example title="核心模式"    >}}
```python
# 代码中的类名、方法名、参数名已经指定，请勿修改，直接返回方法规定的值即可
# 计算两个数之和
# @param s string字符串 表示第一个整数
# @param t string字符串 表示第二个整数
# @return string字符串
class Solution:
    def solve(self , s: str, t: str) -> int:
        # write code here
        a=int(s)
        b=int(t)
        c=a+b
        return c
```

>  这是核心模式代码，也比较简单。用python实现，后面有时间用Go试试吧。

{{< /admonition >}}


# 8. 字符串替换


# 机考小结



