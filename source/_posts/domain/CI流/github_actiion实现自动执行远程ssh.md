---
title: github_action实现远程ssh连接并执行linux命令
date: 2023-10-01 16:37:00
tags:
  - github_action
  - 服务器
  - ssh
  - 技术
categories: CI流
cover: https://niu.goree.tech/2024/12/22/16:47:261734857246298.png
---
> 作为一个新手入门`github_action`由于对不对成加密不熟悉，可谓走了很多弯路，历时两天终于把这个搞明白了

ok 正文开始

# 首先理解GitHub action

github action 是一个持续集成的工作流，可以检测仓库的事件，然后github提供了一个虚拟机，可以暂时执行一些事件。

# 构思
可以在提交更新到仓库中之后，与远程服务器建立ssh连接，然后就可以发送命令，让服务器执行一些命令，比如打包部署，拉取最新代码等
> 我这里就是简单的一个更新文章,所以就直接拉取最新仓库就行了

# 原理
ssh需要有公钥和私钥,先查询GitHub网站,参考官方博客`https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent`
然后生成对应的公钥和私钥,之后执行以下几步
1. 添加公钥到鉴权文件中, 会发现鉴权文件是空的,直接全部粘进去就行了
2. 把私钥添加到仓库的变量中
    > 注意私钥要复制全比如before,after不能落下
3. 建立github_action的工作流配置文件,修改需要执行的命令即可


```shell
name: deploy

on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Initialize SSH agent and deploy
        run: |
          eval $(ssh-agent -s)
          echo "$SSH_PRIVATE_KEY" > deploy.key
          mkdir -p ~/.ssh
          chmod 0600 deploy.key
          ssh-add deploy.key
          echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config
          ssh root@$SERVER_IP "cd /root/blog_data/source/_posts && git pull origin master"
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          SERVER_IP: ${{ secrets.SERVER_IP }}
```