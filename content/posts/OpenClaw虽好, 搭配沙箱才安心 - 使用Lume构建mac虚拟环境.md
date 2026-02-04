---
title: OpenClaw 虽好，搭配沙箱才安心 - 使用 Lume 构建 mac 虚拟环境
date: 2026-02-04
description: OpenClaw 这种应用最吸引人、最方便的地方在于 ta 可以成为「代理」, 去执行任何「你可以做」的事。你能做，ta 就能做。不过前提便是宽松的权限管理，超级多的权限。下面介绍一个 基于 lume 的 Mac OS 虚拟机方案，将其作为沙箱环境节点供 openclaw 调用。
categories:
  - 技术
  - 指南
tags:
  - 技术
  - LLM
  - AI
---


![open-claw-img|700x368](https://openclaw.ai/og-image.png)

近日 OpenClaw(原 Clawdbot, 曾用名 Moltbot) 爆火，独立自主操控电脑完成各种任务，一个挺 fancy 的 agent 应用。结合 moltbook 网站的新闻，在各种媒体的鼓吹下故事画风变得愈发科幻，巴不得明天就统治全人类。


爆火没两天，就爆出不少安全新闻，要么是 agent 自作主张直接电脑删空，要么是大量小白用户把实例暴露在公网，别有用心的人可以直接通过 openclaw 的黑洞控制整个电脑。OpenClaw 这种应用最吸引人、最方便的地方在于 ta 可以成为「代理」, 去执行任何「你可以做」的事。你能做，ta 就能做。不过前提便是宽松的权限管理，超级多的权限。

下面介绍一个 基于 lume 的 Mac OS 虚拟机方案，将其作为沙箱环境节点供 openclaw 调用。

## OpenClaw Gateway 安装

由于 gateway 节点负责与各个 channel 即 bot 通信，最好有公网 ip, 这里我在 linux 云服务器上进行部署。

官方默认给的安装命令很简单，`npm install -g openclaw` 就可以了。但是为了安全起见我还是偏好使用 docker 容器。官方对 docker 容器的支持并不是很好，文档也有点混乱，实际测下来从代码库自行构建镜像比较好

首先是克隆仓库。注意默认的 main 分支可能不是很稳定，最好切换到某个 tag 或者 github actions 全部通过的 commit 再构建。

```bash
# 示例: 切换到指定tag
git pull --rebase
git fetch --tags
git checkout v2026.1.29
```

之后准备构建。很多教程包括官方文档是让用户直接在克隆后的 git 仓库里，修改 compose 和 env 文件。但是这样的话后续不方便更新，因为工作区有修改，每次都要 stash 再 pull 还要解决冲突的问题，太麻烦了。建议的方式是在代码库上级目录创建 compose 和 env, 这样下次要更新的话，直接到 repo 里面 git pull 再重新构建镜像就可以了。

目录结构如下：

```
.
├── .env
├── docker-compose.yml
└── **openclaw-repo**
```

compose 文件设置好构建的目录即可。示例如下

```yaml
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE:-openclaw:local}
    container_name: openclaw-gateway
    build: ./openclaw-repo
    environment:
      xxx:xxx
```

准备好目录和 docker compose 之后，即可开始构建

完整 docker compose

```yaml
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE:-openclaw:local}
    container_name: openclaw-gateway
    build: ./openclaw-repo
    environment:
      HOME: /home/node
      TERM: xterm-256color
      OPENCLAW_GATEWAY_BIND: ${OPENCLAW_GATEWAY_BIND}
      OPENCLAW_GATEWAY_PORT: ${OPENCLAW_GATEWAY_PORT}
      XDG_CONFIG_HOME: ${XDG_CONFIG_HOME}
      PATH: /home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      - "${OPENCLAW_GATEWAY_PORT}:${OPENCLAW_GATEWAY_PORT}"
      # - "${OPENCLAW_BRIDGE_PORT:-18790}:18790"
    init: true
    restart: unless-stopped
    command:
      [
        "node",
        "dist/index.mjs",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND:-lan}",
        "--port",
        "${OPENCLAW_GATEWAY_PORT}",
      ]
     
  openclaw-cli:
    image: ${OPENCLAW_IMAGE:-openclaw:local}
    environment:
      HOME: /home/node
      TERM: xterm-256color
      BROWSER: echo
      OPENCLAW_GATEWAY_BIND: ${OPENCLAW_GATEWAY_BIND}
      OPENCLAW_GATEWAY_PORT: ${OPENCLAW_GATEWAY_PORT}
      XDG_CONFIG_HOME: ${XDG_CONFIG_HOME}
      PATH: /home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    stdin_open: true
    tty: true
    init: true
    entrypoint: ["node", "dist/index.mjs"]
    restart: none


```

env 示例

```ini
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=8089

OPENCLAW_CONFIG_DIR=/data/openclaw
OPENCLAW_WORKSPACE_DIR=/data/openclaw/workspace
XDG_CONFIG_HOME=/home/node/.openclaw
```

```bash

# 镜像中的应用默认以uid 1000账户运行, 手动设置data数据目录的权限避免出现预期外的问题
sudo chown -R 1000:1000 /data/openclaw

# 构建镜像
sudo docker compose build

# 第一次使用, 初始化相关配置
docker compose run --rm openclaw-cli onboard

# 之后启动gateway
docker compose up -d openclaw-gateway
```

如果使用 tg channel ,创建完 bot 后需要配对才可以使用。

```
docker compose run --rm openclaw-cli pairing approve telegram <code>
```

日常配置可以直接使用这个 cli 镜像后面加上需要的参数。或者直接到 Gateway 镜像中执行 `node dist/index.mjs xxx` 代替 官方文档里 `openclaw xxx` . (官方 Dockerfile 里面对 cli 工具名字的处理有问题，先曲线救国)

```
docker compose run --rm openclaw-cli xxx
```

## MacOS VM via Lume

接下来配置 mac os 的虚拟机。使用 lume 工具

```bash
brew install lume

brew services start lume
```

之后可以快捷安装 macos vm. 一般情况下，默认安装当前系统大版本下最新的小版本。你可以通过以下方式查询

```bash
# 查询`latest`参数对应默认的系统镜像和版本, 可以看到显示的url指向的系统版本为15.6.1
~ » lume ipsw                                                                         
[2026-01-31T12:23:12Z] INFO: Fetching latest supported IPSW URL

[2026-01-31T12:23:13Z] INFO: Found latest IPSW URL url=https://updates.cdn-apple.com/2025SummerFCS/fullrestores/093-10809/CFD6DD38-DAF0-40DA-854F-31AAD1294C6F/UniversalMac_15.6.1_24G90_Restore.ipsw

https://updates.cdn-apple.com/2025SummerFCS/fullrestores/093-10809/CFD6DD38-DAF0-40DA-854F-31AAD1294C6F/UniversalMac_15.6.1_24G90_Restore.ipsw


# 查询当前系统版本为15.2
~ » sw_vers                                                                           
ProductName: macOS
ProductVersion: 15.2
BuildVersion: 24C101
```

也可以从这里获取指定版本的系统镜像下载地址，手动下载。https://ipsw.me/. 一般 mac os 的镜像大小在 15-20GB

创建 vm

```bash
# 默认安装最新的版本
lume create openclaw --os macos --ipsw latest

# 如果失败报错"Installation requires a software update." 可以下载不高于当前系统运行的版本手动创建

lume create openclaw --os macos --cpu 2 --memory 8GB --disk-size 100GB --ipsw ./<YOUR-DOWNLOAD-IPSW-FILE>

# 启动vm
lume run openclaw

```

之后会输出一个 vnc 地址，可以用 mac 自带的「屏幕共享」app 进行查看。菜单「连接」- 「新建」
![](https://blog-1301127393.cos.ap-shanghai.myqcloud.com/BlogImgs/20260205000147405.png)

完成系统安装的几个必要步骤，然后到设置里把远程控制打开，后面 SSH 要用到。系统更新建议关掉。

![[image-23.png]]

![[image-24.png]]

在 mac vm 上安装 openclaw

```bash
# 基础环境配置
## brew 安装 (加速版本, 使用中科大镜像源)

export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.ustc.edu.cn/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.ustc.edu.cn/homebrew-core.git"
export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles"
export HOMEBREW_API_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles/api"

/bin/bash -c "$(curl -fsSL https://mirrors.ustc.edu.cn/misc/brew-install.sh)"

## brew 镜像配置

echo 'export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles"' >> ~/.zshrc
echo 'export HOMEBREW_API_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles/api"' >> ~/.zshrc
echo 'export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.ustc.edu.cn/homebrew-core.git"' >> ~/.zshrc
echo 'export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.ustc.edu.cn/brew.git"' >> ~/.zshrc
source ~/.zshrc

## 安装基础环境
brew install node
brew install pnpm
pnpm setup
source ~/.zshrc

echo "registry=https://npmreg.proxy.ustclug.org/" >> ~/.npmrc
pnpm install -g openclaw@latest
openclaw onboard --install-daemon
```

client 端配置好 gateway 地址和 token .第一次连接显示需要配对。在服务端执行

```bash

openclaw devices list

openclaw devices apprve <复制的request id>

```

![image-25](https://blog-1301127393.cos.ap-shanghai.myqcloud.com/BlogImgs/20260205000019565.png)

```bash
# 作为node注册

openclaw node install --host <gateway-host> --port 18789 --display-name "Mac mini"

# 然后在服务端approve

# 查看状态
openclaw node status
# 启动节点
openclaw node start
```

之后就可以让 openclaw 调用这个节点啦

![](https://blog-1301127393.cos.ap-shanghai.myqcloud.com/BlogImgs/20260204235959805.png)



THE END
