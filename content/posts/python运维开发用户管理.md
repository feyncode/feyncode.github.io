---
title: "Python运维开发用户管理"
subtitle: ""
date: 2023-02-01T09:48:11+08:00
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

### python开发用户管理命令行工具

有了前面的基础，很容易就可以完成。

我们先理清编写程序的几个步骤：

- 创建解析对象
- 添加解析参数
- 获得解析对象
- openstack命令执行，当需要命令行参数时，使用解析对象获取

我们开始吧。

### 创建解析对象并添加解析参数

```python
import argparse

parse = argparse.ArgumentParser()
parse.add_argument('option')
parse.add_argument('-n', '--name', type=str, metavar='name', help='user name')
parse.add_argument('-i', '--input', type=str, metavar='', help='user message')
parse.add_argument('-o', '--output', type=str, metavar='output', help='output')
```

### openstack定义相应行为

```python
import openstack

conn = openstack.connect(
	auth_url='http://192.168.10.25:5000',
	username='admin',
	password='passwordadmin',
    project_name='admin',
    project_domain_name='Default',
)

def create_user(name, password, description):
    response = conn.create_user(name, password, description, domain_id='default')
    return response

def getall_users(output):
    response = conn.list_users()
    if output:
        file = open(output, mode='a', encoding='utf-8')
        print(response, file=file)
        file.close()
    else:
        print(response)
    return response

def get_user(name, output):
    response = conn.get_user(name)
    if output:
        file = open(output, mode='a', encoding='utf-8')
        print(response, file=file)
        file.close()
    else:
        print(response)
    return response

def delete_user(name):
    resposne = conn.delete_user(name)
    return response
```

### 获得解析参数，完善程序

```python 
import json
if __name__ == '__main__':
    args = parse.parse_args()
    if args.option == 'create':
        dict = json.loads(args.input)
        print(create_user(dict['name'], dict['password'], dict['description']
    if args.option == 'getall':
        getall_users(args.output)
    if args.option == 'get':
        get_user(args.name, args.output)
    if args.option == 'delete':
        print(delete_user(args.name))       
```

程序整体就是这样了，只要理清程序的逻辑，将它完整的复现并不难。
