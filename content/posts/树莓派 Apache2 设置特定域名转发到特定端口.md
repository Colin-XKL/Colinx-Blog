---
title: 坑爹的阿里云 & Swap 的重要性
date: 2020-03-10
lastmod: 2020-04-29
description: 随着树莓派上部署的项目越来越多，每次使用 IP + 端口 访问也越来越不方便。考虑使用 Apache 来特定子域名跳转到不同端口对应的不同应用。
categories:
- 技术
tags:
- 服务器
- 树莓派
---


随着树莓派上部署的项目越来越多，每次使用 **IP + 端口**访问也越来越不方便。考虑使用 Apache 来特定子域名跳转到不同端口对应的不同应用。

*   树莓派 3B+  Raspbian 系统
*   Apache2
*   自己的域名

在 Apache2 的配置文件（路径 /etc/Apache2/Apache2.conf ) 文件末尾加上以下配置：

```
<VirtualHost *:80>
  ServerName YOUR_DOMAIN.COM #此处为本条规则绑定的域名，下同
  ServerAlias YOUR_DOMAIN.COM
  ProxyPreserveHost On
  ProxyRequests Off
  ProxyPass / http://localhost:8080/ #修改此处的端口号为实际所需，下同
  ProxyPassReverse / http://localhost:8080/
</VirtualHost>


```

之后开启 Apache2 的相关模块

```
a2enmod rewrite
a2enmod proxy
a2enmod proxy_http


```

最后重启 Apache2 即可 sudo systemctl restart Apache2