---
title: "Openstack命令行参考"
subtitle: ""
date: 2023-02-01T08:48:56+08:00
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

hello，大家好，这里是费冰。在使用OpenStack的过程中，固然我们可以通过 web 页面完成绝大多数的操作，但作为管理人员，不能不知晓 OpenStack 命令行的有关知识。

OpenStack 是一个开源的公有云和私有云的云计算平台，一系列的相关项目提供了一套云基础设施的解决方案。当我们提及OpenStack命令时，一般指两种命令。

其一，OpenStackClient 项目提供了一个统一的命令行客户端，它使我们能够通过易于使用的命令访问项目API，创建和管理OpenStack中的资源。

其二，大多数的OpenStack项目为每个服务提供了一个命令行客户端，我们可以通过项目命令来创建和管理项目中的有关资源。

在命令行或脚本中执行命令之前，我们必须提供 OpenStack 凭据，这些命令才能够成功运行。

在使用 OpenStack API 前，我假设你熟悉 HTTP， RESTful Web服务，OpenStack服务和JSON或XML数据序列化格式。

## 约定

本文档使用多种排版约定。

其实是我用这个语雀还不太熟，这和markdown还不太一样，很多语法不是共通的，而且显示代码块也不是我常用的格式，那么为了阅读不烧cpu。作以下约定。

```plain
这里是代码，比如说
openstack --version
```

为了方便理解和阅读，在介绍每个命令时，会在其后给出示例。

```plain
[root@controller ~]# openstack --version    # 这里是演示的代码
openstack 5.4.0  #这里是代码的输出
```

并且，在命令中的大写字段为需要替换的在实际环境中的具体字段。

本文档参考了无名小歌的博客，以下是博客原文。

[【收藏级】88条关于OpenStack命令的手册（常看常新）](https://blog.csdn.net/qq_48450494/article/details/125557500)

## 显示OpenStack客户端版本号

为了查看 openstack 客户端版本号，运行以下命令：

```plain
openstack --version
```

示例

```plain
[root@controller ~]# openstack --version
openstack 5.4.0
```

## 认证（keystone）

### 列出所有的用户

```plain
openstack user list
```

示例

### 列出认证服务目录

```plain
openstack catalog list
```

### 列出可用服务

```plain
openstack service list
```

### 创建用户

```plain
openstack user create --domain Default --password PASSWORD USERNAME
```

### 将项目和用户加入到角色中

```plain
openstack role add --project PROJECTNAME --user USERNAME ROLE
```

### 创建服务实体

```plain
openstack service create --name NAME --description “DESCTIPTION” SERVICE
```

### 创建 API 端点

OpenStack使用三个API端点来代表每种服务：admin, internal 和 public。默认情况下，管理API端点允许修改用户和租户而公共和内部API不允许这些操作。

```plain
openstack endpoint create --region RegionOne identity public http://controller:5000/v3
openstack endpoint create --region RegionOne identity internal http://controller:5000/v3
openstack endpoint create --region RegionOne identity admin http://controller:5000/v3
```

public代表公共地址，internal表示组件内部地址，admin则是管理员地址

### 查看用户信息

```plain
openstack user show USERNAME
```

### 列出所有的端点地址

```plain
openstack endpoint list
```

### 列出当前用户的token

```plain
openstack token issue
```

在openstack中，运行命令行前都必须通过身份验证，否则命令将无法正常运行。

一般想要通过身份验证，我们需要提供以下信息

- username 用户的名称
- user_domain_name 用户域的名称
- password 用户的密码
- project_name 项目名称
- project_domain_name 项目域的名称
- auth_url Keystone认证服务的API地址
- identity_api_version 应始终设置为3

我们一方面可以将这些值作为变量传递给openstack命令行，另一方面可以在环境中设置这些变量，显然在环境中设置更为简便，无需每次使用命令行都声明这些变量。

我们可以编写如下所示的环境变量文件，在系统中加载以便快速通过认证。

```plain
 export OS_USERNAME=my_username
 export OS_USER_DOMAIN_NAME=my_user_domain
 export OS_PASSWORD=my_password
 export OS_PROJECT_NAME=my_project
 export OS_PROJECT_DOMAIN_NAME=my_project_domain
 export OS_AUTH_URL=http://localhost:5000/v3
 export OS_IDENTITY_API_VERSION=3
```

## 镜像服务(glance)

glance在openstack中提供镜像服务，glance中的镜像可以处于以下几种状态之一：

- queued glance服务已经创建了镜像的注册表，但是并没有上传实体镜像，这时候镜像大小为0。
- saving 表示镜像的原始数据正在上传至glance，需要注意的是，当镜像通过调用 post/images注册并且提供了“x-image-meta-location”信息头，则该镜像不会处于saving状态，因为镜像数据已经在其他的可用位置。
- uploading 表示已经进行导入镜像的过程中
- importing 表示已经进行导入调用，但是镜像尚未准备就绪供使用
- active 表示在glance中完全可用的镜像
- deactivated 表示不允许任何非管理员访问镜像。
- killed 表示上传镜像数据时发生错误，并且镜像不可读。
- deleted glance保留了镜像的信息，但是不再可供使用，该镜像将会在指定的日期自动删除。
- pending_delete 类似于删除，但是glance尚未删除镜像数据，该状态下镜像不可恢复。

### 列出镜像

```plain
openstack image list
```

### 查看镜像详细信息

```plain
openstack image show 镜像id/name
```

### 上传镜像

```plain
openstack image create cirros --disk-format qcow2 --container-format bare --public --file cirros-0.6.1-x86_64-disk.img
```

参数说明：

- disk-format： 磁盘格式，可选项有 raw, vhd, vhdx, vmdk, vdi, iso, **ploop** qcow2
- container-format: 容器格式，可选项有bare, ovf, aki, ari, ami, ova, docker, compressed
- --file 指定上传的镜像文件
- --public 指定镜像的访问权限，public表示公开可用，也可以设置为--private | --community | --shared

### 更新镜像

```plain
openstack image set 镜像id/name 镜像属性
```

示例，修改镜像cirrors的最小启动ram和最小启动磁盘

```plain
openstack image set cirrors --min-ram 1 --min-disk 2
```

### 删除镜像

```plain
openstack image delete 镜像id/name
```

### 镜像下载

```plain
openstack image save 镜像id/name --file 镜像保存文件
```

## 网络服务(neutron)

网络服务管理一般需要管理员对网络知识作相应了解，例如BGP动态录有，DHCP分发服务，浮动IP端口转发等等，这里我列举几个常用的命令。

网络创建有太多的参数可用，就留给各位补充吧。

### 创建内部网络

```plain
openstack network create NETWORK --internal
```

### 创建外部网络

```plain
openstack network create NETWORK --external 
```

可选参数补充：

- provider-network-type：虚拟网络类型，例如flat，geneve，gre，vlan，vxlan
- privider-physical-network 物理网络名称
- project 所属的项目

### 创建网络子网

```plain
openstack subnet create SUBNETNAME --subnet-range 192.168.10.0/24 --gateway 192.168.10.2 --network NETWORK
```

参数说明：

- subnet-range：子网范围
- gateway： 指定子网的网关
- network：指定子网所属的网络

### 列出所有网络

```plain
openstack network list
```

### 查看网络详细信息

```plain
openstack network show NETWORK
```

### 删除网络

```plain
openstack network delete NETWORK
```

如果删除网络前，删除的网络被占用，则删除失败。

### 创建路由器

```plain
openstack router create ROUTER
```

### 设置路由器外部网络

```plain
openstack router set ROUTER --external-gateway NETWORK --fixed-ip subnet=SUBNET,ip-address=192.168.10.10 
```

其中，

- external-gateway 用作路由器的外部网络
- fixed-ip 路由器的外部网络ip地址

### 设置路由器内部网络

```plain
openstack router add subnet ROUTER SUBNET
```

### 列出所有路由器

```plain
openstack router list
```

### 查看路由器详细信息

```plain
openstack router show ROUTER
```

### 删除路由器

```plain
openstack router delete ROUTER
```

## 计算服务(NOVA)

### 列出所有云主机类型

```plain
openstack flavor list
```

### 创建云主机类型

```plain
openstack flvaor create FLAVOR --ram 1024 --disk 10 --vcpu 2
```

参数说明：

- id 指云主机类型的id，默认创建uuid
- ram 指内存大小，单位为MB ，不指定时默认值为256M
- disk 磁盘大小，单位为GB，默认值为0
- vcpu cpu数量，默认为1
- ephemeral 指临时磁盘，单位为GB,默认为0
- swap 指swap磁盘，单位为MB
- rxtx-factor 指rx/tx因子，默认为1.0
- public 云主机类型的权限，public为公开可用，private则为不共享
- project 指定一个项目共享云主机类型，必须和private参数同时使用

### 查看云主机类型详细信息

```plain
openstack flavor show FLAVOR
```

### 列出所有安全组

```plain
openstack security group list
```

### 查看安全组中的规则

```plain
openstack security group rule list 安全组id/name
```

### 查看安全组规则的详细信息

```plain
openstack security group rule show 规则id
```

### 创建安全组

```plain
openstack security group create 安全组name
```

### 删除安全组

```plain
openstack security group delete 安全组id/name
```

### 删除安全组规则

```plain
openstack security group rule delete 规则id 
```

### 在安全组中添加规则

安全组规则非常多，常见的有icmp，tcp，udp，dns等等。

安全组规则在添加时需要指定方向，分为入口(ingress)和出口(egress)。

添加安全组规则时还需要选择策略类型，指定如icmp，igmp，tcp，udp的一种。

安全组规则当然还可以指定端口范围或ip，当你需要指定端口范围时，使用dst-port参数，当你需要指定ip时，使用remote-ip参数。需要说明的是，如果你希望指定一个端口，那么dst-port的参数可用是这样 22:22，这就指定22端口。

以下演示从入口方向放行所有icmp，tcp，udp规则

```plain
openstack security group rule create --protocol icmp --ingress 安全组name/id
openstack security group rule create --protocol tcp --ingress 安全组name/id
openstack security group rule create --protocol udp --ingress 安全组name/id
```

### 创建云主机

```plain
openstack server create --image 镜像id/name --flavor 云主机类型name/id --network 网络name/id 云主机name
```

参数说明：

- image 使用的镜像
- flavor 创建云主机应用的云主机类型
- network 将创建的云主机连接到的网络

更多可用参数不一一列举。

### 列出云主机

```plain
openstack server list
```

### 查看云主机的控制台日志

```plain
openstack console log show 云主机name/id
```

### 显示云主机的远程控制台 URL

```plain
openstack console url show 云主机name/id
```

### 云主机的重启，关机，启动，暂停，取消暂停

说实话我很难区分暂停，挂起，释放。我相信你也是一样，所以让我来解释一下。

暂停，是指将虚拟机的状态保存到内存，暂停中的虚拟机仍然以冻结状态运行。

挂起，这是虚拟机管理器级别的操作，是指将虚拟机的内存保存到磁盘中，会释放虚拟机在宿主机中的资源，由libvirt完成该操作。内存信息会被保存到 /var/lib/libvirt/qemu/save/ 下。当再次启动该虚拟机时，会从该目录下使用内存文件启动，启动完成后，该文件会自动消失。

释放，也叫搁置，目的时将长时间不使用的虚拟机从底层释放从而节约服务器资源。一些长时间不使用的虚拟机，即使处于关机状态，仍然会占用集群资源，因此可用释放来释放资源。

如果虚拟机使用glance镜像创建，则创建一个快照并存储至glance，然后将虚拟机占用的资源从集群中删除；如果虚拟机从cinder存储中创建，则删除虚拟机占用的物理资源但保留磁盘，磁盘状态为保留。

```plain
openstack server reboot 云主机name/id
openstack server stop 云主机name/id
openstack server start 云主机name/id
openstack server pause 云主机name/id
openstack server unpause 云主机name/id
```

### 云主机的挂起，取消挂起

```plain
openstack server suspend 云主机name/id
openstack server resume 云主机name/id
```

### 云主机的释放和取消释放

```plain
openstack server shelve 云主机name/id
openstack server unshelve 云主机name/id
```

### 云主机创建快照

在创建云主机快照时，云主机状态应为关机。

```plain
openstack server image create 云主机name/id --name 快照name
```

如果不指定--name，则默认使用云主机name作为快照name

### 调整云主机大小

你需要确定nova中配置了scheduler_default_filters参数。

```plain
openstack server resize 云主机name/id --flavor 云主机类型name/id
openstack server resize --confirm 云主机名称
```

### 云主机挂载云硬盘

```plain
openstack server add volume 云主机name/id 磁盘id/name
```

### 创建密钥对

```plain
openstack keypair create test
```

可用重定向将输出结果导入到文件中。

### 创建云主机使用密钥对

```plain
openstack server create --image cirros --flavor os --key-name test 云主机name
```

这里的key-name参数指定的就是密钥对。

### 使用密钥连接云主机

```plain
ssh -i test.pem cirros@172.17.0.1
```

这里的test.pem就是创建密钥对时，将输出重定向保存的文件。

### 创建浮动ip

```plain
openstack floating ip create 网络name/id --floating-ip-address 172.17.0.1
```

如果不指定floating-ip-address，则由dhcp服务自动分发ip。

### 云主机绑定浮动ip

```plain
openstack server add floating ip 云主机name/id 浮动ip地址
```

例如：

openstack server add floating ip cirros 172.17.0.2

## 块存储(cinder)

### 显示所有卷

```plain
openstack volume list
```

### 创建新卷

```plain
openstack volume create 卷name --size 1 --type __DEFAULT__
```

参数说明：

- size 指定卷大小，单位为GB，此为必须参数
- type 指定卷类型
- read-only 指定卷为只读，默认情况下为可读写
- image 将镜像作为卷资源添加至卷中

### 创建卷类型

```plain
openstack volume type create 卷类型name
```

### 将卷连接到云主机

```plain
openstack server add volume 云主机name/id 卷name/id
```

可选参数：

- device 指定卷内部名称
- enable-delete-on-termination 意思是云主机被清除后删除卷
- disable-delete-on-termination 云主机被清除后保留卷

### 扩展卷

在卷状态为 available的前提下。

```plain
openstack volume set 卷name/id --size 调整的大小
```

实际上set可以更新很多volume的属性，以下是部分可选参数。

- name 设置卷name
- size 以GB为单位设置卷大小
- type 设置卷类型

## 对象存储(swift)

在开始之前，你需要自行了解什么是对象存储，以及对象存储是如何存储数据的。

### 创建容器

```plain
openstack container create 容器name
```

### 列出容器

```plain
openstack container list
```

### 查看容器详细信息

```plain
openstack container show 容器name/id
```

### 创建对象，上传对象

```plain
openstack object create create 容器name/id 文件name/id 
```

### 列出容器中的对象

```plain
openstack object list 容器name/id
```

### 查看对象详情

```plain
openstack object show 容器name/id 对象name/id
```

### 从容器中下载对象

```plain
openstack object save 容器name/id 对象name/id
```

### 删除对象

```plain
openstack object delete 容器name/id 对象name/id
```

### 删除容器

```plain
openstack container delete 容器name/id
```

只能删除空容器

### 递归删除容器

```plain
openstack container delete 容器name/id -r
```

参数说明：

- r  是recursive的简写，指先删除容器内存储的对象，再删除容器本身

## 模板编排服务（heat）

orchestration服务可以实现对多个组合云应用的编排，这个服务支持通过REST API 使用原生HOT模板。

这些灵活的模板语言使应用程序开发人员能够借以描述openstack的资源类型，如实例，浮动ip地址，卷，安全组，用户等等。这些资源一旦创建，将被称为stacks，可以通过heat服务加以管理。

### 编排创建cinder卷

```plain
$ cat cinder.yaml
heat_template_version: "2018-08-31"
resources: 
  Volume_1: 
    type: "OS::Cinder::Volume"
    properties: 
      name: heat_cinder1
      size: 2
      volume_type: "__DEFAULT__"
#创建堆栈，并创建volume
openstack stack create -t cinder.yaml cinder-1
```

### 编排创建swift容器

```plain
$ cat swift.yaml
heat_template_version: "2018-08-31"
resources: 
  Container_1: 
    type: "OS::Swift::Container"
    properties: 
      name: heat_swift
      "X-Container-Write": true

#创建堆栈,并创建swift
$ openstack stack create -t swift.yaml swift-1（栈名称）
```

### 编排创建network和subnet

```plain
heat_template_version: "2018-08-31"
resources: 
  Network_1: 
    type: "OS::Neutron::Net"
    properties: 
      admin_state_up: true
      name: Heat_Network
      shared: true
  Subnet_1: 
    type: "OS::Neutron::Subnet"
    properties: 
      name: "Heat-Subnet"
      ip_version: 4
      cidr: "172.17.0.0/24"
      network_id: 
        get_resource: Network_1
      
# #创建堆栈,并创建网络
$ openstack stack create -t network.yaml network（栈名称）
```

### 编排创建flavor

```plain
$ cat falvor.yaml
heat_template_version: "2018-08-31"
resources:
  Nova_flavor:
    type: OS::Nova::Flavor
    properties:
      name: os.flavor
      disk: 20
      is_public: True
      ram: 1024
      vcpus: 2
      
# 创建堆栈,并创建falvor
$ openstack stack create -t falvor.yaml flavor
```

### 编排创建云主机

```plain
$ cat server.yaml
heat_template_version: "2018-08-31"
resources:
  Server_1:
    type: "OS::Nova::Server"
    properties:
      networks:
        - network: "Heat_Network"
      flavor: "os.flavor"
      name: Heat_Server
      image: "cirros-0.6.1-x86_64-disk.img"

# #创建堆栈,并创建server
$ openstack stack create -t server.yaml server（栈名称）
```

### 编排创建user

```plain
$ cat user.yaml
heat_template_version: "2018-08-31"
resources:
  user:
    type: OS::Keystone::User
    properties:
      name: "Heat-User"
      password: "123456"
      domain: "default"
      default_project: "admin"
      roles: [{"role": admin, "project": admin}]

# 创建堆栈,并创建user
$ openstack stack create -t user.yaml user（栈名称）
```

### 列出所有堆栈

```plain
openstack stack list
```

### 列出堆栈详细信息

```plain
openstack stack show 堆栈name/id
```

### 删除堆栈

```plain
openstack stack delete 堆栈name/id -y
```

## 文件共享服务(manila)

openstack共享文件系统服务manila为虚拟机提供文件存储，简单点说，和云盘差不多，但是这是一个专用于虚拟机的云盘。

在开始之前，我浅浅的解释一下什么是DHSS。在部署manila时，有两个驱动模式可选，其一是不使用驱动程序来支持共享服务器的管理，在这种模式下，需要部署nfs服务器和一块manila卷组的附加磁盘，这种模式称为DHSS=False模式；其二，选择将manila与管理共享服务器的后端驱动程序一起运行，因此需要安装nova，glance，neutron一同管理的驱动程序，和块存储服务cinder来创建共享，

这种模式称为DHSS=True模式。

在官方示例中，为简单期间，配置使用DHSS=False模式。但是个人觉得DHSS=True的配置更为简单，仅仅只是配置项多了一部分，换来集中的管理和磁盘的节约，并且在某些发行版中，manila服务无法正确运行LVM驱动程序。

### 创建禁用DHSS的默认共享类型

```plain
manila type-create default_share_type False
```

### 列出共享类型

```plain
manila type-list
```

### 创建共享目录

```plain
manila create NFS 1 --name 共享名称
```

参数说明：

- share_protocol 共享协议（NFS,CIFS,CephFS,GlusterFS,HDFS,MAPRFS）
- size 共享大小，单位为GB
- --name 共享名称

### 列出共享目录

```plain
manila list
```

### 查看共享目录详情

```plain
manila show 共享名称
```

### 允许访问共享目录

```plain
manila access-allow 共享name/id ip 172.17.0.0/24
```

参数说明：

- share 共享的名称或者是id
- access_type 访问规则类型，仅限于ip，user，支持cert或cephx
- access_to 定义访问的网段ip
- access-level 定义共享访问权限，rw，ro均受支持，不指定时，默认为rw。

### 查看共享目录权限和开放网段

```plain
manila access-list 共享name/id
```

### 挂载共享目录

```plain
manila show 共享name/id |grep path
```

计算节点挂载共享目录

```plain
mount -t nfs 172.17.0.1:/path /mnt/
```
