---
layout: post
title: 'Docker 学习笔记3'
subtitle: '点滴记忆空间'
date: 2017-04-20
categories: 技术
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-theme-h2o-postcover.jpg'
tags: docker 
---


# Docker

## 7.  Docker网络

Docker允许通过外部访问容器或容器互联的方式来提供网络服务。

安装Docker时，会自动安装一块Docker网卡称为docker0，用于Docker各容器及宿主机的网络通信，网段为172.0.0.1。

Docker网络中有三个核心概念：沙盒（Sandbox）、网络（Network）、端点（Endpoint）。

- 沙盒，提供了容器的虚拟网络栈，也即端口套接字、IP路由表、防火墙等内容。隔离容器网络与宿主机网络，形成了完全独立的容器网络环境。

- 网络，可以理解为Docker内部的虚拟子网，网络内的参与者相互可见并能够进行通讯。Docker的虚拟网络和宿主机网络是存在隔离关系的，其目的主要是形成容器间的安全通讯环境。

- 端点，位于容器或网络隔离墙之上的洞，主要目的是形成一个可以控制的突破封闭的网络环境的出入口。当容器的端点与网络的端点形成配对后，就如同在这两者之间搭建了桥梁，便能够进行数据传输了。


### 7.1.  Docker的四种网络

Docker服务在启动的时候会创建三种网络，bridge、host和none，还有一种共享容器的模式container

 **Bridge**

桥接模式，主要用来对外通信的，docker容器默认的网络使用的就是bridge。

使用bridge模式配置容器自定的网络配置

```
# 配置容器的主机名
docker run --name t1 --network bridge -h [自定义主机名] -it --rm busybox
# 自定义DNS
docker run --name t1 --network bridge --dns 114.114 -it --rm busybox
# 给host文件添加一条
docker run --name t1 --network bridge --add-host [hostname]:[ip] -it --rm busybox
```

 **Host**

 host类型的网络就是主机网络的意思，绑定到这种网络上面的容器，内部使用的端口直接绑定在主机上对应的端口，而如果容器服务没有使用端口，则无影响。

**None**

从某种意义上来说，none应该算不上网络了，因为它不使用任何网络，会形成一个封闭网络的容器

**container**

 共享另外一个容器的network namespace，和host模式差不多，只是这里不是使用宿主机网络，而是使用的容器网络.

## 8.  开放端口

Docker0为NAT桥，所以容器一般获得的是私有网络地址

给docker run命令使用-p选项即可实现端口映射，无需手动添加规则

- -p 选项的使用

    - -p <containerPort>

        - 将指定的容器端口映射到主机所有地址的一个动态端口

    - -p <hostPort>:<containerPort>

        - 将容器端口 <containerPort> 映射到指定的主机端口 <hostPort>

    - -p <ip>::<containerPort>

        - 将指定的容器端口 <containerPort> 映射到主机指定 <ip> 的动态端口

    - -p <ip>:<hostPort>:<containerPort>
- 将指定的容器端口 <containerPort> 映射至主机指定 <ip> 的端口 <hostPort>
    
 - 动态端口指随机端口，可以使用docker port命令查看具体映射结果

- -P 暴露所有端口（所有端口指构建镜像时EXPOSE的端口）

 

自定义docker0桥的网络属性信息：/etc/docker/daemon.json文件

```
{
 "bip": "192.168.1.5/24",
 "fixed-cidr": "10.20.0.0/16",
 "fixed-cidr-v6": "2001:db8::/64",
 "mtu": 1500,
 "default-gateway": "10.20.1.1",
 "default-gateway-v6": "2001:db8:abcd::89",
 "dns": ["10.20.1.2","10.20.1.3"]
}
```

核心选项为bip，即bridge ip之意，用于指定docker0桥自身的IP地址；其它选项可通过此地址计算得出

远程连接

创建自定义的桥

```
docker network create -d bridge --subnet "172.26.0.0/16" --gateway "172.26.0.1" mybr0
```

