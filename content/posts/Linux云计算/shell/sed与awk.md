---
title: "sed与awk"                         
author: "博文"   
date: 2023-04-19          
comments: true 
 
tags : [                                    
     "Linux基础",
 ]
categories : [                              
     "Linux云计算",
 ]
---

`awk` 和 `sed` 都是Unix/Linux环境下常用的文本处理工具，它们的主要作用是对文本进行处理和转换。

使用 `sed`
--------

`sed` 是一个流编辑器，它能够对输入的文本进行一些编辑和替换操作。以下是一些常见的使用方法：

### 替换文本

{{< admonition type=example  title="sed替换文本示例"  >}} 
```bash
sed 's/old_text/new_text/g' file.txt
```

这个命令会将文件 `file.txt` 中的所有 `old_text` 替换成 `new_text`。

其中 `s` 表示替换操作，`g` 表示全局匹配（即替换所有匹配项）。如果不加 `g` 参数，`sed` 只会替换每行中的第一个匹配项。

> 这个操作还是很常用的，需要记住。

{{< /admonition >}}

### 删除行

{{< admonition type=example  title="sed删除行"  >}} 

```bash
sed '/pattern/d' file.txt
```

这个命令会删除文件 `file.txt` 中所有匹配 `pattern` 的行。

{{< /admonition >}}

### 添加文本
{{< admonition type=example  title="sed添加文本"  >}} 

```bash
sed '1s/^/new_text\n/' file.txt
```

这个命令会在文件 `file.txt` 的第一行前添加 `new_text`。

其中 `1s` 表示对第一行进行操作，`^` 表示行首，`\n` 表示换行符。
{{< /admonition >}}

使用 `awk`
--------

`awk` 是一种数据处理工具，它可以对数据进行格式化和转换。以下是一些常见的使用方法：

### 格式化输出
{{< admonition type=tip  title="awk格式化输出"  >}} 

```bash
awk '{printf "%-10s %-10s %-10s\n", $1, $2, $3}' file.txt
```

这个命令会将文件 `file.txt` 中的每一行按照指定的格式输出。

其中 `%-10s` 表示左对齐的字符串格式，宽度为10个字符。`$1`、`$2`、`$3` 表示第1、2、3列的数据。
{{< /admonition >}}

### 计算总和

{{< admonition type=tip  title="awk计算总和"  >}} 
```bash
awk '{sum+=$1} END {print sum}' file.txt
```

这个命令会计算文件 `file.txt` 中第一列的总和，并将结果输出。

其中 `sum+=$1` 表示每次将第一列的值加到变量 `sum` 中，`END` 表示在文件处理结束时执行 `print sum` 命令。
{{< /admonition >}}

### 过滤数据

{{< admonition type=tip  title="awk过滤数据"  >}} 
```bash
awk '$1 > 10 {print $0}' file.txt
```

这个命令会输出文件 `file.txt` 中第一列大于 10 的行。

其中 `$1` 表示第一列的值，`$0` 表示整行的值。如果表达式 `$1 > 10` 为真，则输出整行的值。
{{< /admonition >}}
