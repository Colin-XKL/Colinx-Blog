---
title: RSSManX安装部署指南
date: 2022-05-07
description: RSS Man X是我两年前入坑RSS后，为了方便更多人更方便地使用RSS所发起的一个项目，主要是用docker-compose整合了一些常用的服务如TTRSS、RSSHub、Huginn、OpenCC等并进行了一些优化调整，如自动更新，反反爬虫等。这篇文章会尽可能详尽地讲解安装部署的步骤以及安装过程中可能会遇到的一些问题。
categories:
- 技术
- 指南
tags:
- RSS
- Linux
- 教程

---

<!-- # RSSManX安装部署指南 -->

RSS Man X是我两年前入坑RSS后，为了方便更多人更方便地使用RSS所发起的一个项目，主要是用docker-compose整合了一些常用的服务如TTRSS、RSSHub、Huginn、OpenCC等并进行了一些优化调整，如自动更新，反反爬虫等。这篇文章会尽可能详尽地讲解安装部署的步骤以及安装过程中可能会遇到的一些问题。

RSS Man X会利用到docker和docker-compose，首先需要确保你的服务器正确安装了这些软件

## 1. Docker环境准备

检查服务器中是否已经安装了docker和docker-compose，并检查他们的版本是否太老。比如输入`docker --version`来检查docker的版本

docker的版本建议不要低于19，docker-compose的版本建议不要低于1.20

```shell
~ » docker --version
Docker version 20.10.12, build 20.10.12-0ubuntu2~20.04.1
```

```shell
~ » docker-compose --version
docker-compose version 1.25.0, build unknown
```

如果上面命令执行后提示`command not found`，那么说明并没有安装对应的软件包（或是安装的路径并不在当前用户的PATH变量中，尝试切换到root操作，详细的原因和解决方法见后文）

### 1.1 Ubuntu/Debian安装Docker

ubuntu和debian主要是用apt安装和管理软件包，可以使用以下命令来安装，根据实际情况选择是否要加sudo。下面以Ubuntu为例。Debian下的安装略有不同，可以参见[Docker官方文档](https://docs.docker.com/engine/install/debian/)，以及[清华镜像源](https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/)。其中`download.docker.com`连接不同的可以尝试讲域名部分更换为[中科大提供的镜像](https://mirrors.ustc.edu.cn/help/docker-ce.html)地址`mirrors.ustc.edu.cn/docker-ce`

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

如果你的服务器在墙外，可以直接使用docker官方的软件源

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

导入docker软件源后即可开始安装docker，安装完成后可以输入`docker --version`来检查docker的版本

```shell
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

### 1.2 CentOS/Fedora安装Docker

CentOS和Fedora使用yum安装和管理软件包，新版本则是使用dnf，不过两者功能一致且高度兼容。下面以CentOS为例，Fedora可参见[官方文档](https://docs.docker.com/engine/install/fedora/)或[清华镜像源](https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/)

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



### 1.3 安装或更新docker-compose

docker-compose目前有两个主要版本V1和V2，都是可用的。V1使用Python编写，使用时类似`sudo docker-compose up -d`，V2则是Golang编写，与前者高度兼容，但是是作为docker的插件存在的，安装方式不一样。使用命令类似`sudo docker compose up -d`，中间的短杠不需要了。

如果是按照上文的步骤安装的docker，那么默认已经安装了docker compose v2，可以通过`docker compose version`查看版本

```shell
~ » docker compose version
Docker Compose version v2.3.3
```

如果需要是安装的比较老的v1版本的docker-compose，想要单纯更新docker-compose而又不想动其他东西的话，可以使用以下命令进行更新，安装v1版docker-compose的最新版本。

首先删除老旧版本（如果有的话）

```shell
rm $DOCKER_CONFIG/cli-plugins/docker-compose
sudo rm /usr/local/lib/docker/cli-plugins/docker-compose
pip uninstall docker-compose
```

然后使用pip来安装

```shell
pip3 install docker-compose
```

墙内也可以临时使用镜像源下载

```shell
pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple
```

如果pip报错试着升级一下pip的版本

```shell
python3 -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --upgrade pip
```
如果提示pip命令不存在需要先安装一下
```shell
# Ubuntu/Debian
sudo apt install python3-pip

# CentOS/Fedora
sudo yum install python3-pip
```


### 1.4 为Docker配置镜像源

如果Docker下载镜像非常慢，你可能需要单独配置一下Docker的镜像源。==注意：Docker安装软件源的镜像和Docker镜像或者说映像的国内源并不是同一个，英文表述可能更准确一些：==

* 上文安装docker时配置的是docker的repo，repo里含有docker的软件包，[国内的镜像](https://mirrors.ustc.edu.cn/help/docker-ce.html)为Docker CE安装软件包的镜像
* 现在我们要配置的是Docker Hub的Mirror，Docker里拉取Image默认会访问[Docker Hub](https://hub.docker.com)，国内有多个Docker Hub镜像或是Docker镜像加速器，如[中科大的源](https://mirrors.ustc.edu.cn/help/dockerhub.html)

在`/etc/docker/daemon.json`文件中写入以下内容，如不存在可先行创建，注意json中列表的最后一项末尾是没有逗号的

```json
{
  "registry-mirrors": [
    "https://mirror.ccs.tencentyun.com",
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
```

配置完成后`sudo systemctl restart docker`重启docker服务，然后输入`sudo docker info`，在输出结果的末尾可以看到`Registry Mirrors`里会出现我们刚刚配置的Docker Hub镜像



## 2. 安装RSS Man X



本项目旨在为 RSS 的同好提供一个方便地搭建自己的 RSS 服务的捷径。毕竟不是所有 RSS 爱好者都懂代码 😂。RSS Man X能够有 RSS 订阅管理、RSS 在线阅读界面，进阶功能包括服务健康自检、海外站点 RSS 解锁等，提供不同的版本供选择，三个版本的 `docker-compose` 文件对应不同的需求，包含的组件和服务有差异。

| 组件 / 服务 / 功能名称 | 标准版 | Lite 版 | Ultimate 版 ✨ |
| ---------------------- | ------ | ------- | ------------- |
| TTRSS                  | ✅      | ✅       | ✅             |
| RSSHub                 | ✅      | ✅       | ✅             |
| Huginn                 |        |         | ✅             |
| Mercury                | ✅      |         | ✅             |
| OpenCC                 |        |         | ✅             |
| Redis                  | ✅      |         | ✅             |
| Browserless            |        |         | ✅             |
| 数据持久化保存         | ✅      | ✅       | ✅             |
| 容器自动更新           | ✅      |         | ✅             |
| 容器健康检查           | ✅      |         | ✅             |
| 海外站点加速           |        |         | ✅             |
| 智能路由               |        |         | ✅             |
| 反反爬虫               |        |         | ✅             |

### 详细安装流程

安装好了 `docker` 和 `docker-compose` 后，可以使用`git clone https://github.com/Colin-XKL/RSSmanX --depth=1`快速克隆本仓库，也可以通过[这个镜像地址](https://archive.fastgit.org/Colin-XKL/RSSmanX/archive/refs/heads/master.zip)下载仓库zip文件，[Gitee上也有镜像](https://gitee.com/colin-xkl/RSSmanX)不过不经常更新。

1. cd 进入文件夹，修改.env中的值，如密码和TTRSS入口URL等
2. 运行 `sudo docker-compose up -d`
3. 等待程序跑完
4. 安装完成 ✅

#### 相关事宜

1. 访问你设置的 `SELF_URL` 即可看到 Tiny Tiny RSS 的登陆页面，使用默认账户 `admin`，密码 `password` 登陆即可开始使用
2. 如开启海外站点解锁支持，第一次冷启动需要等待 3-5 分钟才能完全启动所有组件。
3. 数据保存位置`/data/docker/`
4. 在 TTRSS 中将原来订阅的 `rsshub.app/*` 更改为 `rsshub/*` 即可使用 RSS Man X内的自建 RSSHub 实例，并激活反反爬虫和海外源加速等功能
5. 关于 ARM 平台的支持可查阅[置顶的 issue](https://github.com/Colin-XKL/RSSmanX/issues/5)
6. 默认情况下只有TTRSS和Huginn可以从外部访问，其他组件互相可以访问但不能直接从内部访问以提高安全性
7. RSS Man X的除 lite 以外的版本默认包含了自托管的 mercury 实例，你只需要在插件配置页面设置 mercury 实例地址为 `service.mercury:3000` 即可，同理，OpenCC实例地址为`service.opencc:3000`



**获取帮助**

* **For Tiny tiny RSS problems:**
  [https://tt-rss.org/wiki.php](https://tt-rss.org/wiki.php)
  [http://ttrss.henry.wang/](http://ttrss.henry.wang/)
* **For RSShub problems:**
  [https://docs.rsshub.app/faq.html](https://docs.rsshub.app/faq.html)
* **For Huginn problems:**
  [https://github.com/huginn/huginn#readme](https://github.com/huginn/huginn#readme)

### 链接

* [我的 RSS 方案与心得](https://blog.colinx.one/posts/%E6%88%91%E7%9A%84rss%E6%96%B9%E6%A1%88%E4%B8%8E%E5%BF%83%E5%BE%97/)
* [RSS Man X GitHub repo](https://github.com/Colin-XKL/RSSmanX)
