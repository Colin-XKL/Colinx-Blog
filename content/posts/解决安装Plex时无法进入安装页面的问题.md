---
title: 解决安装Plex时无法进入安装页面的问题
date: 2023-12-31
description: 近期在部署 Plex 的时候死活找不到安装初始化的页面, 看了几遍安装文档和官网的流程, 都没找到问题在哪. 后来 Google 了一圈, 才意识到自己这次不是局域网环境部署, Plex 为了安全性考虑, 每个新实例的安装部署页面是只对局域网访问的 IP 开放的, 甚至不是全部局域网 IP, 只有本机(Localhost). 这里简单记录下排查和解决的流程, 其他朋友在这块可以少踩些坑.
draft: false
categories:
  - 指南
tags:
  - Plex
  - Linux
  - SSH
  - Docker
---



Plex 是一款非常好用的影音库, 可自托管, 支持 Web, 颜值在线. 近期在部署 Plex 的时候死活找不到安装初始化的页面, 看了几遍安装文档和官网的流程, 都没找到问题在哪. 后来 Google 了一圈, 才意识到自己这次不是局域网环境部署, Plex 为了安全性考虑, 每个新实例的安装部署页面是只对局域网访问的 IP 开放的, 甚至不是全部局域网 IP, 只有本机(Localhost). 这里简单记录下排查和解决的流程, 其他朋友在这块可以少踩些坑.

问题页面如图所示, 正常来说新 Plex 服务器是要设定电影资料库, 电视节目资料库的文件夹位置, 绑定 Plex 账号等. 官方文档和其他人的流程都大致如此. 但是我访问这个新 Plex 实例对应的初始化 URL, 进来怎么也找不到配置文件夹位置的入口.

![Pasted image 20221216213737](https://blog-1301127393.file.myqcloud.com/BlogImgs/202312312254779.png)
后来在这两个地方的资料才发现一些端倪, 人家一般都是在 NAS 本机安装, 然后访问本机地址进入初始化页面, 但是我这个就不是.

https://askubuntu.com/questions/1321062/plex-installation-doesnt-give-option-to-setup-server

https://www.linode.com/docs/guides/install-plex-media-server-on-ubuntu-18-04/#configuring-plex-media-server-on-ubuntu-1804

知道了问题所在, 解决方案还是比较简单的, 那就是 SSH 连接使用端口转发, 将目标服务器 Plex 网页服务的端口, 映射到本机的一个端口. 这样访问本机的这个端口, 就能访问到 Plex 服务了, 而对于 Plex 服务器的 Web 端来说, 我们就是访问的本机地址.

访问 http://localhost:8888/web/ 即可看到Plex的页面. 如果没有带上后面的`web` 后缀的话, 直接访问可能会显示一个XML页面.

```bash
ssh user@192.0.2.1 -L 8888:localhost:32400
```

上面这句话就是ssh连接到192.0.2.1这个远端,然后在本地,即ssh客户端的机器,监听8888端口,本地所有访问8888端口的请求都会被重定向,等同为远端机器访问自身的32400端口,即为上文提到的,部署plex服务的端口


![Pasted image 20221216213514](https://blog-1301127393.file.myqcloud.com/BlogImgs/202312312254781.png)
如下是正常的初始化页面:
![Pasted image 20221216214020](https://blog-1301127393.file.myqcloud.com/BlogImgs/202312312254782.png)
有些情况下可能会遇到需要额外执行关联操作的情况, 不过问题不大

![Pasted image 20221216214456](https://blog-1301127393.file.myqcloud.com/BlogImgs/202312312254783.png)
![Pasted image 20221216214521](https://blog-1301127393.file.myqcloud.com/BlogImgs/202312312254784.png)

配置成功后就可以正常享受影音资源啦 😎

![Pasted image 20231231225353](https://blog-1301127393.file.myqcloud.com/BlogImgs/202312312254785.png)
