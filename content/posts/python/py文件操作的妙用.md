---
title: "python文件操作的妙用"                         
author: "博文"   
date: 2023-05-29          
comments: true 
tags : [                                    
     "python基础",
 ]
categories : [                              
     "Python",
 ]
---

## 需求来源
---
这几天一直在研究obsidian，喜欢上了它的标签系统，很适合零散类的笔记
[[聊聊我折腾过的笔记软件#🆕互联网记录]]。我把之前写的日记逐步整理到了obsidian中来，我的日记一开始按照月份写的，一个月一个page。比较好整理。后来按照日期写的，一个月有30篇日记，而从notion导出来的日记也是单独的，并且文件名后面还带了文件的长长的id号。鉴于上面这两个问题，我的需求来了：
1. 希望去掉文件名带的id，让文件名简短点。一开始我是手动操作了一个月的日记，很繁琐，且浪费时间。
2. 希望能够插入文件标识，因为我利用obsidian的db folder和dataview自动生成了一个简单的database。这个database虽然没notion好，但是也够用了。我尝试手动一篇篇去操作，很繁琐！且不准确。

##  代码示例
---
{{< admonition type=example title="代码示例"  >}}
```python
import os  
str = ''  
  
foldler_path = r'C:\Users\sunnydongbowen\Desktop\mon007days'  
folder_path2 = r'C:\Users\sunnydongbowen\Desktop\mon002days'  
filenames = os.listdir(foldler_path)  
# print(filenames)  
  
newfile = []  
for file in filenames:  
    file = folder_path2 + '\ ' + file  
    file = file.replace(' ', '', 1)  
    file = file[:-36] + '.md'  
    newfile.append(file)  
print(newfile)  
  
for file, newfile in zip(filenames, newfile):  
    count = 0  
    ## 这个是为了拼接第一个file，是不能动的  
    file = foldler_path + '\ ' + file  
    file = file.replace(' ', '', 1)  
  
    # print(file)  
    ## 这个是为了拼接第二个newfile  
    pathf = os.path.dirname(newfile)  
    fileName = file[len(pathf) + 1:]  
    # print(fileName)  
    fileName = fileName[:-36]  
    print(fileName)  
  
    with open(file, encoding='utf-8') as f, open(newfile, 'w', encoding='utf-8') as f1:  
        for line in f:  
            if count == 0:  
                line = '---\n'  
            if count == 1:  
                line = '身体:  "[[{}#身体]]"\n'.format(fileName)  
            if count == 2:  
                line = '饮食:  "[[{}#生活]]"\n'.format(fileName)  
            if count == 3:  
                line = '学习: "[[{}#学习]]"\n'.format(fileName)  
            if count == 4:  
                line = '杂记:  "[[{}#杂想]]"\n'.format(fileName)  
            if count == 5:  
                line = '特殊的事情: " "\n'  
            if count == 6:  
                line = '---\n'  
            # 写文件  
            f1.write(line)  
            count = count + 1
```
{{< /admonition >}}

{{< admonition type=tip title="代码分析"  >}}
我一开始想的简单了，每个文件想插入标识，我直接文件读写直接写不就行了？后来发现这种方式前面的几行数据会被覆盖掉。下面是我刚尝试时的思路。不仅怎么都不行，还把obsidian弄的卡住了。原因是文件权限的问题！
```python
with open(r'D:\私密日记\日记\2022\mon006days\7 周二 1.md', 'r+',encoding='utf-8') as file:  
    # 将文件指针移动到第5个字符之后的位置  
    file.seek(0)  
    # 写入内容到指定位置  
    file.write('---\n')  
    file.write('身体: "[[004#身]]"\n')  
    file.write('学习: "[[004#学习]]\n')  
    file.write('杂记: "[[004#杂记]]"\n')  
    file.write('饮食: "[[004#生活]]"\n')  
    file.write('---\n')
```

但是我大体的思路是没错的。
1. 遍历这个月日记的目录，`filenames = os.listdir(foldler_path)  `得到的是一个列表。但这个列表只是文件名，而不是文件完整路径。而我去读文件内容肯定是需要全路径的，于是，拼接了一下。得到了第一个文件句柄的file
```python
file = foldler_path + '\ ' + file  
file = file.replace(' ', '', 1)   # 去掉第一个空格，因为我再拼接的时候有一个空格。
```
2. 那么，第二个newfile的路径怎么来呢？,另起一个文件，同样进行拼接，只是这次拼接要注意把自带的长长的id给去掉，利用切片截取。
```python
newfile = []  
for file in filenames:  
    file = folder_path2 + '\ ' + file  
    file = file.replace(' ', '', 1)  
    file = file[:-36] + '.md'  
    newfile.append(file)  
print(newfile)  
```
3. 这两个参数有了，于是，我把程序跑了起来。发现我插入的标识中带了文件id。很长。问题出现在这里，这是因为我用了file，而不是newfile这个参数，两个解决办法，第一个，重新切片截取一下就是了；第二个，file改成newfile即可。
```python
## 这个是为了拼接第二个newfile  
pathf = os.path.dirname(newfile)  
fileName = file[len(pathf) + 1:]  
# print(fileName)  
fileName = fileName[:-36]  
print(fileName)  
```

{{< /admonition >}}

## 运行结果
---
运行前:

![](/python/20230529153323.png)

运行后:

![](/python/20230529153518.png)

并且每个文件都被我写入了标识头，结合obsidian的插件，得到了我想要的database

![](/python/20230529153725.png)

## 新增需求1
除了上面**去掉download下来的id**，**插入文件标识外**。后面整理过程中又增加了几个需求。例如**修改文件名，日记全部按照年月日排名**，每一天都是独一无二的。而不是重复，不是去年今天的重复，也不是上个月今天的重复。我发现去年写的日记看起来多，事实上都是重复性内容。而且我搜索起来不能一眼看到具体是什么时候。这个时候就有了这个需求。实现效果如下。

![](/python/20230531113841.png)

这个需求是在上面两个基础上稍微做了一些修改就可以了。
```python
import  os  
  
foldler_path = r'C:\Users\sunnydongbowen\Desktop\mon007days'  
folder_path2 = r'C:\Users\sunnydongbowen\Desktop\M0062'  
filenames = os.listdir(foldler_path)  
print(filenames)  
  
newfilenames=[]  
for file in filenames:  
    file='2022-07-'+file  
    newfilenames.append(file)  
print(newfilenames)  
  
  
for file, newfile in zip(filenames, newfilenames):  
    count=0  
    file = foldler_path +"\ " +file  
    file = file.replace(' ', '', 1)  
    #print(file)  
  
    newfile = folder_path2 + "\ " + newfile  
    newfile = newfile.replace(' ', '', 1)  
    #print(newfile)  
  
    pathf = os.path.dirname(newfile)  
    fileName = newfile[len(pathf) + 1:]  
    print(fileName)  
    #fileName = fileName[:-36]  
    #print(fileName)  
    with open(file, encoding='utf-8') as f, open(newfile, 'w', encoding='utf-8') as f1:  
        for line in f:  
            if count == 0:  
                line = '---\n'  
            if count == 1:  
                line = '身体:  "[[{}#身体]]"\n'.format(fileName)  
            if count == 2:  
                line = '饮食:  "[[{}#生活]]"\n'.format(fileName)  
            if count == 3:  
                line = '学习: "[[{}#学习]]"\n'.format(fileName)  
            if count == 4:  
                line = '杂记:  "[[{}#杂想]]"\n'.format(fileName)  
            if count == 5:  
                line = '特殊的事情: " "\n'  
            if count == 6:  
                line = '---\n'  
            # 写文件  
            f1.write(line)  
            count = count + 1
```

## 新增需求2
我从notion下载了12月份的日记，发现它每个文件名是这样的：

![](/python/20230531175239.png)

而我理想中是这样的:

![](/python/20230531175359.png)

代码实现：
```python
# 12月日记处理  
  
import  os  
foldler_path = r'C:\Users\sunnydongbowen\Desktop\1222'  
folder_path2 = r'C:\Users\sunnydongbowen\Desktop\M0062'  
filenames = os.listdir(foldler_path)  
print(filenames)  
  
newfilenames=[]  
for filename in filenames:  
    filename=filename[:-36] + '.md'  
    filename=filename.replace(" ","-",2)  
    filename=foldler_path+"\ "+filename  
    filename = filename.replace(" ", "", 1)  
    newfilenames.append(filename)  
  
filenames_old=[]  
  
for filename in filenames:  
    filename = foldler_path + "\ " + filename  
    filename = filename.replace(" ", "", 1)  
    filenames_old.append(filename)  
  
print(filenames_old)  
  
for  filename,newfilename  in zip(filenames_old,newfilenames):  
    os.rename(filename,newfilename)
```

这么一小段代码，就帮我省去了很大的繁琐操作，不然我还要手动一点点修改。而且后面几个月的日记都是这样排列的，有了这个demo，就可以很快实现我想要的效果了。
这个需求的实现比上面的还简单一些。上面涉及到了读写文件，而这里只是修改了文件名。

## 编程随想
---
这是在我整理日记中无意间发现的需求，一开始我觉得手动整理不就得了。干嘛花这个时间去调试脚本。直到今天上午我用脚本整理7月份日记时，体会到了乐趣。全部自动化完成，快到起飞！我才抽出时间来做其他事情。不然下午也不会有时间写这篇blog。
1. 这次完成了小需求，下次遇到类似的，是不是就可以很快写出来了。别人如果有类似的需求，是不是也可借鉴一下？这个时间投入是值得的。就如同做自动化一样。
2. 当遇到批量但是又有规律的任务，重复性的任务。要想到用python实现自动化，这会帮我节省时间，减少重复性劳动。事实上，我之前在工作中[[py~聊聊Python在我工作中的应用]]  写的所有自动化脚本，基本上都有这几个特点:  **重复性高；有一定规律；数据量大**；遇到这几个特点时，要想到去利用python脚本来实现自动化，这个时间绝对是值得的。
3. python在自动化和数据分析方面十分强大，而且应用十分广泛。基本上能想到的需求都可以用它来做。
4. 这套整理日记中发现的小需求，也挺有意思的。有了基础代码，一步步修改迭代满足我的需求，实现我想要的效果。有点类似实际开发中的过程了。