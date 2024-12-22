---
title: blog 部署SOP
date: 2024-12-22 01:41:41
categories: life
updated: 
cover: /img/default.avif
tags:
  - 脚本
  - 技术
---
## 同步知识库与blog库文件diff 

>注意为了方便个人pc使用，这里采用绝对路径直接调用

```shell
python D:\obsidianKnowledgeV2.0\space\life\博客部署\sync_blog.py
```

## 登录服务器执行服务重启

```shell
sh /root/goree_blog/obsidian/goree-s-BlogV2/restart.sh
```