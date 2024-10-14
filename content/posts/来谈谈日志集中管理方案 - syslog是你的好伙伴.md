---
title: 来谈谈日志集中管理方案 - syslog 是你的好伙伴.md
date: 2024-10-13
lastmod: 2024-10-13
description: 身为一个数码宅，家里总是会有越来越多的数码设备，智能化的设备多起来，总是难免会遇到各种各样的问题，排查问题的时候日志就很重要了，对于一些偶发的 case, 比如偶发网络卡顿，能够看到足够完整清晰的日志，就很方便了。
categories:
- 技术
tags:
- Linux
- NAS
- OpenWRT
- Docker
---


身为一个数码宅，家里总是会有越来越多的数码设备，智能化的设备多起来，总是难免会遇到各种各样的问题，排查问题的时候日志就很重要了，对于一些偶发的 case, 比如偶发网络卡顿，能够看到足够完整清晰的日志，就很方便了。

对于一些低功耗设备，比如路由器，其磁盘资源很紧张，日志通常不会保存很久。这个时候要排查一些偶发的持续时间长的小 bug 就很头痛了。好在 openwrt 原生就支持将日志通过 syslog 协议发送到远程服务器。不过看了一圈，发送是发送过去了，但是看日志还得命令行一个个看，要想有个方便的地方看的话就得上 ELK 那一套非常繁琐且吃资源的那一套，我的需求是这样的：

- 免费，可以持续使用
- 能够通过 syslog 协议收集日志
- 能够自部署最佳，但是不希望是 ELK 那种繁琐且非常消耗资源的
- 有个 web 界面能方便的查找日志
- 能够配置监控，那样的话如果设备有 fatal 日志我可以立即知道

搜寻了一番，发现有个 datalust 公司出的 seq 看起来挺不错，在自己服务器上部署了一套，完美符合我的需求，分享一下。

seq 不仅支持 syslog, 还支持程序主动上报、采集容器日志等很多功能，这里只主要围绕 syslog, 收集各种终端设备日志做介绍

![Pasted image 20241013170403](https://blog-1301127393.file.myqcloud.com/BlogImgs/202410132128864.png)

![Pasted image 20241013170340](https://blog-1301127393.file.myqcloud.com/BlogImgs/202410132128865.png)

## 安装：通过 docker compose 部署 seq

首先生成下默认密码，这里需要通过程序自身的指令生成明文密码对应的密文才行。比如我这里指定初始密码的明文为`initP@ss`, 运行这个指令，就会输出对应的密文，这个待会部署的时候需要使用

```bash
➜ echo 'initP@ss' | sudo docker run --rm -i datalust/seq config hash


QE6k2bZLWkh7BwWYRNsG3h9sZPcLskSLKJGii4mvU0rsDyN0/UqW1TwEZp43O09wvOCjbOgswZxHX7FeNo05cfiv3KkB8/q/Msj8nlXL4TGd
```

之后通过 docker compose 进行部署，下面为 compose 文件示例

```yaml
services:
  seq-input-syslog:
    image: datalust/seq-input-syslog:latest
    container_name: seq_syslog_input
    depends_on:
      - seq
    ports:
      - "20014:514/udp" # 接收 syslog 协议请求的端口，这里配置的是 20014, 可以自定义，后面需要用到
    environment:
      SEQ_ADDRESS: "http://seq:5341" # 要转发到主程序进行处理，端口为默认的 5341, 不用改
    restart: unless-stopped

  seq:
    image: datalust/seq:latest
    container_name: seq_log
    ports:
      - "20054:80" # web ui
    environment:
      ACCEPT_EULA: Y
      SEQ_FIRSTRUN_ADMINPASSWORDHASH: QE6k2bZLWkh7BwWYRNsG3h9sZPcLskSLKJGii4mvU0rsDyN0/UqW1TwEZp43O09wvOCjbOgswZxHX7FeNo05cfiv3KkB8/q/Msj8nlXL4TGd # 这里填写刚生成的密文
    restart: unless-stopped
    volumes:
      - /data/seq-data:/data
```

之后执行 sudo docker compose up -d , 顺利的话，在配置的 web ui 端口，就可以看到管理台了。默认账户为 admin, 密码为刚才设置的密码的原始明文。

## 使用：syslog 配置

基本上所有 Linux-base 系统都支持将系统日志通过 syslog 协议发送到远端服务器。我手头主要的 openwrt 路由器、运行 truenas 系统的 nas、树莓派上跑的 debian, 以及其他 linux 云服务器都可以无缝接入，这里简单介绍下

### openwrt 配置远程 log

在 system -> system 菜单下，有个 logging 的 tab, 里面配置好服务器 ip、端口就可以使用了。协议记得选 udp

![Pasted image 20241013165852](https://blog-1301127393.file.myqcloud.com/BlogImgs/202410132128867.png)

### truenas 配置远程 log

在 system setting -> advaned -> syslog 菜单下，配置远程服务器地址和端口，协议选择 udp, 保存就可以了

![Pasted image 20241013171725](https://blog-1301127393.file.myqcloud.com/BlogImgs/202410132128868.png)

### linux 服务器配置远程 log

大多数 Linux 发行版已经预装了 rsyslog。可以通过以下命令检查是否已安装：
`rsyslogd -version`
如果未安装，可以使用以下命令安装：

```bash
# Debian/Ubuntu:
sudo apt update
sudo apt install rsyslog

# CentOS/RHEL:
sudo yum install rsyslog
```

步骤 2：配置客户端的 rsyslog
编辑 rsyslog 配置文件：打开配置文件进行编辑，通常位于 `/etc/rsyslog.conf` 或 `/etc/rsyslog.d/` 目录下。

```bash
sudo nano /etc/rsyslog.conf
```

添加远程服务器配置：在文件末尾，添加以下行以指定远程服务器的 IP 地址和端口（默认是 UDP 514 或 TCP 514）：

```conf
*.* @remote-server-ip:514    # 使用 UDP
*.* @@remote-server-ip:514   # 使用 TCP
```

这里我们需要使用 udp 协议的。
将 remote-server-ip 替换为远程 Syslog 服务器的实际 IP 地址或者域名，保存并关闭文件。

比如

```conf
*.* @syslog.myserver.com:20014
```

步骤 3：重启 rsyslog 服务
在进行更改后，重启 rsyslog 服务以应用新配置：

```bash
sudo systemctl restart rsyslog
```

## 其他配置

### 配置日志定期清理

默认情况下，是没有配置日志清理规则的，时间一长服务器硬盘可能直接满了，所以这里一定不要忘记配置自动清理。在菜单里的 retention 可以配置过多少天删除

![Pasted image 20241013170659](https://blog-1301127393.file.myqcloud.com/BlogImgs/202410132128869.png)

![Pasted image 20241013170654](https://blog-1301127393.file.myqcloud.com/BlogImgs/202410132128870.png)

### 配置告警通知

各种通知渠道的支持是通过插件形式提供的，在 设置里可以自行安装。可以支持 email、http webhook, 钉钉机器人、telegram 机器人等多种渠道。

[插件市场链接](https://www.nuget.org/packages?q=Tags%3A%22seq-app%22)

---
