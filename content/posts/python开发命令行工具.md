---
title: "Python开发命令行工具"
subtitle: ""
date: 2023-02-01T09:45:03+08:00
author: "feyncode"
authorLink: "https://github.com/feyncode"
authorEmail: "feyncode@gmail.com"
description: ""
#keywords: ""
#password:
message: 真正的大师永远都怀着一颗学徒的心

weight: 0

tags:
- openstack,python
categories:
- python

# 如果设为true，这篇文章将不会显示在主页上
hiddenFromHomePage: false
#如果设为true，这篇文章将不会显示在搜索结果钟
hiddenFromSearch: false

resources:
- name: featured-image
  src: featured-image.jpg
- name: featured-image-preview
  src: featured-image-preview.jpg

toc:
  enable: true
math:
  enable: false
lightgallery: false

repost:
  enable: false
  url: ""

license: "<a rel=\"license external nofollow noopener noreferrer\" href=\"https://creativecommons.org/licenses/by-nc/4.0/\" target=\"_blank\">CC BY-NC 4.0</a>"
comment: true
---

<!--more-->

### 介绍

Python模块argparse，这是一个命令行选项，参数和子命令的解释器，使用该模块可以编写友好的命令行工具，在程序中定义好需要的参数，argparse将弄清楚如何解析 sys.argv中的参数。argparse模块还支持自动生成帮助和用法信息，当模块解析到无效参数时，还可以发出错误。

### python标准库sys模块

```
sys.agv				#命令行参数List,第一个元素时程序本身路径
sys.exit(n)		#退出程序，正常退出时exit(0)
sys.version 	#获取Python解释程序的版本信息
sys.maxint 		#最大的Int值
sys.path			#返回模块的搜索路径，初始化时使用PYTHONPATH环境变量的值
sys.stdin			#输入线管
sys.stdout		#输出相关
sys.stderror	#错误相关
```

### 命令行工具

我们使用过很多的命令行工具，那么python开发出的命令行工具，能列举出哪些呢？
其实python本身就是一个命令行工具，在cmd中使用python --help，能输出python的帮助语句，这就是命令行工具解析参数后输出的一个例子。
再有，pip 也是一个典型的例子，在使用pip时，不同的子命令可以达成不同的效果。
当然知道和做到相距实在太过遥远，我也很难写出一个python或pip。

### 概念

argparse是一个较大的模块，提供了很多功能，它的文档相当详细和完整，包含大量示例。所以学习该模块最好的教程就是官方文档，argparse([https://docs.python.org/zh-cn/3/library/argparse.html](https://docs.python.org/zh-cn/3/library/argparse.html)),那么为什么我没去看呢，一方面是我并不需要复杂的功能，另一方面我的英语阅读能力不足以让我完全理解该文档。我们必须承认，英语水平分开了普通的技术人员，因此我正在努力提升英语水平。
在开发一个命令行工具前，我们首先需要知道一个命令行工具包含了哪些内容，它又是怎么识别我们提供的参数的，识别后又是怎么提供输出的，对待错误的选项，它又是如何调整的。
因此，官网使用了 'ls' 这个命令来介绍命令行工具的几个概念：

- **ls** 是一个即使在运行的时候没有提供任何选项，也非常有用的命令。在默认情况下他会输出当前文件夹包括的内容。
- 如果我们想要使用比它默认提供的更多功能，我们需要告诉该命令更多信息。我们可以指定所谓的位置参数，之所以这样命名，是因为程序应该如何处理该参数值，完全取决于它在命令行出现的文职。更能体现这个概念的命令如 cp ，它最基本的用法是 `cp SRC DEST`，第一个位置参数指的是你想要复制的，第二个位置参数指的是你想要复制到的位置。
- 现在假设我们想要改变这个程序的行为。使用 **ls -l ，**我们可以输出更多的信息，在这个例子中，-l 被称为可选参数。
- **\--help**通常被用来输出帮助文档，它是非常有用的，因为当你遇到一个你从未使用过的程序时，你可以通过阅读它的帮助文档来弄清楚它是如何运行的。

了解了这些，我们通过几个例子可以更好的说明这个模块是如何解析从命令行获取的参数的，并且解析出的参数又是如何被程序使用的。

### 基础

命令行参数解析很容易，但是它也可以被加入很多参数，注入很多定义，这让最后的程序变得不堪入目，为了防止我的朋友，也就是你，在开始就头痛，我首先带来一个我认为最简单的案例。

### 示例

所有解析都应当配有示例，并展示对比来说明程序的行为有哪些改变。

> argparse使用主要有四个步骤：
>
> - 导入argparse包
> - 创建 ArgumentParser() 参数对象
> - 调用 add_argument() 方法往参数对象中添加参数
> - 使用 parse_args() 解析添加参数的参数对象，获得解析对象
> - 程序其他部分，当需要使用命令行参数时，使用解析对象.参数获取

在未使用argparse模块前，我编写了这样一个python文件。

```
print("hello world!")
```

我们很熟悉这个案例，毫无疑问我们清楚python运行该文件时会得到什么样的输出，当我们在终端运行这个python文件demo1时，我们会使用以下命令。

```
$ python demo1.py
hello world!
```

是的，我们看到在命令行中，运行该文件输出了我们想要的结果，一般情况下，我们需要python文件输出某个值时，要么我们将该值在文件中写死，就像上面这个例子一样，我们已经在文件中将想要的输出写死在文件中，这个值在未被输出时就已经在程序中被决定。如果我们需要用户在终端中向程序提供某些值，来改变程序的行为，得到不同的输出，我们常用的手段是使用input，让程序读入终端的输入来达到这样的效果。不过这样做仍然需要手动的输入，如果是自动化程序，那么可以使用参数直接带入需要的变量将大大减少这样的交互。
接下来我将演示如何使用argparse来完成一个简单的参数解析。

```
import argparse #导入argparse包

parse = argparse.ArgumentParser() #创建参数对象
parse.add_argument('hello') #向参数对象中添加参数
args = parse.parse_args() #解析参数对象获得解析对象
if __name__ == '__main__':
   print(args.hello)
```

让我们来运行一下这个程序

```
$ python demo1.py helloworld
helloworld
```

很简单，这是一个容易的例子，需要注意的是，由于没有指定，所以命令行参数输入默认按顺序复制，顺序不同的话得到的结果将不同。
接下来我将带入一些官网上的案例，来继续解释argparse的一些进阶示例。

### 使用介绍

- --help 选项，也可缩写为 -h，是唯一一个可以直接使用的选项（即不需要指定该选项的内容）。指定任何内容都会导致错误。即便如此，我们也能直接得到一条有用的用法信息。

add_argument()方法，定义如何解析命令行参数，对于该方法，参数解释如下:

- name or flags - 选项字符串的名字或者列表，例如 foo 或者 -f, --foo。
- action - 命令行遇到参数时的动作，默认值是 store。
- - store_const，表示赋值为const；
- – append，将遇到的值存储成列表，也就是如果参数重复则会保存多个值;
- – append_const，将参数规范中定义的一个值保存到一个列表；
- – count，存储遇到的次数；此外，也可以继承 argparse.Action 自定义参数解析；
- nargs - 应该读取的命令行参数个数，可以是
- 具体的数字，或者是?号，当不指定值时对于 Positional argument 使用 default，对于 Optional argument 使用 const
- 或者是 * 号，表示 0 或多个参数；
- 或者是 + 号表示 1 或多个参数。
- const - action 和 nargs 所需要的常量值。
- default - 不指定参数时的默认值。
- type - 命令行参数应该被转换成的类型。
- choices - 参数可允许的值的一个容器。
- required - 可选参数是否可以省略 (仅针对可选参数)。
- help - 参数的帮助信息，当指定为 argparse.SUPPRESS 时表示不显示该参数的帮助信息.
- metavar - 在 usage 说明中的参数名称，对于必选参数默认就是参数名称，对于可选参数默认是全大写的参数名称.
- dest - 解析后的参数名称，默认情况下，对于可选参数选取最长的名称，中划线转换为下划线.

### 可选参数设置

通过在参数名前加 `--`, 设置为可选参数，如果未输入，则使用`default`默认值，若为设置`default`则默认赋值 `None`

```
parse.add_argument('-n', '--name', type=str, metavar='name', help='New flavor name')
```

引用名，可以缩短参数名，简化命令行参数输入；也就是使用-n或者-name可以获得一样的效果

### 必选参数设置

需要一个必选参数时，则设置`required=True`,这时，无论参数是否时可选参数，都必须输入

### 列表参数传入设置

添加`narg='+'` 这样可以在命令行中添加若干个参数，传入后将被添加到列表中。

### 互斥参数

也就是`add_mutually_exclusive_group`方法，这让我们可以添加两个互相排斥的参数，也就是只能选择其中一个参数添加

### 默认参数设置

> set_defaults()可以设置一些参数的默认值

