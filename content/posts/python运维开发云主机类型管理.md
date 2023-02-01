---
title: "Python运维开发云主机类型管理"
subtitle: ""
date: 2023-02-01T09:48:04+08:00
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

### python开发云主机类型管理脚本

开发flavor_manager.py程序，来完成云主机类型管理的相关操作。

该文件拥有以下功能:

- 根据命令行参数，创建一个云主机类型，返回response。
- 查询admin账户下所有的云主机类型
- 查询给定具体名称的云主机类型
- 支持删除指定的id云主机类型

操作并不复杂，程序也相对较简单，大致分为两大部分

一，解析传入的参数，将参数转化为程序使用的变量

二，通过openstacksdk，使用获得的变量完成相应行为。

### 基础

首先构建我们的程序主题，肯定是需要argparse模块，那么我们先把argparse模块添加进我们的程序。

```python
import argparse

parse = argparse.ArgumentParser()
args = parse.parse_args()
```

这做不了什么，也不能满足题目的要求，所以我做出改变，既然我们需要让程序拥有四种功能，那肯定要写出四个相应的实现方法，而参数解析是必须的，那么我们需要这样一个操作，当我们首先解析出需要的操作时，就进行反馈，将继续的参数解析到相应的功能里，这样就把四种功能分开，整个程序会变得很整洁。

### 创建解析对象

再回来看看题目，四种功能使用位置参数来分别区分，既然如此，我们第一个参数就设置为 `option` ，用来识别程序需要完成的功能，之后将带入相应的解析中，并调取需要的程序来完成相应的行为。

整体思路就是这样，可能你还没跟上。那么让我来演示第一步的解析：

```python
import argparse

parse = argparse.ArgumentParser()
parse.add_argument('option')

if __name__ == '__main__':
    args = parse.parse_args()
    print(args.option)
```

可以预见，这一步后，程序中被添加进了一个必须的参数，也就是我们需要的位置参数，如果你输入 `create`，那么程序中的`args.option`就设置为`create`，诸如此类，**getall** ,**get**,**delete**。也就很好区分了，接下来我们需要一个判断，在不同的判断中，我们读入不同的参数。

### 解析位置参数

还是一步一步来吧，我们先以`create`为例：

```python
import argparse

parse = argparse.ArgumentParser()
parse.add_argument('option')
parse.add_argument('-n', '--name', type=str, metavar='name', help='New flavor name')
parse.add_argument('-m', '--ram', type=int, metavar='size-mb', default=256, help='Memory size in MB (DEFAULT 256M)')
parse.add_argument('-v', '--vcpus', type=int, metavar='vcpus', default=1, help='Number of vcpus (default 1)')
parse.add_argument('-d', '--disk', type=int, metavar='size-gb', default=0, help='Disk size in GB (default 0G)')
parse.add_argument('-id', '--id', type=str, metavar='id', default='auto', help='Unique flavor ID')

if __name__ == '__main__':
    args = parse.parse_args()
    if args.option == 'create':
        print("i will create new flavor")
```

 可以看到，我已经把create这个行为剥离开来，接下来想必你已经知道我要做什么了。让我们把四种功能补全看看。

### 解析可选参数

```python
import argparse

parse = argparse.ArgumentParser()
parse.add_argument('option')
parse.add_argument('-n', '--name', type=str, metavar='name', help='New flavor name')
parse.add_argument('-m', '--ram', type=int, metavar='size-mb', default=256, help='Memory size in MB (DEFAULT 256M)')
parse.add_argument('-v', '--vcpus', type=int, metavar='vcpus', default=1, help='Number of vcpus (default 1)')
parse.add_argument('-d', '--disk', type=int, metavar='size-gb', default=0, help='Disk size in GB (default 0G)')
parse.add_argument('-id', '--id', type=str, metavar='id', default='auto', help='Unique flavor ID')

if __name__ == '__main__':
    args = parse.parse_args()
    if args.option == 'create':
        print("i will create new flavor")
    if args.option == 'getall':
        print("i will inquire all flavor")
    if args.option == 'get':
        print("i will get flavor by id")
    if args.option == 'delete':
        print("i will delete flavor by id")
```

很显然，我还没有把具体实现填入，仅仅只是更改了输出用以区分，检查一下我们是否解析了所有的参数。

### 填入程序主体

因为接下来我要将实现填入了，这是程序中较为重要的一步。

- -n指定flavor名称，数据类型为字符串
- -m指定内存大小，数据类型为int，单位M
- -v指定虚拟cpu个数，数据类型为int
- -d指定磁盘大小，内存大小类型为int，单位G
- -id指定id，类型为字符串

在程序中我们已经做到了这些参数的解析，那么我们先填入create的实现，程序变成了下面这样。

```python
import argparse
import opentack

parse = argparse.ArgumentParser()
parse.add_argument('option')
parse.add_argument('-n', '--name', type=str, metavar='name', help='New flavor name')
parse.add_argument('-m', '--ram', type=int, metavar='size-mb', default=256, help='Memory size in MB (DEFAULT 256M)')
parse.add_argument('-v', '--vcpus', type=int, metavar='vcpus', default=1, help='Number of vcpus (default 1)')
parse.add_argument('-d', '--disk', type=int, metavar='size-gb', default=0, help='Disk size in GB (default 0G)')
parse.add_argument('-id', '--id', type=str, metavar='id', default='auto', help='Unique flavor ID')

conn = opentack.connect(
    auth_url='http://192.168.10.25:5000',
    username='admin',
    password='passwordadmin',
    project_name='admin',
    project_domain_name='Default',
)


def create_flavor(name, ram, vcpus, disk, flavorid):
    response = conn.create_flavor(name, ram, vcpus, disk, flavorid=flavorid)
    return response


if __name__ == '__main__':
    args = parse.parse_args()
    if args.option == 'create':
        print(create_flavor(args.name, args.ram,args.vcpus, args.disk, args.id))
    if args.option == 'getall':
        print("i will inquire all flavor")
    if args.option == 'get':
        print("i will get flavor by id")
    if args.option == 'delete':
        print("i will delete flavor by id")
```

我希望你还能看懂，在这一步，我导入了openstack这个包，我创建了一个连接对象`conn`，这样我就能操作openstack，我写入了一个方法`create_flavor`并指定了它可以传入的参数，当option读入create时，将会调用这个方法，并传入相应的参数。由方法中的连接对象调取create_flaovr方法来完成最后的创建，完成后，这个方法将response返回并打印到终端。

我将运行的结果贴在下方：

```python
(venv) PS D:\Python> python .\demo3.py create -n flavor_small -m 1024 -v 1 -d 10 -id 100000
openstack.compute.v2.flavor.Flavor(disk=10, OS-FLV-EXT-DATA:ephemeral=0, id=100000, os-flavor-access:is_public=True, name=flavor_small, ram=1024, rxtx_factor=1.0, swap=, vcpus=1, description=None, OS-FLV-DISABLED:disabled=False, extra_specs={}, location=Munch({'cloud': 'zed', 'region_name': '', 'zone': None, 'project': Munch({'id': '80074c3b4d09419e87ba0b8c05ce5164', 'name': 'admin', 'domain_id': None, 'domain_name': 'Default'})}))
```

很明显，这是一个正确的输出，如此，我就不废话了，将所有功能快速的填入，只需要根据功能创建剩余的三个方法即可。

```python
import argparse
import openstack

parse = argparse.ArgumentParser()
parse.add_argument('option')
parse.add_argument('-n', '--name', type=str, metavar='name', help='New flavor name')
parse.add_argument('-m', '--ram', type=int, metavar='size-mb', default=256, help='Memory size in MB (DEFAULT 256M)')
parse.add_argument('-v', '--vcpus', type=int, metavar='vcpus', default=1, help='Number of vcpus (default 1)')
parse.add_argument('-d', '--disk', type=int, metavar='size-gb', default=0, help='Disk size in GB (default 0G)')
parse.add_argument('-id', '--id', type=str, metavar='id', default='auto', help='Unique flavor ID')

conn = openstack.connect(
    auth_url='http://192.168.10.25:5000',
    username='admin',
    password='passwordadmin',
    project_name='admin',
    project_domain_name='Default',
)


def create_flavor(name, ram, vcpus, disk, flavorid):
    response = conn.create_flavor(name, ram, vcpus, disk, flavorid=flavorid)
    return response

def getall_flavor():
    response = conn.list_flavors()
    return response

def get_flavor(id):
    response = conn.get_flavor_by_id(id)
    return response

def delete_flavor(id):
    response = conn.delete_flavor(id)
    return response
if __name__ == '__main__':
    args = parse.parse_args()
    if args.option == 'create':
        print(create_flavor(args.name, args.ram, args.vcpus, args.disk, args.id))
    if args.option == 'getall':
        print(getall_flavor())
    if args.option == 'get':
        print(get_flavor(args.id))
    if args.option == 'delete':
        print(delete_flavor(args.id))
```

### 总结

到这里整个命令行脚本就大体完成了，如果需要别的改动，只需要按照这个思路修改就可以了。更多的argparse模块的使用可以参照官方文档的解析，非常详细，而且配有非常多的案例，当然，这太麻烦了，我找到了一个写的很不错的文档推荐给大家，我自己写的就太糟糕了。

> [【python 学习杂记】argparse模块使用教程_Onedean的博客-CSDN博客_argparse 教程](https://blog.csdn.net/edc3001/article/details/113788716)
