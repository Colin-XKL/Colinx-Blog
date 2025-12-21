---
title: Search-as-a-RSS! 把任何搜索查询转换为RSS! FeedCraft 新功能速递
date: 2025-12-22
description: 作为 RSS 5 年忠实用户, 我非常享受 RSS 主动管理信息源给我带来的掌控感. 但是传统 RSS 订阅方式只能基于站点或者频道, 更多时候我其实是想关注某一个特定领域信息, 使用方式局限性很大. 其实最理想的方式是直接把搜索引擎的结果拿来作为信息源. 在 FeedCraft 新版本中, 我新增了一个 Search-as-a-RSS 的功能, 用户只需要使用自然语言指定好要搜索什么, 接下来就可以自动根据搜索结果生成一个 RSS 了.
categories:
  - 技术
  - 指南
tags:
  - 技术
  - LLM
  - AI
---

## 前言

作为 RSS 5 年忠实用户, 我非常享受 RSS 主动管理信息源给我带来的掌控感. 但是传统 RSS 订阅方式只能基于站点或者频道, 更多时候我其实是想关注某一个特定领域信息, 比如 AI 领域世界模型有什么新的新闻, 或者是想订阅一个特定的信息, 比如我关注的 xxx 歌手有没有新的巡演规划.

这些需求通过传统 RSS 方式难以实现, 你只能定向的订阅某个新闻站点, 然后过滤一下关键词. 使用方式局限性很大. 其实最理想的方式是直接把搜索引擎的结果拿来作为信息源. 问题主要是噪声太多:

- 搜索查询一般是按照关键词来的, 不够灵活
- 垃圾内容农场泛滥
- 高质量的信息很多时候是多种语言的网页, 直接阅读会很困难

好在我们有了 AI, 很多问题有了新的解决方法. 在 FeedCraft 新版本中, 我新增了一个 Search-as-a-RSS 的功能, 用户只需要使用自然语言指定好要搜索什么, 接下来就可以自动根据搜索结果生成一个 RSS 了.

接下来简要介绍一下流程:

## FeedCraft 如何通过搜索结果创建 RSS

FeedCraft 本身是一个一站式处理 RSS 的工具, 这里的搜索需要依赖第三方服务. 首发支持的搜索服务是 LiteLLM Proxy(一个 AI 服务的代理转发平台, 开源可自部署, 可以方便对接各种第三方搜索服务比如 Exa, Tavily, Plexirity, Perplexity, Brave 等等)

以 Tavily 为例, 这个平台提供了每月 1000credits 的额度, 可以执行上百次搜索, 轻度使用绰绰有余了. 前往官网注册个账号, 生成一个 api key 即可.

> [Tavily](https://www.tavily.com/) 是一个专为人工智能代理（AI Agents）设计的搜索引擎，旨在优化 AI 在执行任务时的信息检索过程。它不同于传统的面向人类用户的搜索引擎（如 Google 或 Bing），而是专门为 AI 系统“理解”和“查找”所需信息而构建，强调高效、准确和上下文相关的搜索结果。

![image-14](https://blog-1301127393.cos.ap-shanghai.myqcloud.com/BlogImgs/202512220059928.png)

接下来到 LiteLLM 的后台, Tool - Search Tool 里面增加一个 search tool. 这里 search tool name 可以自定义, 先记下来待会在 feed craft 的设置页面需要填入.

![image-15](https://blog-1301127393.cos.ap-shanghai.myqcloud.com/BlogImgs/202512220059929.png)
在 LiteLLM 后台生成一个 API KEY, Key Name 可以随便写主要是备注. 这个生成的 api key 可以用于请求 LLM, 也可以调用刚才配置的搜索工具. 确认生成后, 复制 api key.

![image-16](https://blog-1301127393.cos.ap-shanghai.myqcloud.com/BlogImgs/202512220059930.png)

在 FeedCraft 后台, 设置里面配置搜索服务, 这里 API URL 是你的 LiteLLM 服务加上`/search` 后缀.
例如你的 LiteLLM 服务部署在 `https://my-litellm.example.com`, 那么这里就填写 `https://my-litellm.example.com/search` . 工具填写刚才在 LiteLLM 后台接入 Tavily 的时候填写的 search tool name

![image-17](https://blog-1301127393.cos.ap-shanghai.myqcloud.com/BlogImgs/202512220059931.png)

之后在「搜索转 RSS」页面, 输入你想查询的东西即可. 你可以直接用自然语言描述, 比如「SpaceX 的最新新闻」. 之后点击下一步即可预览搜索结果.

![image-18](https://blog-1301127393.cos.ap-shanghai.myqcloud.com/BlogImgs/202512220059932.png)

你可以按需调整搜索语句. 确认没问题一直下一步, 可以保存为自定义的配方(Custom Recipe) , 就可以生成一个唯一的 RSS 链接, 在你喜欢的任意 RSS 阅读器里面查看啦

![image-19](https://blog-1301127393.cos.ap-shanghai.myqcloud.com/BlogImgs/202512220059934.png)

更进一步, 你可以使用 FeedCraft 的各种 Craft 来做进一步的处理. 比如获取全文, 添加总结、翻译文章、以及调用 AI 使用自然语言对文章进行筛选等等

![image-21](https://blog-1301127393.cos.ap-shanghai.myqcloud.com/BlogImgs/202512220059935.png)

总体功能就是这样啦, 欢迎试用 FeedCraft 和 Star 🌟! Have fun!

https://github.com/Colin-XKL/FeedCraft


