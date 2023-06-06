---
title:  Minecraft 上云笔记 - MC 自定义皮肤并支持联机
date: 2021-02-22
lastmod: 2021-02-22
description: Minecraft 上云笔记 - MC 自定义皮肤并支持联机
categories:
- 技术
- 指南
tags:
- Java
- MC
- MC 服务器
- Minecraft
- 我的世界
- 游戏
- 服务器

---

<!-- # Minecraft 上云笔记 - MC 自定义皮肤并支持联机 -->



方案概述：

* 仪表盘 - 红石皮肤站
  https://mcskin.cn/user
* [CSL] 万用皮肤补丁 (CustomSkinLoader) - MC 百科 | 最大的 Minecraft 中文 MOD 百科
  https://www.mcmod.cn/class/883.html
* 全员安装皮肤 mod，即可保证所有人都可以看见对方的皮肤





使用：

1. 安装好 forge

2. 安装好皮肤补丁的 mod

3. 启动一次 MC，让他自动生成配置文件

4. 在皮肤站注册，并建立与游戏中角色名字相同的角色并设定皮肤

5. 修改`".minecraft\CustomSkinLoader\CustomSkinLoader.json"`

   ```json
   {
     "version": "14.12",
     "loadlist": [
       {
         "name": "红石皮肤站",
         "root": "https://mcskin.cn/csl/",
         "type": "CustomSkinAPI"
       },
       {
         "name": "Mojang",
         "type": "MojangAPI"
       },
   ......
   ```

   > **注意**：
   >
   > 自定义皮肤站最好排在 Mojang API 的前面，不然的话如果有正版用户起的角色名字和你的一样，就会默认载入他的皮肤（巨坑 +1）
   >
   > 





