---
title: "make和make install"                         
author: "博文"   
date: 2023-06-19        
comments: true  
tags : [                                    
     "Linux基础",
 ]
categories : [                              
     "Linux云计算",
 ]
---
简单来说，make 是编译，make install 是安装。
总结：linux编译安装中configure、make和make install各自的作用
- ./configure是用来检测你的安装平台的目标特征的。比如它会检测你是不是有CC或GCC，并不是需要CC或GCC，它是个shell脚本。
    
- make是用来编译的，它从Makefile中读取指令，然后编译。
    
- make install是用来安装的，它也从Makefile中读取指令，安装到指定的位置。

## 1. configure
---
这一步一般用来生成 Makefile，为下一步的编译做准备，你可以通过在 configure 后加上参数来对安装进行控制，比如代码:`./configure --prefix=/usr`上面的意思是将该软件安装在 /usr 下面，执行文件就会安装在 /usr/bin.同时一些软件的配置文件你可以通过指定 --sys-config= 参数进行设定。有一些软件还可以加上 --with、--enable、--without、--disable 等等参数对编译加以控制，你可以通过允许 ./configure --help 察看详细的说明帮助。
> 注意root权限的问题。如果用户是普通用户，则要指定目录，不要用默认目录。会报权限的错误。 [ViennaRNA安装](https://github.com/ViennaRNA/ViennaRNA)  这是我在安装这个的时候遇到的。  

## 2. make
---
这一步就是编译，大多数的源代码包都经过这一步进行编译（当然有些perl或Python编写的软件需要调用perl或python来进行编译）。如果在 make 过程中出现 error ，你就要记下错误代码（注意不仅仅是最后一行），然后你可以向开发者提交 bugreport（一般在 INSTALL 里有提交地址），或者你的系统少了一些依赖库等，这些需要自己仔细研究错误代码。make 的作用是开始进行源代码编译，以及一些功能的提供，这些功能由他的<font color="#ffc000"> Makefile </font>设置文件提供相关的功能，比如 make install 一般表示进行安装，make uninstall 是卸载，不加参数就是默认的进行源代码编译。  
make 是 Linux 开发套件里面自动化编译的一个控制程序，他通过借助 Makefile 里面编写的编译规范进行自动化的调用 gcc 、ld 以及运行某些需要的程序进行编译的程序。一般情况下，他所使用的 Makefile 控制代码，由 configure 这个设置脚本根据给定的参数和系统环境生成。

## 3. make install
---
这条命令来进行安装（当然有些软件需要先运行 make check 或 make test来进行一些测试），这一步一般需要你有 **root 权限**（因为要向系统写入文件）
