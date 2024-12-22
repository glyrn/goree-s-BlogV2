---
title: 基于ftp和nignx的视频服务器的原理与实现
date: 2023-08-07 23:34:00
keywords:
  - ftp
  - nginx
  - 视频服务器
description: 这篇文章介绍了基于ftp和nignx的视频服务器的原理与实现
tags:
  - nginx
  - ftp
  - docker
categories: 后端
cover: /img/default.avif
---
# 基于ftp和nignx的视频服务器的原理与实现

## 1.需要实现什么

一个基本的视频服务器需要具有以下几个功能

- 能够支持视频文件的上传与存储
- 能够实现视频等静态文件的远程访问

## 2.如何实现

- 视频文件的上传与存储
  - 可以通过配置支持ftp协议的服务器实现，在这个服务器中允许用户上传视频或者图片文件到制定目录，用户可以通过ftp客户端或者语言的第三方库比如go-ftp连接到ftp服务器，视频文件上传到ftp服务器之后，会保持持久化的操作
- 静态资源的访问
  - 可以通过nignx，tomcat等web服务器实现，通过修改nginx的配置文件，将nginx配置成为静态资源处理器，直接提供用户访问静态资源的能力。用户可以通过HTTP请求直接从nginx获取文件，不需要访问ftp服务器
- 权限控制
  - ftp身份认证，只有授权的用户才能上传视频文件
  - 与nginx访问控制限制，限制特定用户或者特定ip的访问权限

## 问题难点

- 用户怎么通过nginx，访问到ftp服务器上的静态资源

## 解决思路

1. 写shell命令，每当ftp的资源出现变动，把资源同步到nginx下面

> 这一个方法对服务器资源消耗较大

2. 内存共享

>  将ftp服务器存相关资源的内存指针与nginx解析资源的内存指针挂载到同一块内存地址上面，通过内存共享，能保证nginx可以获取到ftp服务器的静态文件

## 解决过程

我的服务器是centos7操作系统，为了实现内存共享，我使用了docker数据卷技术，通过数据卷挂载，实现docker数据持久化操作

1. 创建用于挂载的目录，这个目录将同时用于nginx容器与ftp容器数据卷挂载

```powershell
# 创建目录
cd ~
mkdir -f ftp/vedio ftp/image
```

2. 创建容器

```powershell
# 拉取镜像
docker pull vsftpd
docker pull nginx
# 启动容器
docker run -it --name nginx bash
# 创建nginx的数据文件夹 用于挂载操作
cd ~
mkdir data/
# 保存容器镜像 --nginx是没有data文件夹的 所以需要先自定义一个文件夹
docker commit myNginx:0.1
# 创建ftp容器
docker run -d -p 200:20 -p 210:21 -p 21100-21110:21100-21110 -v /root/ftp/Ftpfile:/home/vsftpd -e FTP_USER=user -e FTP_PASS=123456 -e PASV_ADDRESS=39.101.72.240 -e PASV_MIN_PORT=21100 -e PASV_MAX_PORT=21110 --name cloudFtp --restart=always fauria/vsftpd
# 创建nginx容器
docker run -p 9002:80 --name cloudNginx -v /root/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /root/nginx/conf/conf.d:/etc/nginx/conf.d -v /root/nginx/log:/var/log/nginx -v /root/nginx/html:/usr/share/nginx/html -v /root/ftp/Ftpfile/user:/root/data -d myNginx:0.1
```

3. 配置nginx

```
# /etc/nginx/nginx.conf

user root;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    sendfile on;
    keepalive_timeout 65;

    server {
        listen 80;

	server_name localhost;

        # 处理静态资源的根目录是挂载到容器内的 /root/data
        # root /root/data;

        location /images/ {
            # 当访问以 /images/ 开头的 URL 时，会去寻找 /root/data/images/ 目录下的文件
        	root /root/data;
		autoindex on;
	}

        location /videos/ {
            # 当访问以 /videos/ 开头的 URL 时，会去寻找 /root/data/videos/ 目录下的文件
        	root /root/data;
		autoindex on;
	}

        # 其他配置...
    }
}

```

5. 开放对应端口
