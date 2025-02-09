---
title: 安全不是特权 - 雷池 WAF 使用体验及部署安装教程  
date: 2023-12-18 
description: 近期刚好关注到雷池 WAF 这个项目比较火，看了下是国内一个安全公司搞得，看 Github Repo 和官方文档像模像样的，虽然文档跟大多数中文项目一样，很多地方写的不够清晰，一些小细节没有跟最新的版本同步，但是总体而言，可用，不难用。值得一试  
categories: 
- 技术
tags:
- 技术
- WAF
- docker
---


自从开始倒腾服务器，部署各种各样服务以来，网站安全一直是那个令我焦虑的一个话题。暴露了那么多服务在公网，总免不了有无聊的人或者别有用心的人在搞事。(当然也有可能不是人类 hhh) 之前也有几次被攻击的历史，比如阿里云控制台一堆 SSH 爆破攻击记录，还有 Wordpress 被攻陷，出现一堆奇怪的文章和评论。虽然损失不是很大，但是处理起来还是有点麻烦的。(被攻击之后学乖了，老老实实定期备份) 况且我在明敌在暗，我们总是处于被动的那个，时间一长，网站的风险和潜在的损失还是不小的，最好是可以有一些防火墙和策略能够拦截点大部分不友好的请求。

很早以前就有调研过世面上的网站防火墙类产品，主要就几种类型：

- 各大云厂商配套的 WAF, 不过收费都很贵
- Cloudflare 等 CDN 服务商提供的服务，不过必须要接入其 CDN 反代才能用。而且大陆地区联通性并不好
- [ModSecurity](https://github.com/SpiderLabs/ModSecurity) 等开源方案，开源免费可自托管，不过配置比较麻烦，而且没有好用的 web 页面

近期刚好关注到雷池 WAF 这个项目比较火，看了下是国内一个安全公司搞得，看 Github Repo 和官方文档像模像样的，虽然文档跟大多数中文项目一样，很多地方写的不够清晰，一些小细节没有跟最新的版本同步，但是**总体而言，可用，不难用。值得一试**

![Pasted image 20231218222249](https://blog-1301127393.file.myqcloud.com/BlogImgs/202312182326408.png)

## 使用体验

谈一谈我为什么选择使用雷池 WAF:

- 免费，开源 (这点存疑，没有详细审计过所有 submoudule, 目前看起来只有部分开源，仓库里大部分都是文档啥的。不过有有安全厂商实名，应该不会做出来一些自毁招牌的操作)
- 支持通过 `docker compose` 自部署
- 功能够用。基本反代站点配置，SSL, 多种规则的黑白名单，频率限制，人机验证，攻击历史看板

目前已经使用了有几个月了，来谈一谈一些个人感受

优点：

- 又要免费，又要能自部署，又要有方便使用的 web 页面，目前这个好像是唯一能选的...
- 支持 `docker compose` 自部署，可以自行编排和自定义很多配置
- TOTP 认证，加分
- UI 简约清晰，加分

缺点：

- 有一些小 bug, 挺影响使用体验的，比如我最开始用的那个版本，那时候还不知道这个只能 TOTP 验证，又找不到地方和文档设置初始账号密码，但是进首页直接跳到了仪表盘，又一直刷报错。这种情况没登录就不应该显示仪表盘界面的，非常误导人，我还以为是强制开了 HTTPS 但是我没有上传 SSL 证书导致的后端请求一直报错。
- 开源了但是又没有完全开。人家企业做这个的本意是推销自家的收费套餐，这个没毛病。但是请不要挂羊头卖狗肉，影响不好。GitHub Repo 都是些文档内容，有个所谓的语义检测引擎倒是放了一点代码，但是这个代码量级太少了，像个玩具项目，不像是正儿八经投入使用的引擎的源码。看了下这个社区版本的文档之类的，从头到尾就没有提过怎么参与项目贡献代码，挂了个 GitHub 链接似乎只是诱导大家去 star, 这样好做宣传。合着这个所谓的社区版就只是部署了之后有个按钮，可以共享攻击记录的 ip?
- `docker compose` 写的方式令人费解。竟然需要在`.env` 文件中自定义一个内网网段，然后用 docker compose 的时候自动创建一个子网，每个容器根据子网的前缀指定了一个 ip, 然后其他容器连接都是在环境变量里写死....明明可以用 compose 文件里定义的 service name 或者 container name 作为 alias 使用内置的 dns 轻松实现的服务发现，为什么搞得如此麻烦？而且有没有考虑过，如果要更新，再执行 compose up 的时候，先前的子网 ip 是被占用的，每次还得先看一下当前用了哪些子网，然后找个没用的，这操作也是没谁了... 你说人家懂吧，他还通过 env 文件自定义 compose 创建的子网前缀，说人家不懂吧，用 container name 做 alias 自动实现服务发现这么简单的方式竟然不用。可能有安全或者性能方面的考量？不过我个人使用还是全部删掉了写死子网前缀的配置，这个使用方法真实太离谱了
- 目前只会记录疑似恶意的请求，正常请求不会记录，没法通过这个来看网站总访问情况了，有点可惜

## 安装部署流程

简单分享下我调优后的安装流程，不使用写死的子网前缀做服务发现。

### 1. 获取 compose 文件

官方文档默认提供的不是通过 `docker compose` 安装，而是他们的一个脚本。可能是为了照顾广大墙内被 CSDN "惯坏"的用户吧，脚本里除了基本的环境检测，主要是提供 docker 的便捷安装，但是怎么说呢...本意是给墙内用户便捷安装，但是为什么安装的操作还是到 docker 官方安装脚本，有没有一种可能，大部分大陆的服务器访问这个站点都很慢甚至连不上呢？但凡这里切一下 docker ce 镜像源都没这么离谱...

回到正题，这里还是大家通过 `docker compose` 安装。没安装 docker 的建议先去安装，有条件的可以直接按照官方文档来，[安装文档链接](https://docs.docker.com/engine/install/ubuntu/). 也可以参考我之前写的 RSSMAN 部署文档中 Docker 安装的部分，有讲解在国内环境下提前更换镜像源以更快完成下载安装。[Docker 安装-RSSMAN 部署指南 - RSS101](https://rss101.colinx.one/deploy/rssman-full-guide/#1-docker)
[RSSMan 部署指南](https://blog.colinx.one/posts/rssmanx%E5%AE%89%E8%A3%85%E9%83%A8%E7%BD%B2%E6%8C%87%E5%8D%97/)

之后获取官方最新的 compose 文件。文档里贴的链接是在离线安装的章节有个 compose 文件链接 https://waf-ce.chaitin.cn/release/latest/compose.yaml

当然也可以直接去 Github 找，在 release 文件夹下。https://github.com/chaitin/SafeLine/blob/main/release/latest/compose.yaml

之后需要按照文档指示，在 compose 文件同文件夹位置，创建一个`.env`文件，并填写相关的变量。这里推荐使用我调整后的版本。后续如果官方对于整个架构有更新，以官方的为准，可以自行根据需要调整 compose 文件。

```yaml
services:
  safeline.manager:
    container_name: safeline-manager
    restart: always
    image: chaitin/safeline-mgt-api:${IMAGE_TAG:?image tag required}
    volumes:
      - ${SAFELINE_DIR?safeline dir required}/resources/management:/resources/management
      - ${SAFELINE_DIR}/resources/nginx:/resources/nginx
      - ${SAFELINE_DIR}/logs:/logs
      - /etc/localtime:/etc/localtime:ro
    ports:
      - ${MGT_PORT:-9443}:1443
    environment:
      - MANAGEMENT_RESOURCES_DIR=/resources/management
      - NGINX_RESOURCES_DIR=/resources/nginx
      - DATABASE_URL=postgres://safeline-ce:${POSTGRES_PASSWORD}@safeline-db/safeline-ce
      - MARIO_URL=http://safeline-mario:3335
      - FVM_MANAGER_URL=safeline.fvm-manager:9004
      - MANAGEMENT_LOGS_DIR=/logs/management
    dns:
      - 119.29.29.29
      - 223.5.5.5
      - 8.8.8.8
    networks:
      - safeline
    cap_drop:
      - net_raw
  safeline.detector:
    container_name: safeline-detector
    restart: always
    image: chaitin/safeline-detector:${IMAGE_TAG}
    volumes:
      - ${SAFELINE_DIR}/resources/detector:/resources/detector
      - ${SAFELINE_DIR}/logs/detector:/logs/detector
      - /etc/localtime:/etc/localtime:ro
    environment:
      - LOG_DIR=/logs/detector
    networks:
      - safeline
    cap_drop:
      - net_raw
  safeline.mario:
    container_name: safeline-mario
    restart: always
    image: chaitin/safeline-mario:${IMAGE_TAG}
    volumes:
      - ${SAFELINE_DIR}/resources/mario:/resources/mario
      - ${SAFELINE_DIR}/logs/mario:/logs/mario
      - /etc/localtime:/etc/localtime:ro
    environment:
      - LOG_DIR=/logs/mario
      - GOGC=100
      - DATABASE_URL=postgres://safeline-ce:${POSTGRES_PASSWORD}@safeline.postgres/safeline-ce
    networks:
      - safeline
    cap_drop:
      - net_raw
  safeline.tengine:
    container_name: safeline-gateway
    restart: always
    image: chaitin/safeline-tengine:${IMAGE_TAG}
    volumes:
      - ${SAFELINE_DIR}/resources/nginx:/etc/nginx
      - ${SAFELINE_DIR}/resources/management:/resources/management
      - ${SAFELINE_DIR}/resources/detector:/resources/detector
      - ${SAFELINE_DIR}/logs/nginx:/var/log/nginx
      - /etc/localtime:/etc/localtime:ro
      # - ${SAFELINE_DIR}/resources/cache:/usr/local/nginx/cache
      - tengine-cache:/usr/local/nginx/cache
    environment:
      - MGT_API=https://safeline.manager:1443/api/publish/server
    ulimits:
      nofile: 131072
    networks:
      - safeline
    ports:
      - 80:80
      - 443:443
  safeline.fvm-manager:
    container_name: safeline-fvm-manager
    restart: always
    image: chaitin/safeline-fvm-manager:${IMAGE_TAG}
    environment:
      - FVM_LOGS_DIR=/logs/management
      - DETECTOR_URL=http://safeline-detector:8001
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ${SAFELINE_DIR}/logs:/logs
    networks:
      - safeline
    cap_drop:
      - net_raw

  safeline.postgres:
    container_name: safeline-db
    restart: always
    image: postgres:15-alpine
    volumes:
      - ${SAFELINE_DIR}/resources/postgres/data:/var/lib/postgresql/data
      - /etc/localtime:/etc/localtime:ro
    environment:
      - POSTGRES_USER=safeline-ce
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD:?postgres password required}
    networks:
      - safeline
    cap_drop:
      - net_raw
    command: [postgres, -c, max_connections=200]

volumes:
  tengine-cache:

networks:
  safeline:
    name: safeline
    driver: bridge
    driver_opts:
      com.docker.network.bridge.name: safeline
```

### 2. 安装部署

部署成功后，第一次使用理论上会跳转到 TOTP 绑定页面。我之前这里卡到 bug, 第一次访问不是首页，没有看到那个绑定 TOTP 的二维码，后面就一直没法跳转到绑定页面了....不知道后面这个 bug 修复了没有，如果大家也遇到这个问题可以考虑全部清理干净，重新安装。

TOTP 软件推荐和对比：

- Microsoft Authenticator( 微软出品，支持云同步，国内可用，微软账户验证有单独的通道和验证方式，速度和体积一般)
- Google Authenticator (谷歌出品，简约好用，体积小速度快，可惜大陆地区没法云同步)
- BitWarden (支持使用自建实例，可惜 app 略卡)

## 总结

雷池 WAF 目前来看是一款够用，不难用的免费 WAF, 建议一试。

此外重要数据还是要经常备份，多副本 + 多地存储，数据无价！
