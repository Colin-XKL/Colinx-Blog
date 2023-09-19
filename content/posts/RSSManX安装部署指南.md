---
title: RSSManX 安装部署指南
date: 2022-05-07
description: RSS Man X 是我两年前入坑 RSS 后，为了方便更多人更方便地使用 RSS 所发起的一个项目，主要是用 docker-compose 整合了一些常用的服务如 TTRSS、RSSHub、Huginn、OpenCC 等并进行了一些优化调整，如自动更新，反反爬虫等。这篇文章会尽可能详尽地讲解安装部署的步骤以及安装过程中可能会遇到的一些问题。
categories:
  - 技术
  - 指南
tags:
  - RSS
  - Linux
  - 教程
  - Docker
---

<!-- # RSSManX 安装部署指南 -->

RSS Man X 是我两年前入坑 RSS 后，为了方便更多人更方便地使用 RSS 所发起的一个项目，主要是用 docker-compose 整合了一些常用的服务如 TTRSS、RSSHub、Huginn、OpenCC 等并进行了一些优化调整，如自动更新，反反爬虫等。这篇文章会尽可能详尽地讲解安装部署的步骤以及安装过程中可能会遇到的一些问题。

RSS Man X 会利用到 docker 和 docker-compose，首先需要确保你的服务器正确安装了这些软件

## 1. Docker 环境准备

检查服务器中是否已经安装了 docker 和 docker-compose，并检查他们的版本是否太老。比如输入`docker --version`来检查 docker 的版本

docker 的版本建议不要低于 19，docker-compose 的版本建议不要低于 1.20  
(注：Docker 新版本可使用`docker compose` 替代 `docker-compose`)

```shell
~ » docker --version
Docker version 20.10.12, build 20.10.12-0ubuntu2~20.04.1
```

```shell
~ » docker-compose --version
docker-compose version 1.25.0, build unknown
```

如果上面命令执行后提示`command not found`，那么说明并没有安装对应的软件包（或是安装的路径并不在当前用户的 PATH 变量中，尝试切换到 root 操作，详细的原因和解决方法见后文）

### 1.1 Ubuntu/Debian 安装 Docker

ubuntu 和 debian 主要是用 apt 安装和管理软件包，可以使用以下命令来安装，根据实际情况选择是否要加 sudo。下面以 Ubuntu 为例。Debian 下的安装略有不同，可以参见[Docker 官方文档](https://docs.docker.com/engine/install/debian/)，以及[清华镜像源](https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/)。其中`download.docker.com`连接不同的可以尝试讲域名部分更换为[中科大提供的镜像](https://mirrors.ustc.edu.cn/help/docker-ce.html)地址`mirrors.ustc.edu.cn/docker-ce`

```shell
# 首先移除老旧的软件包
sudo apt-get remove docker docker-engine docker.io containerd runc

sudo apt-get update
# 安装一些必要的依赖
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

如果你的服务器在墙外，可以直接使用 docker 官方的软件源

```shell
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

如果你的服务器在墙内，可以使用由中科大提供的镜像

```shell
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

导入 docker 软件源后即可开始安装 docker，安装完成后可以输入`docker --version`来检查 docker 的版本

```shell
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

### 1.2 CentOS/Fedora 安装 Docker

CentOS 和 Fedora 使用 yum 安装和管理软件包，新版本则是使用 dnf，不过两者功能一致且高度兼容。下面以 CentOS 为例，Fedora 可参见[官方文档](https://docs.docker.com/engine/install/fedora/)或[清华镜像源](https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/)

```shell
# 1. 首先移除老旧的软件包

sudo yum remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-engine
# 2.
sudo yum install -y yum-utils

# 3.A 之后使用官方docker软件库
sudo yum-config-manager \
   --add-repo \
   https://download.docker.com/linux/centos/docker-ce.repo

# 3.B 墙内方案
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo sed -i 's+download.docker.com+mirrors.tuna.tsinghua.edu.cn/docker-ce+' /etc/yum.repos.d/docker-ce.repo


# 4. 即可开始安装
sudo yum update
sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

### 1.3 安装或更新 docker-compose

docker-compose 目前有两个主要版本 V1 和 V2，都是可用的。V1 使用 Python 编写，使用时类似`sudo docker-compose up -d`，V2 则是 Golang 编写，与前者高度兼容，但是是作为 docker 的插件安装的，使用时类似`sudo docker compose up -d`，中间的短杠不需要了。

如果是按照上文的步骤安装的 docker，那么默认已经安装了 docker compose v2，可以通过`docker compose version`查看版本

```shell
~ » docker compose version
Docker Compose version v2.3.3
```

如果需要是安装的比较老的 v1 版本的 docker-compose，想要单纯更新 docker-compose 而又不想动其他东西的话，可以更新安装 v1 版本的 docker-compose。

首先删除老旧版本（如果有的话）

```shell
rm $DOCKER_CONFIG/cli-plugins/docker-compose
sudo rm /usr/local/lib/docker/cli-plugins/docker-compose
pip uninstall docker-compose
```

然后使用 pip 来安装

```shell
pip3 install docker-compose
```

墙内也可以临时使用镜像源下载

```shell
pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple
```

如果提示 pip 命令不存在需要先安装一下

```shell
# Ubuntu/Debian
sudo apt install python3-pip

# CentOS/Fedora
sudo yum install python3-pip
```

如果 pip 报错试着升级一下 pip 的版本

```shell
python3 -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --upgrade pip
```

### 1.4 为 Docker 配置镜像源

如果 Docker 下载镜像非常慢，你可能需要单独配置一下 Docker 的镜像源。==**注意：Docker 安装软件源的镜像和 Docker 镜像或者说映像的国内源并不是同一个，英文表述可能更准确一些：**==

- 上文安装 docker 时配置的是 docker 的 repo，repo 里含有 docker 的软件包，[国内的镜像](https://mirrors.ustc.edu.cn/help/docker-ce.html)为 Docker CE 安装软件包的镜像
- 现在我们要配置的是 Docker Hub 的 Mirror，Docker 里拉取 Image 默认会访问[Docker Hub](https://hub.docker.com)，国内有多个 Docker Hub 镜像或是 Docker 镜像加速器，如[中科大的源](https://mirrors.ustc.edu.cn/help/dockerhub.html)

在`/etc/docker/daemon.json`文件中写入以下内容，如不存在可先行创建，注意 json 中列表的最后一项末尾是没有逗号的

```json
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com",
    "https://docker.nju.edu.cn",
    "https://docker.mirrors.sjtug.sjtu.edu.cn",
    "https://dockerproxy.com",
    "https://docker.m.daocloud.io"
  ]
}
```

update 20230720: 部分镜像源不再可用，更新镜像源配置。参考 https://gist.github.com/y0ngb1n/7e8f16af3242c7815e7ca2f0833d3ea6

配置完成后`sudo systemctl restart docker`重启 docker 服务，然后输入`sudo docker info`，在输出结果的末尾可以看到`Registry Mirrors`里会出现我们刚刚配置的 Docker Hub 镜像

群辉等 NAS 的系统并不是标准版 Linux，安装的也是魔改版 docker，上面的配置文件地址并不适用，建议自行搜索对应的文档或教程。

## 2. 安装 RSS Man X

[RSS MAN X 项目](https://github.com/Colin-XKL/RSSmanX)旨在为 RSS 的同好提供一个方便地搭建自己的 RSS 服务的捷径，毕竟不是所有 RSS 爱好者都懂代码 😂。RSS Man X 能够有 RSS 订阅管理、RSS 在线阅读界面，进阶功能包括服务健康自检、海外站点 RSS 解锁等，提供不同的版本供选择，三个版本的 `docker-compose` 文件对应不同的需求，包含的组件和服务有差异。

| 组件 / 服务 / 功能名称 | 标准版 | Lite 版 | Ultimate 版 ✨ |
| ---------------------- | ------ | ------- | -------------- |
| TTRSS                  | ✅     | ✅      | ✅             |
| RSSHub                 | ✅     | ✅      | ✅             |
| Huginn                 |        |         | ✅             |
| Mercury                | ✅     |         | ✅             |
| OpenCC                 |        |         | ✅             |
| Redis                  | ✅     |         | ✅             |
| Browserless            |        |         | ✅             |
| 数据持久化保存         | ✅     | ✅      | ✅             |
| 容器自动更新           | ✅     |         | ✅             |
| 容器健康检查           | ✅     |         | ✅             |
| 海外站点加速           |        |         | ✅             |
| 智能路由               |        |         | ✅             |
| 反反爬虫               |        |         | ✅             |

### 2.1 安装流程

安装好了 `docker` 和 `docker-compose` 后，可以使用`git clone https://github.com/Colin-XKL/RSSmanX --depth=1`快速克隆本仓库，也可以通过[这个镜像地址](https://archive.fastgit.org/Colin-XKL/RSSmanX/archive/refs/heads/master.zip)下载仓库 zip 文件，[Gitee 上也有镜像](https://gitee.com/colin-xkl/RSSmanX)不过不经常更新。

1. cd 进入文件夹，修改`.env`中的值，如密码和 TTRSS 入口 URL 等
2. 运行 `sudo docker-compose up -d`
3. 等待程序跑完
4. 安装完成 ✅

### 2.2 相关事宜

1. 访问你设置的 `SELF_URL` 即可看到 Tiny Tiny RSS 的登陆页面，使用默认账户 `admin`，密码 `password` 登陆即可开始使用

2. 如开启海外站点解锁支持，第一次冷启动需要等待 3-5 分钟才能完全启动所有组件。

3. 数据默认保存位置`~/.docker/Database`（注意以执行 docker 命令的用户为准，如使用 root 账户执行，则文件位于 root 用户 home 目录）

4. 默认情况下只有 TTRSS 和 Huginn 可以从外部访问，其他组件互相可以访问但不能直接从外部访问以提高安全性。组件间互相访问可以使用`容器名+指定端口`，端口默认为 80，如`http://rsshub/xxxxx`即可访问到 RSS Man X 内的监听 80 端口的 rsshub 实例。

5. 在 TTRSS 中将原来订阅的 `https://rsshub.app/*` 更改为 `http://rsshub/*` 即可使用 RSS Man X 内的自建 RSSHub 实例，并激活反反爬虫和海外源加速等功能

6. 如无法访问 rsshub 的官方文档站点，可以使用我维护的反代站点[https://rsshub-doc.azure.colinx.one/](https://rsshub-doc.azure.colinx.one/)

7. 关于 ARM 平台的支持可查阅[置顶的 issue](https://github.com/Colin-XKL/RSSmanX/issues/5)，替换部分不支持 arm 架构的 docker 镜像为支持 arm 的镜像即可。

8. RSS Man X 的除 lite 以外的版本默认包含了自托管的 mercury 实例，你只需要在插件配置页面设置 mercury 实例地址为 `service.mercury:3000` 即可，同理，OpenCC 实例地址为`service.opencc:3000`

9. 如果部分 RSS 源不能订阅，检查是否使用了非常规端口。在`.env`文件中设置`RSS_ALLOW_PORTS`

10. 若部署后某个应用一直无法通过浏览器访问，请检查是否绑定到了`6000`/`6666`等特殊端口，浏览器会拦截对这些端口的访问参见[这里](https://blog.colinx.one/posts/docker-compose%E7%9A%84%E9%94%99%E8%AF%AF%E4%BD%BF%E7%94%A8%E5%A7%BF%E5%8A%BF/)

11. `ls`没有显示`.env`文件是因为以点开头的文件在 Linux 中都是默认隐藏的，可以使用`ls -a`查看到

12. vi/vim 编辑文本太麻烦可以尝试使用 nano

13. RSS Man 里所有容器绑定了服务器的/etc/localtime，使用`sudo timedatectl set-timezone Asia/Shanghai` 设定系统的时区为上海后，容器里的时区也可以同步，这样日志里的时间就是东八区了

**获取帮助**

- **For Tiny tiny RSS problems:**
  [https://tt-rss.org/wiki.php](https://tt-rss.org/wiki.php)
  [http://ttrss.henry.wang/](http://ttrss.henry.wang/)
- **For RSSHub problems:**
  [https://docs.rsshub.app/faq.html](https://docs.rsshub.app/faq.html)
- **For Huginn problems:**
  [https://github.com/huginn/huginn#readme](https://github.com/huginn/huginn#readme)

### 2.3 链接

- [我的 RSS 方案与心得](https://blog.colinx.one/posts/%E6%88%91%E7%9A%84rss%E6%96%B9%E6%A1%88%E4%B8%8E%E5%BF%83%E5%BE%97/)
- [RSS Man X GitHub repo](https://github.com/Colin-XKL/RSSmanX)
- [Huginn 指南：为任意网站制作 RSS](https://blog.colinx.one/posts/huginn%E6%8C%87%E5%8D%97%E4%B8%BA%E4%BB%BB%E6%84%8F%E7%BD%91%E7%AB%99%E5%88%B6%E4%BD%9Crss/)
- [docker compose 的错误使用姿势](https://blog.colinx.one/posts/docker-compose%E7%9A%84%E9%94%99%E8%AF%AF%E4%BD%BF%E7%94%A8%E5%A7%BF%E5%8A%BF/)
