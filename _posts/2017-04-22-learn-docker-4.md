---
layout: post
title: 'Docker 学习笔记4'
subtitle: '点滴记忆空间'
date: 2017-04-20
categories: 技术
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-theme-h2o-postcover.jpg'
tags: docker 
---


# Docker

## 9.  Docker compose

从上一节课我们了解到可以使用一个Dockerﬁle模板文件来快速构建一个自己的镜像并运行为应用容器。但是在平时工作的时候，我们会碰到多个容器要互相配合来使用的情况，比如数据库加上咱们Web应用等等。这种情况下，每次都要一个一个启动容器设置命令变得麻烦起来，所以Docker Compose诞生了。

### 9.1.  Dockercompse简介

Compose的作用是“定义和运行多个Docker容器的应用”。使用Compose，你可以在一个配置文件（yaml格式）中配置你应用的服务，然后使用一个命令，即可创建并启动配置中引用的所有服务。

Compose中两个重要概念：

- 服务 (service)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。

- 项目 (project)：由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml文件中定义。


### 9.2. Dockercompse安装

Compose支持三平台Windows、Mac、Linux，安装方式各有不同。我这里使用的是Linux系统，其他系统安装方法

可以参考官方文档和开源GitHub链接：

Docker Compose官方文档链接：https://docs.docker.com/compose

Docker Compose GitHub链接：https://github.com/docker/compose

Linux上有两种安装方法，Compose项目是用Python写的，可以使用Python-pip安装，也可以通过GitHub下载二进制文件进行安装。

**通过Python-pip安装**

1.安装Python-pip

```
yum install -y epel-release
yum install -y python-pip
```

2.安装docker-compose

```
pip install docker-compose
```

3.验证是否安装

```
docker-compose version
```

4.卸载

```
pip uninstall docker-compose
```

**通过GitHub链接下载安装**

 

非ROOT用户记得加sudo

1.通过GitHub获取下载链接，以往版本地址：https://github.com/docker/compose/releases

```
curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname-s)-$(uname -m)" -o /usr/local/bin/docker-compose 
```

2.给二进制下载文件可执行的权限

```
chmod +x /usr/local/bin/docker-compose
```

3.可能没有启动程序，设置软连接，比如:

```
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

4.验证是否安装

```
docker-compose version
```

5.卸载

如果是二进制包方式安装的，删除二进制文件即可。

```
rm /usr/local/bin/docker-compose
```

简单实例

 

Compose的使用非常简单，只需要编写一个docker-compose.yml，然后使用docker-compose 命令操作即可。

docker-compose.yml描述了容器的配置，而docker-compose 命令描述了对容器的操作

1.我们使用一个微服务项目先来做一个简单的例子，首先创建一个compose的工作目录，然后创建一个eureka文件夹，里面放可执行jar包和编写一个Dockerﬁle文件，目录结构如下：

```
compose
	eureka
		Dockerfile
		eureka-server-2.0.2.RELEASE.jar
```

2.在compose目录创建模板文件docker-compose.yml文件并写入以下内容

```
version: '1'
services:
	eureka:
		build: ./eureka
		ports:
		  - 3000:3000
		expose:
          - 3000 
```



### 9.3.  DockerCompose常用命令

##### 9.3.1.1.      image

指定镜像名称或者镜像id，如果该镜像在本地不存在，Compose会尝试pull下来。

示例：

```
image: java:8
```

##### 9.3.1.2.      build

指定Dockerﬁle文件的路径。可以是一个路径，例如：build: ./dir

也可以是一个对象，用以指定Dockerﬁle和参数，例如：

```
build: context: ./dir dockerﬁle: Dockerﬁle-alternate args: buildno: 1
```

##### 9.3.1.3.      command

覆盖容器启动后默认执行的命令。

示例：

```
command: bundle exec thin -p 3000
```

也可以是一个list，类似于Dockerﬁle总的CMD指令，格式如下：

```
command: [bundle, exec, thin, -p, 3000]
```

##### 9.3.1.4.      links

链接到其他服务中的容器。可以指定服务名称和链接的别名使用SERVICE:ALIAS 的形式，或者只指定服务名称，示

例：

```
web: links: - db - db:database – redis
```

##### 9.3.1.5.      external_links

表示链接到docker-compose.yml外部的容器，甚至并非Compose管理的容器，特别是对于那些提供共享容器或共同

服务。格式跟links类似，示例：

```
external_links: - redis_1 - project_db_1:mysql - project_db_1:postgresql
```

##### 9.3.1.6.      ports

暴露端口信息。使用宿主端口:容器端口的格式，或者仅仅指定容器的端口（此时宿主机将会随机指定端口），类似于docker run -p ，示例：

```
ports:
    "3000"
    "3000-3005"
    "8000:8000"
    "9090-9091:8080-8081"
    "49100:22"
    "127.0.0.1:8001:8001"
    "127.0.0.1:5000-5010:5000-5010"
```



##### 9.3.1.7.      expose

暴露端口，只将端口暴露给连接的服务，而不暴露给宿主机，示例：

```
expose: - "3000" - "8000"
```

##### 9.3.1.8.      volumes

卷挂载路径设置。可以设置宿主机路径 （HOST:CONTAINER） 或加上访问模式 （HOST:CONTAINER:ro）。示例：

volumes:

Just specify a path and let the Engine create a volume

- /var/lib/mysql


Specify an absolute path mapping

- /opt/data:/var/lib/mysql


Path on the host, relative to the Compose ﬁle

- ./cache:/tmp/cache


User-relative path

- ~/conﬁgs:/etc/conﬁgs/:ro

Named volums

- datavolume:/var/lib/mysql


##### 9.3.1.9.      volumes_from

从另一个服务或者容器挂载卷。可以指定只读或者可读写，如果访问模式没有指定，则默认是可读写。示例：

volumes_from:

- service_name

- service_name:ro

- container:container_name

- container:container_name:rw


##### 9.3.1.10.    environment

设置环境变量。可以使用数组或者字典两种方式。只有一个key的环境变量可以在运行Compose的机器上找到对应的

值，这有助于加密的或者特殊主机的值。示例：

```
environment: RACK_ENV: development SHOW: 'true' SESSION_SECRET: environment: -
RACK_ENV=development - SHOW=true - SESSION_SECRET
```



##### 9.3.1.11.    env_ﬁle

从文件中获取环境变量，可以为单独的文件路径或列表。如果通过 docker-compose -f FILE 指定了模板文件，则env_ﬁle 中路径会基于模板文件路径。如果有变量名称与 environment 指令冲突，则以envirment 为准。示例：

```
env_ﬁle: .env env_ﬁle: - ./common.env - ./apps/web.env - /opt/secrets.env
```

##### 9.3.1.12.    extends

继承另一个服务，基于已有的服务进行扩展。

##### 9.3.1.13.    net

设置网络模式。示例：

```
net: "bridge" net: "host" net: "none" net: "container:[service name or container name/id]"
```

##### 9.3.1.14.    dns

配置dns服务器。可以是一个值，也可以是一个列表。示例：

```
dns: 8.8.8.8 dns: - 8.8.8.8 - 9.9.9.9
```



##### 9.3.1.15.    dns_search

配置DNS的搜索域，可以是一个值，也可以是一个列表，示例：

```
dns_search: example.com dns_search: - dc1.example.com - dc2.example.com
```

##### 9.3.1.16.    其它

docker-compose.yml 还有很多其他命令，可以参考docker-compose.yml文件官方文档：

[https://docs.docker.com/compose/compose-ﬁle/](https://docs.docker.com/compose/compose-file/)

### 9.4.  使用DockerCompose构建实例

使用docker-compose一次性来编排三个微服务:eureka服务(eureka-server-2.0.2.RELEASE.jar)、user服务(user-2.0.2.RELEASE.jar)、power服务(power-2.0.2.RELEASE.jar)

1.创建一个工作目录和docker-compose模板文件

2.工作目录下创建三个文件夹eureka、user、power，并分别构建好三个服务的镜像文件

以eureka的Dockerﬁle为例:

```
# 基础镜像
FROM java:8
# 作者
MAINTAINER user01
# 把可执行jar包复制到基础镜像的根目录下
ADD eureka-server-2.0.2.RELEASE.jar /eureka-server-2.0.2.RELEASE.jar
# 镜像要暴露的端口，如要使用端口，在执行docker run命令时使用-p生效
EXPOSE 3000
# 在镜像运行为容器后执行的命令
ENTRYPOINT ["java","-jar","/eureka-server-2.0.2.RELEASE.jar"]
```

 

目录文件结构：

```
compose
	docker-compose.yml
	eureka
		Dockerfile
		eureka-server-2.0.2.RELEASE.jar
	user
		Dockerfile
		user-2.0.2.RELEASE.jar
	power
		Dockerfile
		power-2.0.2.RELEASE.jar
```

3.编写docker-compose模板文件：

```
version: '1'
services:
  eureka:
	image: eureka:v1
	ports:
      - 8080:8080
  user:
    image: user:v1
    ports:
      - 8081:8081
  power:
    image: power:v1
    ports:
      - 8082:8082
```

4.启动微服务，可以加上参数-d后台启动

```
docker-compose up -d
```

