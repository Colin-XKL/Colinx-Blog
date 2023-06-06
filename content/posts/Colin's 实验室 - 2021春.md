---
title: Colin's 实验室 - 2021 春
date: 2021-02-25
lastmod: 2021-03-30
description: 小玩具及他们的 demo
categories:
- 技术
tags:
- 技术
- Python
- Github
- Flutter
---

<!-- # Colin's 实验室  - 2021 春 -->

> 小玩具及他们的 demo
>
> 

## 英语文本智能查词（Project Sweeper）

> 自动分割给定的英文文本，提取词元后在数据库中搜索，根据给定的词汇量和难度等级进行筛选，只能提取阅读障碍词汇并高亮当前备考项目（如四级）的重点单词

 ![360 截图 20210206210445677](https://blog-1301127393.file.myqcloud.com/BlogImgs/20210330233722.jpg)

## 知乎爬虫及 JS 逆向

> 日常用 RSS 订阅知乎的一些高质量专栏。但由于知乎本事并不支持 RSS，内容大多通过 RSShub 来的，其本质是个爬虫，经常会受到前端页面结构变更的影响。为了应对这个变化，RSShub 中对于知乎相关页面的爬取规则必须得重写。
>
> 分析了一通知乎的页面，发现有个接口可以直接请求，但是会检验 header。header 里面有几项特殊的自定义参数，`x-zse-83`  和 `x-zse-86`等。为了搞清楚这里面的数值怎么来的，就必须对知乎前端页面资源中的一个 js 文件进行逆向分析。
>
> 逆向折腾了大半天，就快要成功提取核心加密函数的时候，发现知乎的前端页面为了 SEO 有加入一页的文章列表数据，可以直接用上。考虑到逆向的工作量和潜在的法律问题，遂放弃原本的方案。
>
> 新的方案完成后，已向 RSShub 提交 PR 并已被合并。



## MC 服务器搭建与极限优化

> 1 核 2G 学生机，从 2 人联机频频崩溃到 8 人挖矿毫无压力，学习了 JVM GC 种种，对服务端程序的一些参数也渐渐熟悉。优化效果喜人，总有还能压榨的性能，就像资本家看手下的骡子一样 hhh

![屏幕截图 (12)](https://blog-1301127393.file.myqcloud.com/BlogImgs20210207205949.png)



## 洞洞板/冰箱贴/CVPRO/（...没想好名字）

> 一款跨平台的灵感素材收集与简易管理 App，基于 Flutter 构建，跨平台，颜值在线。

![截屏 2021-03-30 23.21.32](https://blog-1301127393.file.myqcloud.com/BlogImgs/20210330233350.png)

> 持续开发中，附 roadmap
>
> version 1.0
>
> [x] Text support  
> [x] Key shortcut support  
> [x] About Page  
> [ ] Content editing  
> [x] List item dismiss
> [x] List Sorting
>
> Future - Key functions
>
> [ ] Data Storage  
> [ ] Pic support  
> [ ] Trashbin  
> [ ] File support  
> [ ] Multiple List
>
> Future - Key features
>
> [ ] Grid view  
> [ ] Drag and Drop support  
> [ ] Card resize
>
> Future - Extra
>
> [ ] Text reformat, delete useless spaces etc.  
> [ ] Pic OCR support  
> [ ] WebDav support and automatically upload files<10M  