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
>💡 

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

> 这段代码其实就是获取输入的方式不同，这两种获取输入的方式都要掌握！

{{< /admonition >}}

## 2. 句子逆序

{{< admonition type=note title="句子逆序"  >}}
[句子逆序试题链接](https://www.nowcoder.com/practice/48b3cb4e3c694d9da5526e6255bb73c3?tpId=37&tqId=21235&rp=1&ru=%2Fexam%2Foj%2Fta&qru=%2Fexam%2Foj%2Fta&sourceUrl=%2Fexam%2Foj%2Fta%3FtpId%3D37&difficulty=undefined&judgeStatus=undefined&tags=&title=)
将一个英文语句以单词为单位逆序排放。例如“I am a boy”，逆序排放后为“boy a am I”

所有单词之间用一个空格隔开，语句中除了英文字母外，不再包含其他字符

数据范围：输入的字符串长度满足 1≤�≤1000 1≤n≤1000   

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

这是算是简单题了，但是有不少**注意点**：
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

{{< admonition type=note title="句子逆序"  >}}
[字符集合试题链接](https://www.nowcoder.com/questionTerminal/784efd40ed8e465a84821c8f3970b7b5)

输入一个字符串，求出该字符串包含的字符集合，按照字母输入的顺序输出。

数据范围：输入的字符串长度满足 1≤n≤100 1≤n≤100  ，且只包含大小写字母，区分大小写。  

**注意本题有多组输入**

**输入描述**：每组数据输入一个字符串，字符串最大长度为100，且只包含字母，不可能为空串，区分大小写。

**输出描述**：每组数据一行，按字符串原有的字符顺序，输出字符集合，即重复出现并靠后的字母不输出。

**示例1**   输入：abcqweracb  输出：abcqwer

**示例2**  输入：aaa     输出：a

 {{< /admonition >}}

{{< admonition type=example title="代码示例"  open=false  >}}
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

