---
title: "Go语言学习之路"
date: 2023-04-01T11:55:58+08:00
weight: 2
tags : [                                    
     "Go基础",
 ]
categories : [                              
     "Go语言",
 ]
---
{{< admonition type=note title="Go语言学习之路"  >}}
 这篇文章是汇总了我Go语言学习笔记，目的主要有两个
 - **方便阅读**，博客比较多且零散，汇总分类到一起读起来方便。
 - **成体系**，汇总到一起可以看到自己在这条路上已经走了多远。
> 路遥知马力，坚持学习，写作，输出，在这门语言上深入下去，沿着go这条学习路线去走，沿途会发现很多以前没看到的花花草草，看到不一样的世界。

{{< /admonition >}}

 ## Go基础语法

{{< admonition type=note title="开篇"  >}}
[Go语言的前世今生](https://sunnydongbowen.github.io/go%E8%AF%AD%E8%A8%80%E7%9A%84%E5%89%8D%E4%B8%96%E4%BB%8A%E7%94%9F/)（`重要指数🌟🌟🌟 `  `难度指数🌟🌟 ` `tony白专栏`）

[Go程序是怎么样的](https://sunnydongbowen.github.io/go%E7%A8%8B%E5%BA%8F%E6%98%AF%E6%80%8E%E4%B9%88%E6%A0%B7%E7%9A%84/)（`重要指数🌟🌟🌟 `  `难度指数🌟🌟 ` `tony白专栏`）

[Go的设计哲学](https://sunnydongbowen.github.io/go%E7%9A%84%E8%AE%BE%E8%AE%A1%E5%93%B2%E5%AD%A6/)（`重要指数🌟🌟🌟 `  `难度指数🌟🌟 ` `tony白专栏`）

[Go VS Python](https://sunnydongbowen.github.io/go-vs-python/)（`重要指数🌟🌟🌟 `  `难度指数🌟🌟 ` `self`）

[Go module初步](https://sunnydongbowen.github.io/go-module%E5%88%9D%E6%AD%A5/)（`重要指数🌟🌟 `  `难度指数🌟🌟 ` `tony白专栏`）

[Go module详细](https://sunnydongbowen.github.io/go-module-%E8%AF%A6%E7%BB%86/)（`重要指数🌟🌟🌟 `  `难度指数🌟🌟 `  `tony白专栏`）

[第一个go程序](https://sunnydongbowen.github.io/%E7%AC%AC%E4%B8%80%E4%B8%AAgo%E7%A8%8B%E5%BA%8F/)  （`重要指数🌟🌟🌟 `  `难度指数🌟🌟 `  `self`）

{{< /admonition >}}

{{< admonition type=example title="变量和数据类型"  >}}
[var](https://sunnydongbowen.github.io/var/)（`重要指数🌟🌟🌟 `  `难度指数🌟🌟 `  `tony白专栏，李文周博客及视频`）

[const](https://sunnydongbowen.github.io/const/)（`重要指数🌟🌟🌟 `  `难度指数🌟🌟 `  `tony白专栏，李文周博客及视频`）

[类型](https://sunnydongbowen.github.io/%E7%B1%BB%E5%9E%8B/)（`重要指数🌟🌟 `  `难度指数🌟🌟 `  `tony白专栏`）

[运算符](https://sunnydongbowen.github.io/%E8%BF%90%E7%AE%97%E7%AC%A6/)（`重要指数🌟🌟🌟 `  `难度指数🌟🌟 ` `李文周博客及视频`

[变量遮蔽和作用域](https://sunnydongbowen.github.io/%E5%8F%98%E9%87%8F%E9%81%AE%E8%94%BD%E5%92%8C%E4%BD%9C%E7%94%A8%E5%9F%9F/)（`重要指数🌟🌟🌟 `  `难度指数🌟🌟 `  `tony白专栏`）

[自定义类型与类型别名](https://sunnydongbowen.github.io/%E8%87%AA%E5%AE%9A%E4%B9%89%E7%B1%BB%E5%9E%8B%E4%B8%8E%E7%B1%BB%E5%9E%8B%E5%88%AB%E5%90%8D/)（`重要指数🌟🌟🌟 `  `难度指数🌟🌟 ` `李文周博客及视频`）

[slice基础篇](https://sunnydongbowen.github.io/slice%E5%9F%BA%E7%A1%80%E7%AF%87/)（`重要指数🌟🌟🌟 `  `难度指数🌟🌟 ` `李文周博客及视频`）

[map基础篇](https://sunnydongbowen.github.io/map%E5%9F%BA%E7%A1%80%E7%AF%87/)（`重要指数🌟🌟🌟🌟 `  `难度指数🌟🌟 ` `李文周博客及视频`）

[指针](https://sunnydongbowen.github.io/%E6%8C%87%E9%92%88/)（`重要指数🌟🌟🌟 `  `难度指数🌟🌟🌟  ` `李文周博客及视频，self`)

{{< /admonition >}}

{{< admonition type=note title="流程控制与函数"  >}}

[go代码的执行顺序](https://sunnydongbowen.github.io/go%E4%BB%A3%E7%A0%81%E7%9A%84%E6%89%A7%E8%A1%8C%E9%A1%BA%E5%BA%8F/)（`重要指数🌟🌟🌟 `  `难度指数🌟🌟 `  `tony白专栏`）

[if语法](https://sunnydongbowen.github.io/if%E8%AF%AD%E6%B3%95/)（`重要指数🌟🌟🌟 `  `难度指数🌟🌟 `  `tony白专栏`）

[for循环](https://sunnydongbowen.github.io/for%E5%BE%AA%E7%8E%AF/)（`重要指数🌟🌟🌟 `  `难度指数🌟🌟 `  `tony白专栏`）

[swtich-case](https://sunnydongbowen.github.io/swtich-case/)（`重要指数🌟🌟🌟 `  `难度指数🌟🌟 `  `tony白专栏`）

[函数基础篇](https://sunnydongbowen.github.io/%E5%87%BD%E6%95%B0%E5%9F%BA%E7%A1%80%E7%AF%87/)（`重要指数🌟🌟🌟🌟 `  `难度指数🌟🌟 ` `李文周博客及视频`）

[函数学习笔记(知乎)](https://sunnydongbowen.github.io/%E5%87%BD%E6%95%B0%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0%E7%9F%A5%E4%B9%8E/)（`重要指数🌟🌟🌟🌟 `  `难度指数🌟🌟 🌟` `知乎专栏`）

[方法基础篇](https://sunnydongbowen.github.io/%E6%96%B9%E6%B3%95%E5%9F%BA%E7%A1%80%E7%AF%87/)（`重要指数🌟🌟🌟🌟 `  `难度指数🌟🌟 ` `李文周博客及视频`）

[值recevier和指针recevier](https://sunnydongbowen.github.io/%E5%80%BCrecevier%E5%92%8C%E6%8C%87%E9%92%88%E5%9E%8Brecevier/)（`重要指数🌟🌟🌟🌟 `  `难度指数🌟🌟 🌟` `tony白专栏`）

[interface](https://sunnydongbowen.github.io/%E6%8E%A5%E5%8F%A3/)（`重要指数🌟🌟🌟 `  `难度指数🌟🌟 ` `李文周博客及视频`）

[panic](https://sunnydongbowen.github.io/panic/)（`重要指数🌟🌟🌟🌟 `  `难度指数🌟🌟 ` `tony白专栏`）

{{< /admonition >}}


{{< admonition type=tip  title="学习方法篇"  >}}
[学习方法篇(tony白专栏笔记)](https://sunnydongbowen.github.io/%E5%AD%A6%E4%B9%A0%E6%96%B9%E6%B3%95%E7%AF%87tony%E7%99%BD%E4%B8%93%E6%A0%8F%E7%AC%94%E8%AE%B0/)（`重要指数🌟🌟 `  `难度指数🌟 `  `tony白专栏`）

[学习资源(个人)](https://sunnydongbowen.github.io/%E5%AD%A6%E4%B9%A0%E8%B5%84%E6%BA%90%E4%B8%AA%E4%BA%BA/)（`重要指数🌟🌟 `  `难度指数🌟🌟 `  `self`）

[tony白专栏推荐的学习资源](https://sunnydongbowen.github.io/tony%E7%99%BD%E4%B8%93%E6%A0%8F%E6%8E%A8%E8%8D%90%E7%9A%84%E5%AD%A6%E4%B9%A0%E8%B5%84%E6%BA%90/)（`重要指数🌟🌟 🌟`  `难度指数🌟🌟 `  `tony白专栏`）

{{< /admonition >}}


##  Go语法进阶

[并发编程基础篇](https://sunnydongbowen.github.io/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E5%9F%BA%E7%A1%80%E7%AF%87/)（`重要指数🌟🌟🌟🌟 `  `难度指数🌟🌟🌟 ` `李文周博客及视频`）

[并发编程进阶篇](https://sunnydongbowen.github.io/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E8%BF%9B%E9%98%B6%E7%AF%87/)（`重要指数🌟🌟🌟🌟 `  `难度指数🌟🌟🌟 ` `大明课程`）

[Context基础篇](https://sunnydongbowen.github.io/context%E5%9F%BA%E7%A1%80%E7%AF%87/)（`重要指数🌟🌟🌟🌟 `  `难度指数🌟🌟🌟 ` `李文周博客及视频`）

[Context进阶篇](https://sunnydongbowen.github.io/context%E8%BF%9B%E9%98%B6%E7%AF%87/)（`重要指数🌟🌟🌟🌟 `  `难度指数🌟🌟🌟 ` `大明课程`）



## Go Web


[http](https://sunnydongbowen.github.io/http/) （`重要指数🌟🌟🌟 `  `难度指数🌟🌟🌟 ` `汪明Go并发编程实战笔记`）

[https](https://sunnydongbowen.github.io/https/)（`重要指数🌟🌟🌟 `  `难度指数🌟🌟🌟 ` `汪明Go并发编程实战笔记`）

[gin~示例及restfulAPI](https://sunnydongbowen.github.io/gin%E7%A4%BA%E4%BE%8B%E5%8F%8Arestfulapi/)（`重要指数🌟🌟🌟 `  `难度指数🌟🌟 ` `李文周博客及视频`）

[gin~路由](https://sunnydongbowen.github.io/gin%E8%B7%AF%E7%94%B1/)（`重要指数🌟🌟🌟 `  `难度指数🌟🌟 ` `李文周博客及视频`）

[gin~redirect](https://sunnydongbowen.github.io/gin%E7%9A%84redirect/) （`重要指数🌟🌟🌟 `  `难度指数🌟🌟 ` `李文周博客及视频`）

[gin~middleware](https://sunnydongbowen.github.io/gin%E7%9A%84middleware/) （`重要指数🌟🌟🌟 `  `难度指数🌟🌟 ` `李文周博客及视频`）

[gin~session](https://sunnydongbowen.github.io/gin~session/) （`重要指数🌟🌟🌟 `  `难度指数🌟🌟 ` `大明上课demo`）

[gin~文件上传](https://sunnydongbowen.github.io/gin%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0/) （`重要指数🌟🌟🌟 `  `难度指数🌟🌟 ` `李文周博客及视频`）

