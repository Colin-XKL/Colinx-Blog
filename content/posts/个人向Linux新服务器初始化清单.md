---
title: 个人向 Linux 新服务器初始化清单
date: 2023-03-14
lastmod: 2023-03-15T09:14:44.308Z
description: 一份 Linux 初始化清单, 避免每次拿到新的服务器都要一个个去各种地方搜集指令, 以做备忘 & 供有需要的朋友参考. 以目前最新的
  Debian 11 Bullseye 为例
draft: false
categories:
  - 技术
tags:
  - 技术
  - Linux
  - Debian
  - Traefik
  - Headless CMS
---
一份 Linux 初始化清单, 避免每次拿到新的服务器都要一个个去各种地方搜集指令, 以做备忘 & 供有需要的朋友参考.

服务器发行版我个人推荐 Debian 系列, CentOS 系现在已经开始分裂而且说实话对新手其实并不友好. Debian 是在兼容性, 易用性和稳定性之间都取得不错平衡的发行版. 新手推荐 Ubuntu, 不过最近商业化有点过度, 夹带了越来越多的私活, 我个人所有新安装的 Linux 已经全线转向 Debian. 下文以目前最新的 Debian 11 Bullseye 为例

> azure 干净的 debian 11 镜像, 资源使用情况供参考
>
> \~1G Disk, \~100M RAM, ~300 packages

本文仅列举主要事项和操作, 新手可先行阅读这篇文章熟悉概念. [云服务器入门指南](https://blog.colinx.one/posts/%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%85%A5%E9%97%A8%E6%8C%87%E5%8D%97/)
	
正文开始

## 1 - 基础环境搭建

### 1.1 防火墙

检查服务商防火墙和系统自带防火墙(debian 系一般为 `ufw`, cent os 系一般为 `firewall-cmd`).放行 SSH(22), 新的 SSH(自定义), HTTP, HTTPS, 以及其他常用开发端口

### 1.2 新建用户与检查 sudoer

```shell
// -m 创建对应home文件夹
// 添加到wheel用户组自动获得sudo权限

useradd -m --groups wheel colin
```

也可直接编辑`/etc/sudoers` 文件为新用户添加 sudo 权限, 使用 `visudo` 指令可以自动帮你校验, 避免配置写错把系统搞崩

### 1.3 镜像源与基础软件

如为境内服务器需要更换镜像源

#### 更换国内镜像源

一般情况下，将  `/etc/apt/sources.list`  文件中 Debian 默认的源地址  `http://deb.debian.org/`  替换为  `http://mirrors.ustc.edu.cn`  即可。

```shell
//for debian
sudo sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list


//for ubuntu
sudo sed -i 's@//.*archive.ubuntu.com@//mirrors.ustc.edu.cn@g' /etc/apt/sources.list
```

CentOS 系建议启用 EPEL, PowerTools 等 repo 以更方便地安装常用软件和工具. 此处不再赘述

推荐的几个国内镜像站: [清华大学 TUNA 镜像站](https://mirrors.tuna.tsinghua.edu.cn/), [中科大 USTC LUG 镜像站](https://mirrors.ustc.edu.cn/), [腾讯镜像站](https://mirrors.cloud.tencent.com/)

#### 基础软件

```shell
sudo apt update
sudo apt install wget curl git nano
sudo apt install zsh tmux htop duf htop tldr screenfetch tree
```

### 1.4 SSH 安全

修改端口, 配置文件`/etc/ssh/sshd_config`. 重启机器或 sshd 服务后生效

在本机检查`~/.ssh/`有无 id_rsa 等已生成的 key. 如没有再使用 `ssh-keygen` 生成私钥
将本机的公钥上传到远端, 再写入远端的 `authorized_keys` 中

`cat ~/id_rsa.pub >> ~/.ssh/authorized_keys`

注意修改权限, `~/.ssh/authorized_keys` 权限为 600. `~/.ssh/`为 400

如失败可参考这篇文章 debug. <https://superuser.com/questions/1137438/ssh-key-authentication-fails>

### 1.5 设置 hostname

可选, 为了便于识别和后续配置 oh-my-zsh 更美观

```shell
sudo hostnamectl set-hostname my-new-server
sudo hostnamectl status
```

修改后需重启

- - -

### 1.6 SWAP

可选, 建议内存<2G 配置 swap, 大小至少为 2 倍物理内存

推荐用 `fallocate` , 因为这个是最简单、最快速的创建交换空间的方法。 `fallocate`  命令用于为文件预分配块 / 大小。

以下内容参考 linux.cn

使用  `fallocate`  创建交换空间，首先在  `/`  目录下创建一个名为  `swap_space`  的文件。然后分配 2GB 到  `swap_space`  文件：

```shell
sudo fallocate -l 2G /swap_space
ls -lh /swap_space

// 然后更改文件权限，让 `/swap_space` 更安全：
sudo chmod 600 /swap_space

// 创建交换空间
sudo mkswap /swap_space

// 启用
sudo swapon /swap_space

// 使用 s 参数查看列表
sudo swapon -s
```

每次重启后都要重新挂载磁盘分区。因此为了使之持久化，就像上面一样，我们编辑  `/etc/fstab`  并输入下面行：

```
/swap_space swap  swap  sw  0  0
```

保存并退出文件。现在我们的交换分区会一直被挂载了。我们重启后可以在终端运行  `free -m`  来检查交换分区是否生效。

- - -

### 1.7 绑定域名  

可选, 绑定一个域名或者改下本地 host 便于后续访问

### 1.8 添加本地 SSH 别名

配置文件为本机的 `~/.ssh/config`
格式如下

```config
Host serverA
	HostName myserver.domain.xyz
	User colinx
	Port 2333
```

## 2 - 常用工具与配置

### 2.1 Docker 环境

#### Docker 安装

可参考官网文档, 注意是 docker engine 的安装. [Docker 官方安装文档](https://docs.docker.com/engine/install/debian/)

或者我之前写的 RSS MAN 部署文档中 docker 安装的部分
[RSS MAN X 安装部署指南/#1-docker 环境准备](https://blog.colinx.one/posts/rssmanx%E5%AE%89%E8%A3%85%E9%83%A8%E7%BD%B2%E6%8C%87%E5%8D%97/#1-docker%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87)

#### Docker 镜像源配置

文件位置`/etc/docker/daemon.json`, 下面的配置供参考

```json
{
  "registry-mirrors": [
    "https://mirror.ccs.tencentyun.com",
    "http://hub-mirror.c.163.com",
    "https://registry.docker-cn.com",
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
```

修改完保存重启 docker 服务即可生效. 可使用 `docker info` 命令检查是否生效

### 2.2 OH-MY-ZSH

安装OH-MY-ZSH, 在此之前确保已安装 `git` 和 `zsh`

```shell
sh -c "$(curl -fsSL [https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh](https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh))"
```

安装插件

```shell
git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions --depth=1 

git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting --depth=1 
```

按需修改配置. 文件位置`~/.zshrc`, 下面为个人常用配置供参考. 注意去源文件修改对应项, 没有再到末尾加

```shell
# custom theme
ZSH_THEME="af-magic"  // 简洁的主题, 一行, 推荐
ZSH_THEME="gnzh"  // 更丰富的主题, 提示信息一行, 输入命令在另一行

# custom conf override
CASE_SENSITIVE="false"
HYPHEN_INSENSITIVE="false"
plugins=(git z zsh-autosuggestions zsh-syntax-highlighting sudo)
```

日常管理服务器比较多,以及经常复制命令的建议用 gnzh 这个主题.

![](https://user-images.githubusercontent.com/49100982/108254837-882d2a00-716c-11eb-9f49-3b5e6e62eb52.jpg "gnzh theme")

自定义配置, 添加到末尾

```shell
export ZSH_AUTOSUGGEST_BUFFER_MAX_SIZE=20
```

保存并应用

```shell
source ~/.zshrc
```





```shell
// 自定义需求较高的推荐 space ship 这个主题

// 本地安装 nerd font 以正确显示部分图标

brew tap homebrew/cask-fonts 
brew install --cask font-<FONT NAME>-nerd-font

// 推荐字体 fira code pro with nerd fonts. 安装完后更改本机终端字体
brew install --cask font-fira-mono-nerd-font

// 安装zsh 主题 space-ship

git clone https://github.com/spaceship-prompt/spaceship-prompt.git "$ZSH_CUSTOM/themes/spaceship-prompt" --depth=1

ln -s "$ZSH_CUSTOM/themes/spaceship-prompt/spaceship.zsh-theme" "$ZSH_CUSTOM/themes/spaceship.zsh-theme"
// 之后设置主题
ZSH_THEME="spaceship"

// 还有一些自用的推荐配置件文末附录

```

### 2.3 nano 代码文件规则

日常常用文本编辑器为 nano, 轻量级编辑需求完全满足.

```shell
curl https://cdn.jsdelivr.net/gh/scopatz/nanorc/install.sh | sh 
```

### 2.4 时区调整

一般安装完都是 UTC+0, 看日志什么的不方便. 服务器初始化的时候配置好后面可以免去很多麻烦

debian 系可用过 `timedatectl` 命令调整时区. 东八区可以用这个命令

```shell
sudo timedatectl set-timezone Asia/Shanghai
```

### 2.5 厂商监控/SDK 卸载

懂得都懂, 自己搜

## 3 - 进阶内容

### 3.1 内核参数调优

一些内核参数调整, 交换内存阈值和 bbr, tcp fast open 等, 按需启用. 启用前务必确认自己了解对应字段的含义, 否则不如保留系统初始值.

配置文件位置 `/etc/sysctl.conf`

```c
# mem
vm.swappiness = 10

# bbr and network tune 
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
net.ipv4.tcp_fastopen=3
net.ipv4.tcp_slow_start_after_idle=0
```

配置完之后保存并应用

```shell
sudo sysctl -p
```

### 3.2 其他进阶配置

一些其他的系统维护技巧与策略,如

**配置文件管理**
所有应用 docker 化,通过 `docker compose` 文件管理
配置共享存储, `rclone` 挂载 webdav, 同步 docker compose 等配置文件;
traefik 网关作为统一出口, 负责服务发现和自动维护 HTTPS 证书, 自定义配置通过 headless CMS 如 directus 管理, traefik 设定为通过 http 方式获取远端配置即可.

**数据库备份**
所有数据相关的统一挂载到`/data/database/xxx`, 配置定时任务进行备份
以及配置 s3 等, 上传到其他存储介质和其他地域.

这些内容此处不再赘述, 有机会再单独写篇文章分享吧

END

- - -

## 附录

* Install Docker Engine <https://docs.docker.com/engine/install/>
* RSSManX 安装部署指南 <https://blog.colinx.one/posts/rssmanx%E5%AE%89%E8%A3%85%E9%83%A8%E7%BD%B2%E6%8C%87%E5%8D%97/>
* 云服务器入门指南 <https://blog.colinx.one/posts/%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%85%A5%E9%97%A8%E6%8C%87%E5%8D%97/>
* Portainer Install Doc <https://docs.portainer.io/start/install-ce/server/docker/linux>



```shell
# 个人自用配置, 搭配 space ship 主题使用, 建议安装 nerd font 以正确显示图标
# ~/.spaceshiprc.zsh

SPACESHIP_USER_SHOW="always"
SPACESHIP_USER_PREFIX=""
SPACESHIP_USER_COLOR="007"
SPACESHIP_USER_COLOR_ROOT="red"
SPACESHIP_USER_SUFFIX=""


SPACESHIP_HOST_SHOW="always"
#SPACESHIP_HOST_PREFIX="\e[3#1m@"
SPACESHIP_HOST_PREFIX="@"
SPACESHIP_HOST_COLOR="black"
SPACESHIP_HOST_COLOR_SSH="green"


SPACESHIP_DIR_TRUNC="0"
SPACESHIP_DIR_TRUNC_PREFIX=".../"    

SPACESHIP_PROMPT_ORDER=(
  dir            # Current directory section
  git            # Git section (git_branch + git_status)
  hg             # Mercurial section (hg_branch  + hg_status)
  package        # Package version
  node           # Node.js section
  bun            # Bun section
  deno           # Deno section
  ruby           # Ruby section
  python         # Python section
  elm            # Elm section
  elixir         # Elixir section
  xcode          # Xcode section
  swift          # Swift section
  golang         # Go section
  perl           # Perl section
  php            # PHP section
  rust           # Rust section
  haskell        # Haskell Stack section
  scala          # Scala section
  kotlin         # Kotlin section
  java           # Java section
  lua            # Lua section
  dart           # Dart section
  julia          # Julia section
  crystal        # Crystal section
  docker         # Docker section
  docker_compose # Docker section
  aws            # Amazon Web Services section
  gcloud         # Google Cloud Platform section
  azure          # Azure section
  venv           # virtualenv section
  conda          # conda virtualenv section
  dotnet         # .NET section
  ocaml          # OCaml section
  vlang          # V section
  zig            # Zig section
  purescript     # PureScript section
  erlang         # Erlang section
  kubectl        # Kubectl context section
  ansible        # Ansible section
  terraform      # Terraform workspace section
  pulumi         # Pulumi stack section
  ibmcloud       # IBM Cloud section
  nix_shell      # Nix shell
  gnu_screen     # GNU Screen section
  exec_time      # Execution time
  async          # Async jobs indicator
  line_sep       # Line break
  battery        # Battery level and status
  jobs           # Background jobs indicator
  exit_code      # Exit code section
  sudo           # Sudo indicator
  char           # Prompt character
)

SPACESHIP_RPROMPT_ORDER=(
  time           # Time stamps section
  user           # Username section
  host           # Hostname section
)


cdl() {
        cd "$@" && ls;
}

cdll() {
        cd "$@" && ls -lh;
}

```