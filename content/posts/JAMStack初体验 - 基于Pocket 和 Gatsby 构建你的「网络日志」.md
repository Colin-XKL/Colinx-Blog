---
title: JAMStack初体验 - 基于 Pocket 和 Gatsby 构建你的「网络日志」
date: 2021-01-09
lastmod: 2021-01-10
description: 折腾了自己的专属RSS信息流，每天都能从那些高质量的信息源中获得不少干货知识，一般就会顺手收藏一下。有一天突然想起自己收藏的那些文章，本身不就是经过二次筛选的高质量文章吗？于是便在构想能不能通过某种方式将这个信息源也公开出来，一方面是记录，另一方面也是间接地展示自己。恰逢遇见JAMStack，最近在国外非常火，国内的阿里和腾讯也在跟进，搞静态托管那一套。经多方物色，最终确定基于Pocket API+Gatsby来构建这样一个自己的「网络日志」。
categories:
- 杂记
tags:
- 技术
- JAMStack
- Pocket
- Gatsby
- Github
---

# JAMStack初体验 - 基于Pocket 和 Gatsby 构建你的「网络日志」

折腾了自己的专属RSS信息流，每天都能从那些高质量的信息源中获得不少干货知识，一般就会顺手收藏一下。有一天突然想起自己收藏的那些文章，本身不就是经过二次筛选的高质量文章吗？于是便在构想能不能通过某种方式将这个信息源也公开出来，一方面是记录，另一方面也是间接地展示自己。恰逢遇见**JAMStack**，最近在国外非常火，国内的阿里和腾讯也在跟进，搞静态托管那一套。经多方物色，最终确定基于`Pocket API`+`Gatsby`来构建这样一个自己的「网络日志」。

> You are what you read.

## 简谈JAMStack

> JAMStack 即基于客户端 **J**avaScript，可重用 **A**PI 和预先构建 **M**arkup 的现代 Web 开发架构。当我们谈论 “堆栈” 时，我们不再谈论操作系统，特定 Web JAMStack 与特定技术无关。这是一种构建网站和应用程序的新方法，可提供更好的性能，更高的安全性，更低的扩展成本以及更好的开发人员体验。

JAMStack是一种新颖的网站架构，与传统的服务端渲染和近些年国内流行的前后端分离等架构不同的是，他的网页是静态的，可以托管在CDN上，内容是动态的，可以方便地进行修改。

**JAMStack几个显著的优势**：

* 页面为静态，TTFT（TimeToFirstByte）的时间可以降到非常低
* 页面为静态，服务器遭受的安全风险要低得多
* 不同于客户端渲染，JAMStack对SEO会非常友好
* 不同于传统纯静态网站，内容可以方便地动态更新

## 为什么选择Gatsby



作为一个网站生成器Site Generator，Gastby的入门门槛其实是较高的，相对于Hexo、Hugo那种简简单单配置下就是一个站点，Gatsby的难度要更高，因为其灵活度更大，所有的配置、页面都是Javascript来配置的，当然你也可以使用Typescript。其基于React，生态丰富且灵活度高，天生`GarphQL`是选择它的主要原因。



## 网站架构及效果



流程：

1. 我在其他地方看到不错的文章，将其收藏到Pocket
2. 定时任务，从`Pocket API`获取文章数据，交给Gatsby生成站点
3. 自动部署，将生成的静态文件部署到CDN
4. [reading.colinx.one](reading.colinx.one)站点主页就会出现我收藏的文章啦



这其中，Gastby充当的是**Site Generator**的角色，而数据来源为我的`Pocket API`，也就是我把Pocket作为**Headless CMS**来管理我的数据，可以方便地进行更新和管理。



## 总结

项目已经开源，地址在https://github.com/Colin-XKL/Colinx-Reading.git。你也可以去申请自己的Pocket API然后部署你自己的站点。

初始尝试JAMStack，感觉对于博客、文档这类的站点会非常友好，国外比较火的像Shopify这种的无头电商也不错，国内的碍于国情应该不大可能了。



## 链接

* Welcome to the Gatsby Way of Building | Gatsby
  https://www.gatsbyjs.com/docs/
* For fast and secure sites | Jamstack
  https://jamstack.org/