---
title: "Python调用OpenStackAPI"
subtitle: ""
date: 2022-12-30T15:49:19+08:00
author: "feyncode"
authorLink: "https://github.com/feyncode"
authorEmail: "feyncode@gmail.com"
description: ""
#keywords: ""
#password:
message: 真正的大师永远都怀着一颗学徒的心

weight: 0

tags:
- openstack
- python
categories:
- openstack

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

> 本文将介绍如何使用 `python` 调用 `OpenStack API`。

### 什么是RESTful API

RESTful API 就是 `RESTful` 风格的 API。遵循 `RESTful` 风格开发的API被叫做 `RESTful API`。

那么什么是 `RESTful`风格呢。

首先需要明确的是，**REST并没有一个明确的标准，而是一种设计风格**，这种风格有这样几个主要特征:

- 统一接口，这是 `RESTful` 设计的基础。这意味着服务器以标准格式传输信息。
- 无状态，客户端可以以任意顺序请求资源，每个请求为无状态或独立于其他请求，这种设计让服务器每次都可以完全识别和满足请求。
- 层次化系统，在这种架构中，客户端可以连接到客户端和服务器之间的其他授权中间方，其仍将接受来自服务器的响应。服务器也可以将请求传递到其他服务器。

### RESTful API如何工作？

RESTful API基于 `HTTP` 协议，以下是在调用 `REST API`时发生的一系列步骤:

1.客户端向服务器发送请求，客户端以服务器可识别的方式设置请求格式。

2.服务器对客户端进行身份验证，并确认客户端有权发送该请求。

3.服务器接受请求并对其进行内部处理。

4.服务器向客户端返回响应。响应包含通知客户端请求是否成功的信息。

### RESTful API 客户端请求包含哪些内容？

RESTful API 要求请求包含以下主要组件:

##### 唯一的资源标识符(URL)

服务器通过唯一的资源标识符识别每个资源。对于 `REST`服务，服务器通常使用统一的资源定位符(URL)执行资源识别。URL指定资源的路径。URL也称为请求端点，并向服务器清晰的指明客户端请求的内容。

##### 方法

也就是 `HTTP方法`，这将通知服务器需要对资源执行什么操作。一般常见的 HTTP 方法有以下几种。

| 方法名 |                      描述                       |
| :----: | :---------------------------------------------: |
|  GET   | 访问位于服务器上指定URL上的资源，查询资源的列表 |
|  POST  |  向服务器发送数据，在标识的资源后创建新的数据   |
|  PUT   |    向服务器发送数据，更新服务器上的现有资源     |
| DELETE |             使用DELETE请求删除资源              |

##### HTTP头

也叫请求头或 `HEAD`，请求头时客户端和服务器之间交换的元数据，它包含了请求和响应的格式，提供有关请求状态的信息等。

##### 数据

也就是请求正文或者叫 `body`体，包含了HTTP方法成功运行所需的数据。

### RESTful API服务器响应包含哪些内容？

##### 状态码

状态码时三位的数字代码，表示请求成功或者请求出现故障。常见的状态代码有这些:

- 200: 通用成功响应
- 201: POST方法成功响应
- 400: 服务器无法处理的不正确请求
- 401: 未找到资源

##### 信息正文

服务器返回的数据实体，一般以JSON格式数据返回给客户端。

### OpenStack API操作

> [OpenStack API文档 — OpenStack API Documentation 文档](https://docs.openstack.org/zh_CN/api-quick-start/)

OpenStack API就是遵循RESTful 风格开发的，因此我们可以根据OpenStack API说明文档来调用OpenStack RESTful 接口完成相关的操作。

### 获取令牌

在使用 RESTful API 管理OpenStack资源之前，需要通过认证服务认证，这里的认证服务一般指`Keystone`，对应的API为 `Identity`。

> [Identity API v3 (CURRENT) — keystone documentation (openstack.org)](https://docs.openstack.org/api-ref/identity/v3/)

在 `Identity` API 说明中，我们知道，每次向 OpenStack 服务发出 REST API 请求时，都需要在请求标头中提供 `X-Auth-Token`身份验证令牌。

OpenStack Identity通过定义RBAC策略规则来保护其API。

#### 获取令牌的例子

让我们用一个例子来直观的感受RESTful API的调用过程。

假设我们的Openstack集群中的 `controller`节点IP地址为 `192.168.100.10`, 登录用户为`admin`，密码为`000000`，域为`demo`。

那么我们应该如何获取有效的`token`呢。

这里使用 APIFOX 来演示，APIFOX 是API一体化协作平台，可以用它进行接口测试。

我们先来创建一个`post`示例，如下图所示:

![image-20221229213322469](https://gitee.com/feyncode/images/raw/master/gitee/image-20221229213322469.png)

其中方法使用 `POST`，URL填写`http://192.168.100.10:5000/identity/v3/auth/tokens`，

在这个示例中，请求头，也就是`Headers`，加入`Content-Type: application/json`，因为我们传入的数据是 `json`格式的，在请求正文中，也就是Body，填入以下信息:

~~~json
{
    "auth": {
        "identity": {
            "methods": [
                "password"
            ],
            "password": {
                "user": {
                    "domain": {
                        "name": "demo"
                    },
                    "name": "admin",
                    "password": "000000"
                }
            }
        },
        "scope": {
            "project": {
                "domain": {
                    "name": "demo"
                },
                "name": "admin"
            }
        }
    }
}
~~~

完成后我们点击发送，就可以获得以下返回![image-20221230103230171](https://gitee.com/feyncode/images/raw/master/gitee/image-20221230103230171.png)

返回状态码为201，说明创建成功，接下来我们看看返回了哪些数据。

首先是 `Body`，也就是信息正文，这部分包含了用户信息和所有可操作的 api 的 `endpoint`，也就是 `URL`。

![image-20221230103459016](https://gitee.com/feyncode/images/raw/master/gitee/image-20221230103459016.png)

接着是 `Header`，这部分的 `X-Subject-Token`就是我们需要的身份验证令牌，也就是`Token`。我们可以直接复制来进行下一步操作。

有了身份验证令牌，我们就可以调用 OpenStack 的 API 来完成我们需要的操作了。

比如，我们想要查看 OpenStack 平台中的用户信息。

> [Identity API v3 (CURRENT) — keystone documentation (openstack.org)](https://docs.openstack.org/api-ref/identity/v3/index.html?expanded=list-users-detail#users)

![image-20221230104359284](https://gitee.com/feyncode/images/raw/master/gitee/image-20221230104359284.png)

我们可以发送 `GET` 请求来完成这一操作。

在进行这一操作前，我们需要明确以下几个要素

- URL： http://192.168.100.10:5000/v3/users
- HTTP方法：`GET`
- Header： `X-Auth-Token: token`
- Body: `Null`

![image-20221230104612378](https://gitee.com/feyncode/images/raw/master/gitee/image-20221230104612378.png)

可以看到，这样的请求被OpenStack API 接受并正确的处理，返回的状态码为200，返回的`Body`中，用户信息以`json`格式显示。

在经过这样几个例子之后，如何使用Python进行调用就变得很容易了。

### Python调用OpenStack API

首先打开Pycharm，这是一个Python IDE，在Pycharm中，我们需要使用Python来复现之前的操作。

需要用到的库有`json`和`requests`，这两个库将帮助我们处理json格式的数据和发送HTTP请求。

第一步我们需要获取身份验证令牌，也就是`Token`。

编写以下Python文件:

```python
import json,requests
# URL
url = f"http://192.168.100.10:5000/v3/auth/tokens"
# 请求正文，Body
body = {
    "auth": {
        "identity": {
            "methods": [
                "password"
            ],
            "password": {
                "user": {
                    "domain": {
                        "name": "demo"
                    },
                    "name": "admin",
                    "password": "000000"
                }
            }
        },
        "scope": {
            "project": {
                "domain": {
                    "name": "demo"
                },
                "name": "admin"
            }
        }
    }
}
# 请求头，Header
headers = {
    "Content-Type": "application/json"
}
# 发送HTTP方法为POST
response = requests.post(url, json.dumps(body), headers)
# 输出返回的Header中的token
print(response.headers["X-Subject-Token"])
```

运行的输出就是我们需要的身份验证令牌`Token`。

那么接下来，我们如何去使用 `Token`来调用其他的OpenStack API呢。

首先我们需要知道OpenStack组件的API地址，也就是`URL`，在OpenStack平台`controller`节点界面中键入以下命令:

```shell
openstack endpoint list -c 'Service Name' -c 'URL'
```

可以看到以下输出:

```shell
+--------------+-----------------------------+
| Service Name | URL                         |
+--------------+-----------------------------+
| neutron      | http://controller:9696      |
| nova         | http://controller:8774/v2.1 |
| placement    | http://controller:8778      |
| glance       | http://controller:9292      |
| neutron      | http://controller:9696      |
| neutron      | http://controller:9696      |
| keystone     | http://controller:5000/v3/  |
| nova         | http://controller:8774/v2.1 |
| glance       | http://controller:9292      |
| keystone     | http://controller:5000/v3/  |
| placement    | http://controller:8778      |
| nova         | http://controller:8774/v2.1 |
| keystone     | http://controller:5000/v3/  |
| glance       | http://controller:9292      |
| placement    | http://controller:8778      |
+--------------+-----------------------------+
```

这就是当前OpenStack平台中API的`URL`。

我们试着查询OpenStack平台中的用户数据，编写以下python代码

```pyrhon
class user:
    def __init__(self,token):
        self.url = f"http://192.168.100.10:5000/v3/users"
        self.headers = {
            "X-Auth-Token": token
        }
    def list(self):
        response = requests.get(self.url, headers=self.headers)
        return response.json()

if __name__ == '__main__':
    user = user(response.headers["X-Subject-Token"])
    userlist = user.list()
    for i in userlist['users']:
        print(i)
```

其中`response.headers["X-Subject-Token"]`就是我们之前拿到的`Token`，你可以直接复制使用之前获取的Token。

输出的结果如下:

```python
{'name': 'admin', 'links': {'self': 'http://192.168.100.10:5000/v3/users/a25fd183981a4c8393dca267a149e3b9'}, 'domain_id': 'c9a6a175f6614182a72cf2ea13915c29', 'enabled': True, 'options': {}, 'default_project_id': '5271453457a441f7afe80175b4bcc830', 'id': 'a25fd183981a4c8393dca267a149e3b9', 'password_expires_at': None}
{'password_expires_at': None, 'name': 'demo', 'links': {'self': 'http://192.168.100.10:5000/v3/users/1b83726b7b60402d929fb1d411d277c1'}, 'domain_id': 'c9a6a175f6614182a72cf2ea13915c29', 'enabled': True, 'id': '1b83726b7b60402d929fb1d411d277c1', 'options': {}}
{'password_expires_at': None, 'name': 'glance', 'links': {'self': 'http://192.168.100.10:5000/v3/users/80132166cf7e43bdbe22b155870caca5'}, 'domain_id': 'c9a6a175f6614182a72cf2ea13915c29', 'enabled': True, 'id': '80132166cf7e43bdbe22b155870caca5', 'options': {}}
{'password_expires_at': None, 'name': 'placement', 'links': {'self': 'http://192.168.100.10:5000/v3/users/8bb206274d9f419aafcb302a4c62cc23'}, 'domain_id': 'c9a6a175f6614182a72cf2ea13915c29', 'enabled': True, 'id': '8bb206274d9f419aafcb302a4c62cc23', 'options': {}}
{'password_expires_at': None, 'name': 'nova', 'links': {'self': 'http://192.168.100.10:5000/v3/users/2bae281a78ae4964bd753370f8be64c4'}, 'domain_id': 'c9a6a175f6614182a72cf2ea13915c29', 'enabled': True, 'id': '2bae281a78ae4964bd753370f8be64c4', 'options': {}}
{'password_expires_at': None, 'name': 'neutron', 'links': {'self': 'http://192.168.100.10:5000/v3/users/276840fa4c9f43e6becc63c044a03af7'}, 'domain_id': 'c9a6a175f6614182a72cf2ea13915c29', 'enabled': True, 'id': '276840fa4c9f43e6becc63c044a03af7', 'options': {}}
```

现在已经明白如何使用Python调用OpenStack API了吧。

如果想要调用OpenStack平台其他组件的API，只需要查阅API文档，根据其不同方法编写对应的Python代码即可。

