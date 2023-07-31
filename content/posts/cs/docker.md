---
title: "Docker"
date: 2023-07-31T11:43:25+08:00
lastmod: 2023-07-31T11:43:25+08:00
author: ["chx9"]
keywords:
  -
categories: # 没有分类界面可以不填写
  -
tags: # 标签
  -
description: ""
weight:
slug: ""
draft: false # 是否为草稿
comments: true # 本页面是否显示评论
reward: false # 打赏
mermaid: true #是否开启mermaid
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示路径
cover:
  image: "" #图片路径例如：posts/tech/123/123.png
  zoom: # 图片大小，例如填写 50% 表示原图像的一半大小
  caption: "" #图片底部描述
  alt: ""
  relative: false
---

# docker 面对对象

容器 :对象

镜像 :类

# 常用命令

## docker run

每次都会新建一个 docker container，可以添加`-it`命令

`docker run -d image`: 在后台运行容器。

`docker run -it image sh`: 交互式地在前台运行容器（常用于调试）。

- `t`: 在新容器内指定一个伪终端或终端。
- `i`: 允许你对容器内的标准输入 (STDIN) 进行交互。
- `image`后面的`sh`选项可以不填，默认会启动镜像内的`CMD`或`ENTRYPOINT`指定的程序。但是如果这些程序不是交互式Shell，那么你将无法得到一个终端

`docker run -p 80:80 image`: 映射80端口到主机上的80端口。`docker run --rm image`: 容器结束后自动删除。

`docker run -v /host/dir:/container/dir image`: 挂载主机目录到容器中。

`docker run --rm -it prakhar1989/static-site` 如果已经存在，直接把旧的删除



`docker run -d -P --name static-site prakhar1989/static-site`  -

1. `-d`选项会使你的终端脱离，此命令将在后台运行容器。
2. `-P`选项将发布所有暴露的端口到随机端口。
3. `--name`对应我们想要给定的名称。
4. 可以通过运行`docker port [CONTAINER]`命令来查看端口

## docker rm

删除一个或者多个 container

`docker rm 305297d7a235 ff0a5c3750b9`

删除所有状态为已退出(status=exited)的 Docker容器

`docker rm $(docker ps -a -q -f status=exited)`

删除所有停止的容器: 

`docker container prune`

删除所有容器(无论是否运行):

 `docker rm $(docker ps -a -q)`

删除创建状态的容器: 

`docker rm $(docker ps -a -q -f status=created)`

删除所有未使用的数据(不只是容器):

 `docker system prune`

强制删除正在运行的容器: 

`docker rm -f $(docker ps -a -q)`

## docker ps
列出所有容器: 

`docker ps -a`

列出正在运行的容器: 

`docker ps`

列出最近创建的容器: 

`docker ps -l`

列出特定状态的容器, 如已退出(exited):

 `docker ps -a -f status=exited`

列出最近停止的n个容器:

 `docker ps -n`

以特定格式显示容器信息: 

`docker ps --format "{{.ID}}: {{.Status}}"`



## docker start 



**docker start变种：**

1. `docker start container_id`: 启动一个已经停止的容器。
2. `docker start -a container_id`: 启动容器并附着到其stdout/stderr和可选地stdin（与正在运行的容器一起使用）。 只显示输出
3. `docker start -i container_id`: 启动容器并附加（attach）TTY。 

**docker attach变种：**

1. `docker attach container_id`: 附着到运行中的容器以查看其输出或与其交互。
2. `docker attach --sig-proxy=true container_id`: 代理所有接收的信号到进程 (默认为true)。
3. `docker attach --no-stdin container_id`: 不附加STDIN。

