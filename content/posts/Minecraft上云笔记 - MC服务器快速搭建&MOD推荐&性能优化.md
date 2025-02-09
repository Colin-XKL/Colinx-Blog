---
title:  Minecraft 上云笔记 - MC 服务器快速搭建&MOD 推荐&性能优化
date: 2021-02-07
lastmod: 2021-02-07
description: Minecraft 上云笔记 - MC 服务器快速搭建&MOD 推荐&性能优化。MC 快速上云，学生机轻松带动，三五好友，畅快联机
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
- JVM
- 优化

---

<!-- # Minecraft 上云笔记 - MC 服务器快速搭建&MOD 推荐&性能优化 -->



> MC 快速上云，学生机轻松带动，三五好友，畅快联机

## 前言&踩坑

距离上一次玩 MC 已有四年之久，上次玩的时候还用的是家里的老爷机，连独显都没有，玩个整合包都费劲。如今设备早已更新换代，B 站不少做 MC 的 UP 主的内容挺有趣，于是尝试重新拾起 MC。

手机端国内只有网易代理，APP 卡顿严重，生态封闭，体验奇差。安卓下载的国际服倒是还不错，不过身为基岩版，与 Java 版的生态并不相通。Github 上找到了可以让基岩版进 Java 版服务器的轮子，不过手机端连接服务器又是一个麻烦事。连接远程服务器又需要登录 Xbox，但是登上了连接的时候又出现奇怪的问题。

贴吧有人给出解决方案，搭建 VPN，形成局域网，那样服务器会被视为局域网内开放的 MC 服务端。但天朝政策收紧，搭建 PPTP、L2TP 等隧道均未果，严重怀疑云服务商干扰了对应服务的端口。几番权衡决定投归 PC 端的 Java 版，生态丰富而且可以充分利用好硬件性能。 

## MC 服务器快速搭建

### MC 服务端选择

物色了很多 MC 服务端，但看了下主要可用且被应用广泛的就两类，

1. Spigot、PaperSpigot 这类加装插件的
2. SpongeForge 这类可以加装 Mod 的

前者生态目测更丰富些，MCBBS 上的开服板块大部分都是 PaperSpigot 搭建的。但是好像都是那种几十上百人的公用服务器，什么权限控制、小游戏、防作弊的插件居多，而我只是想要一个能够装 Mod 的可以供三五好友方便联机的服务器。我的选择是后者

### MC 服务器快速搭建

搭建方案千千万，不过 Docker 来的最实在。Github 找到一个非常方便的镜像，[itzg/minecraft-server](https://github.com/itzg/docker-minecraft-server)。这个镜像是运行时构建的，会根据指定的环境变量下载和安装对应的 MC 服务端程序。支持的服务端有 Bukkit、Spigot、PaperSpigot、SpongeForge 等等，MC 的版本和各项参数均支持通过环境变量自定义，非常方便。而且可以通过更换不同的 Tag 来使用不同的 JDK 版本，从 8 到 15，Hotspot 和 OpenJ9 都有，可以按需选用。有 arm 版本，树莓派也可食用。

安装 Docker 和 docker-compose 后，可以方便地编写脚本来快速启动一个 MC 服务器。这里放上我的`docker-compose.yml`文件

```yaml
version: "3"
# Forge with Sponge API support. THIS REQUIRES DOWNLOADING SPONGEFORGE.
# Place the SpongeForge jar file in /data/mods. Other Forge mods go here as well.
# Place Sponge mods in /data/mods/plugins. Yes, this is a directory inside the Forge mod directory. Do NOT use /data/plugins.

services:
  minecraft:
    image: itzg/minecraft-server
    container_name: "MCServer"
    ports:
      - "25565:25565"
    volumes:
      - "mc:/data" # 主要数据，包括存档数据等
      - "~/.minecraft/forgemods/:/data/mods/" #Forge Mod 位置
      # - "~/.minecraft/sponegemods/:/data/mods/plugins"
      - "~/.minecraft/tmp/:/tmp" # Forge 下载失败可以自己放进去
      - /etc/timezone:/etc/timezone:ro #绑定本机的时区，方便看日志
    environment:
      SERVER_NAME: "MCServer"
      EULA: "TRUE"
      VERSION: "1.12.2" #(Ensure this is compatbile with the version of SpongeForge you are using!)
      TYPE: "FORGE" # 这里以 SpongeForge 的服务端为例，其他服务端也支持
      FORGEVERSION: "RECOMMENDED"
      # FORGEVERSION: "14.23.5.2854"
      CONSOLE: "false"
      ENABLE_RCON: "true"
      RCON_PASSWORD: "mcserver"
      RCON_PORT: 28016
      INIT_MEMORY: 512M
      MAX_MEMORY: 1536M # 内存根据实际需要修改
      OVERRIDE_SERVER_PROPERTIES: "true" 
      SNOOPER_ENABLED: "false" # 统计数据
      VIEW_DISTANCE: 8 #加载区块范围，默认 10，建议 4~8
      SEED: "-505794890" # 初始生成世界的种子
      PVP: "true"
      ONLINE_MODE: "FALSE" #正版校验开关
      ALLOW_FLIGHT: "FALSE" 
      USE_AIKAR_FLAGS: "false" # 一些优化
      RESOURCE_PACK: "https://blog-1301127393.file.myqcloud.com/MC/Distribution/VNR-1.0.1.zip" # 我自己会用的资源包，这里填 url
      NETWORK_COMPRESSION_THRESHOLD: 512 # 网络优化
    restart: unless-stopped
  rcon:
    image: itzg/rcon  # 服务器远程管理面板，具体使用自行百度
    container_name: "RCON"
    depends_on:
      - minecraft
    ports: #这两个端口都需要防火墙放行
      - "3000:4326" # Web UI
      - "4327:4327" # Connection from Web UI
    volumes:
      - "rcon:/opt/rcon-web-admin/db"
      - /etc/timezone:/etc/timezone:ro
    environment:
      RWA_ENV: "TRUE"
      RWA_ADMIN: "TRUE"
      RWA_PASSWORD: "mcadmin"
      RWA_RCON_HOST: "MCServer"
      RWA_RCON_PASSWORD: "mcserver" # RCON server to control
      RWA_RCON_PORT: 28016
    restart: unless-stopped

volumes:
  mc:
  rcon:

```



环境变量可以设置的地方非常多，可以到项目主页的 README 查看。https://github.com/itzg/docker-minecraft-server

`docker-compose up -d` 启动，第一次启动他会下载 MC、下载 MC 服务端、初始化世界。之后就可以在 MC 客户端连接了。（启动完毕就测试一下，测试没问题再进行后面安装 mod、服务器调优（按需）的步骤！）

## MC 客户端安装

### MC 启动器选择

（正版玩家略过，现在貌似正版购买的只支持官方启动器）

对于服务器选择，MCBBS 有一篇讲的挺详细，可以到这里挑选：

> 我的世界 Minecraft Java 版 下载指南 | 文件结构说明 | 推荐启动器 - 软件资源 - Minecraft(我的世界) 中文论坛 -
> https://www.mcbbs.net/forum.php?mod=viewthread&tid=38297&page=1#pid547821

我推荐的启动器：

* **HMCL**（力荐！跨平台、颜值高、功能完备，不过需要先装 Java)
* **BMCL**（次推荐，颜值略低，基于.Net，功能完备）

我不推荐的启动器：

* **Nsiso** （页面倒是做的漂亮，但是我装完 mod 启动竟然启动不了，兼容性或者说功能完备性欠缺，不知道后续版本如何。当时出问题之后我换 HMCL 就可以完美启动）

上一张 HMCL 的主页的图：

![image-20210129172401421](https://blog-1301127393.file.myqcloud.com/BlogImgs20210207205913.png)

其他没有使用过的启动器暂时不予评价。

### MC 本体安装

（正版玩家略过、整合包玩家略过）

推荐方式为启动器内安装，还可以顺带把 Forge 和 OptiFine 也安装了。

* Forge：安装第三方 Mod 需要
* OptiFine：游戏显示调优，有些光影会需要。

版本选择的话，我的选择是 1.12.2。目前 Mod 支持比较完备，生态也很丰富。很多 Mod 最高只支持 1.12.2，目测是后面官方更新了 API 啥的。当然倾向于原版的同学可以选择更新的 1.16，后面的版本有很多有意思的更新。

> **踩坑笔记**：
>
> 1. 这些启动器大多带有三个下载源：官方服务器、MCBBS 镜像和 BMCL 镜像。不过我当时安装的时候 BMCL 的镜像抽风了，下载一直没进度。建议选择 MCBBS 的源（虽然可能部分内容更新没有很勤）
> 2. 通过启动器自动安装 OptiFine 的时候一直报错，换版本也是一样。后来的解决方案是选择窗口上方的【从本地文件安装/升级】
>
> 

## Mod 推荐

针对我实际使用和测试的情况做一些推荐。下文所有 Mod 均在 1.12.2 下测试并实际使用过，兼容性良好。

### 功能增强型 Mod（客户端必装）

这部分 Mod 主要是部分音效和部分不涉及多人游戏的功能的增强，可以随意安装，装不装都不影响好友联机与进服务器。

| Mod 名称              | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| DynamicSurroundings  | 动态环境音效，比如草地有蟋蟀的声音，夜晚有狼的声音，使游戏很有沉浸感 |
| SoundFilters         | 环绕音效，使得 MC 里所有的声音更真实，比如矿洞里的脚步声会有回音，房子里面听外面的声音会更小等等 |
| BetterHUD            | 加强版的 HUD 信息显示，比如方位信息、坐标信息、时间信息、人物护甲、健康状态信息等等很多功能，而且可自定义程度也很高。 |
| JEI                  | 可以方便地查看物品的合成表，一般整合包必备的插件             |
| JustEnoughCharacters | JEI 的扩展，支持按拼音、简拼搜索物品                          |
| Journeymap           | 旅行地图，貌似已经停更，但是功能上没有什么欠缺。支持小地图显示、快速传送等。也是一般整合包必备的插件 |
| InventoryTweaks      | 按 R 键整理背包                                                |

### 功能增强型 Mod（服务端&客户端都必装）

| Mod 名称          | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| FTB-Ultimine     | 超级挖掘，挖矿、清理场地好帮手                               |
| Doubledoors      | 双开门可以一键开关，不用挨个点击开关啦                       |
| OpenGlider       | 滑翔翼，经济实用的交通工具                                   |
| Quark            | 对原版 MC 的很多地方进行了一些加强，我最喜欢的是他增加了纸墙和纸灯笼这类物品，可以建造一些日系的建筑（文末有图） |
| KeepingInventory | 死亡不掉落，不解释，小白福音                                 |



### 内容增强型 Mod（服务端&客户端都必装）



| Mod 名称                  | 说明                                                         |
| ------------------------ | ------------------------------------------------------------ |
| Natura                   | 添加了很多有意思的植物，整个世界变得生机勃勃，可以种樱花树，初期没食物还可以野外采蓝莓吃 |
| MrCrayfish`s Vehicle Mod | 载具 Mod，可以在 MC 开车哦                                      |
| Animania                 | 动物谷，更多的小动物、更加真实的动物喂养与繁殖               |
| Twilightforest           | 鼎鼎有名的暮色森林，不解释                                   |



### 谨慎选用的 Mod

| Mod 名称  | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| LootBags | 击杀怪物得到战利品带，随机开出各种物品。不过你的箱子很快就会被一堆奇奇怪怪的东西堆满（比如我有一大箱无用的钻石马铠） |
| Cuisine  | 中文名烹饪工艺，国人作者开发的 Mod，挺有趣，很好地还原了做菜的过程，做完之后的菜会获得药水一样的效果。不过目测已经停更，而且没有很好的教程 |

**注意**：上面列举的所有 Mod 都是 Mod 本体，有些 Mod 的功能还依赖于其他 Mod，在安装时必须同时安装 Mod 本体和他依赖的 Mod。具体的依赖关系可以参考 mcmod.cn 上对 Mod 介绍页的 Mod 关系选项卡，CurseForge 的 Mod 介绍页的 Relation 选项卡。



> **踩坑笔记**
>
> 1. R 键整理的快捷键与 JEI 的部分功能冲突，可自行修改快捷键
> 2. R 键整理对烹饪工艺中生产出来的菜品没辙
> 3. 推荐仅将 FTB-Ultimine 用于挖泥巴和挖沙，实际使用时可能会出现挖了矿不掉东西的情况┭┮﹏┭┮
> 4. 不要用铁镐、钻石镐开 FTB-Ultimine 挖东西！耐久掉的飞快
> 5. DynamicSurroundings 可以在设置里按需屏蔽特定的声音，比如装上后发现我家所在的地形会有乌鸦叫，就很烦。屏蔽之后整个世界都好起来了
> 6. DynamicSurroundings 与 Animania 有 bug，对 Animania 的某些特有生物进行交互时会触发找不到特定媒体资源的 bug，导致游戏崩溃，重新进入又会触发这个 bug，只有在进去的一瞬间用指令将玩家杀死才能勉强逃离这个无尽的循环（血泪的教训）（最新版本已修复）
> 7. Animania 会对每个维度的世界进行注入，比较耗费资源。在同时安装虚无世界 AoA 的情况下，每次服务器启动 Animania 需要对其包含的二十多的世界维度进行注入，严重拖慢启动速度
> 8. Quark 模组对 1.12 的 MC 只有 1.6 版本而没有后续版本的内容更新和 bug 修复。实际使用中有点吃性能



## MC 服务器优化



> 我使用的是 SpongeForge，因此只对一些通用的优化手段和原版服务器的一些参数做介绍。使用 Spigot 等的可以参考 MCBBS 的帖子进行更加深入的调优：
>
> Minecraft 服务器优化教程 —— 让多带 50% 的玩家不再是梦 - 联机教程 - Minecraft(我的世界) 中文论坛
> https://www.mcbbs.net/forum.php?mod=viewthread&tid=478126

对于一台长期运行的 MC 服务器，我们需要关注的主要是下面几点

### 1. 稳定性

要保证服务的稳定，我们需要防范的是**内在的崩溃**和**外在的影响**。

**防止服务端程序自己崩溃**的方法有：

* 尽量使用稳定版的 MC 和稳定版的 Forge
* 避免使用评分较低的 Mod
* 加载大量 Mod 前先在本地做好测试，避免冲突，同时保障所有依赖项已安装好

**避免外在影响致 MC 服务器崩溃**的方法有：

* 为 JVM 设置合适的内存限制，避免因`OutOfMemory`导致 MC 服务端进程被系统杀掉
* 开放的服务器最好限制下权限并封堵一些游戏内的漏洞，避免熊孩子炸服

### 2. 可维护性

这一项的话其实见仁见智，有些人可能只会开个服务器玩几个星期，有些人会开几年。如果只是短期玩的话麻烦点就麻烦点咯，但是要长期玩的话，注意一下服务器的可维护性后面可以减少很多时间和精力。

我的建议有以下几点：

* **使用 Docker**。这样可以方便地进行服务的停启与迁移、升级。
* **使用 Docker-Compose**并保存`docker-coompose.yml`文件，这样你可以方便地更改一些参数而不用每次都输入一串长长的`docker run`指令。
* **经常备份**。数据无价。
* **尽量避免使用已经停止维护的 Mod**。到时候要是出了奇怪的 Bug，你就只能含泪把这个 Mod 所有相关的内容从你的世界剥离。

### 3. 响应及时性

响应及时性也体现在几个方面：

* CPU 运算能力够强，复杂的时间（战斗、爆炸）等能很快地处理过来
* 存储设备 IO 速度快，服务启动、区块加载可以非常迅速
* 网络连接稳定、延迟低，波动小

上面三点可以通过堆硬件来实现。不过有没有不辛苦钱包又能提高游戏体验的方法呢？

自然是有的。我们可以通过修改服务端程序的部分配置来实现。不过本质上是牺牲一小部分游戏体验换取一大部分游戏体验，就看你能忍受的牺牲的部分能有多少咯。

MC 服务器的优化主要聚焦在两个地方

1. MC 服务端程序的参数优化
2. JVM 参数调优

#### MC 服务端优化

**VIEW_DISTANCE: 8**

这一项可以更改加载的**区块**的数量。这个数值越大，就会加载更多的区块，占用的内存和 CPU 就会更多。这个数值过小可能会导致你家的作物不长以及怪物刷不出来，会显著影响游戏体验。减小这一项的数值可以显著减少内存和 CPU 占用。

默认值 10，建议值 4~8。

> **注意**：1 个***区块***并不是指一个方块，而是指一个`16*16`的区域。这个数值对可视区域的影响并不会很大，你可以放心地减小这个数值。这一项的值每减小 1，你的服务器每次就可以少计算​`16*16*256`个方块

**NETWORK_COMPRESSION_THRESHOLD: 512**

这一项的含义是对大于特定大小的数据包进行压缩。压缩会消耗一定量的 CPU 资源，不过可以减缓网络的压力。如果你的网络条件较差，可以减小这一项的数值。

默认值：256

#### JVM 调优

> 首先需要声明的是，我不是 JVM 调参专业户，下文的数据也并不是严格的评测。结论并不严谨，仅仅从一个技术人员的视角出发，结合我实际的使用体验，给出的一些建议，供大家参考。



##### 当我们在讨论 JVM 调优时，我们在谈论什么？

**垃圾回收**（**G**arbage **C**ollection）一直以来都是众多 JVM 调优教程的核心所在。Java 运行于 VM 上，运行过程中不可避免的垃圾回收是最影响应用整体性能的存在。

这里我们不去纠结那些新生代、老年代的划分、各种比率、各种单元的大小，我们只需要明白几点：

1. 不要觉得你比 JVM 的开发人员牛 X
2. 不要觉得网上各路 JVM 调参大神的方法就适用于你

众多顶级公司都在使用 Java，其母公司 Oracle 自然会招聘全球最优秀的一批开发人员去开发，所以**JVM 的大部分设置都是开箱即用的，默认最佳的。**在你不了解某一项参数的含义、不知道更改某一项配置的目的时，最好是不要做改动。

其次，网络上大部分 JVM 的调优教程都是应对“面试造火箭”的，即便是以实际应用为主的 JVM 优化教程，人家那都是几十核几百 G 内存的机器，一秒钟要承担几百万的访问量，他们做优化的出发点、目标都**不一定适用于你。**



##### 明确优化目标

明确了上面两点我们可以开始着手进行优化了。

首先需要明确你开 MC 服务器的机器是什么配置。以我的为例，我使用的是腾讯云的**1 核 2G 的云服务器**，**带宽 2M**，即峰值速度为 256k/s。

其次需要知道你的优化目标。网络上大多数 JVM 的优化教程都是追求更低的响应时间，以便能够承载更多的并发访问。

以我的情况为例：我开这个 MC 服务器的目的，只是希望能够与**不超过 5 个好友**联机，玩带一些暮色森林这样的**Mod**的 MC。

搭建完服务器后，观察了了一段时间后发现，

* 只有在 MC 服务器刚启动的时候 CPU 占用率会达到 100%，空载（没有玩家在线，后台没有高负荷任务）只有 2% 左右，两人联机时 CPU 平均负载在 30% 左右。
* 加装了十几个 Mod 的情况下，空载占用 CPU 资源在 1G 左右，两人联机 1.4G 左右。
* 只有在客户端刚进入的时候会跑满带宽，之后正常游戏时，每台客户端需要占用 10K/s 左右的带宽。
* 联机游戏间或卡顿，但很快恢复
* 查看日志发现空载时时竟然仍会`can't keep up`

![服务器在空载](https://blog-1301127393.file.myqcloud.com/BlogImgs20210207205922.png)



那么以我的情况来看

* CPU 性能富余
* 内存吃紧
* 带宽够用

那么**我的优化目标**就是：利用好 CPU 资源，尽力减少内存占用，以便为其他程序让路，并且为未来可能要添加的新 Mod 留出空间。



##### 优化

JVM 优化的大头是 GC。网上吹的漫天飞的 G1GC 的确有他的可取之处，一些腐竹的测试数据也表明其实际效果优秀。但是在我的案例中，使用 G1GC 是有问题的：

* **G1GC 天生是为了现代互联网应用环境而设计的**，需要开很多个线程协同来完成垃圾回收工作。他适用于多核处理器 + 大内存机器上的高负荷运算。而我的服务器是单核，内存也并不富裕。
* **多线程的 GC 模式跑在单核机器上**时出现的问题就是：单核机器无法并行处理多个任务，其本质上只是在多个任务之间快速切换，来实现各个任务基本“实时”“并行”运行。但是线程之间的切换是有代价的，**性能损耗积少成多**。
* **G1GC 追求在延迟可控的情况下达到更高的吞吐量**。但他的**代价是[更多的内存占用](https://segmentfault.com/q/1010000021658061)。**小内存机器并不适合使用 G1GC。

物色了一番，我最终选择换用 Serial GC 代替原本的 G1GC。

> **Serial（-XX:+UseSerialGC）**
>
> 从名字我们可以看出，这是一个串行收集器。
>
> Serial 收集器是 Java 虚拟机中最基本、历史最悠久的收集器。在 JDK1.3 之前是 Java 虚拟机新生代收集器的唯一选择。目前也是 ClientVM、ServerVM 4 核 4GB 以下机器默认垃圾回收器。Serial 收集器并不是只能使用一个 CPU 进行收集，而是当 JVM 需要进行垃圾回收的时候，需暂停所有的用户线程，直到回收结束。
>



##### 优化前后对比

1. 优化前 MC 服务器的资源占用情况

![360 截图 20210129163301881](https://blog-1301127393.file.myqcloud.com/BlogImgs20210207205932.jpg)



2. SERIAL GC (1 ONLINE)

![image-20210202194034351](https://blog-1301127393.file.myqcloud.com/BlogImgs20210207205936.png)



3. 进一步对 mod 优化后，2 online

![image-20210202221554906](https://blog-1301127393.file.myqcloud.com/BlogImgs20210207205939.png)



##### 其他优化方法

更换 GC 只是众多优化方法中的一种，其他的诸如换用 OpenJ9 虚拟机也可以有不错的效果，可惜的是 Forge 对其的支持并不友好，只得作罢。还有启用指针压缩也可以获得不错的性能提升，不过 JDK6_u23 之后 `UseCompressedOops` 就已经被默认启用了，使用新版本 Java 基本不用关注这方面的优化了。



## 写在最后



晒几张图

我家：

![屏幕截图 (13)](https://blog-1301127393.file.myqcloud.com/BlogImgs20210207205942.png)



![屏幕截图 (16)](https://blog-1301127393.file.myqcloud.com/BlogImgs20210207205946.png)





我安装的光影 Sildurs Vibrant Shaders v1.281 High 效果图

![屏幕截图 (12)](https://blog-1301127393.file.myqcloud.com/BlogImgs20210207205949.png)



Better HUD + 旅行地图效果图

![屏幕截图 (17)](https://blog-1301127393.file.myqcloud.com/BlogImgs20210207205953.png)





## 参考&推荐阅读

* 大家最推荐用哪种联机方式玩 Minecraft？ - 知乎
  https://www.zhihu.com/question/54048310
* View Distance: optimize your Minecraft server
  https://mtxserv.com/article/12535/view_distance_optimize_your_minecraft_server
* Minecraft 服务器优化教程 —— 让多带 50% 的玩家不再是梦 - 联机教程 - Minecraft(我的世界) 中文论坛
  https://www.mcbbs.net/forum.php?mod=viewthread&tid=478126
* itzg/docker-minecraft-server: Docker image that provides a Minecraft Server that will automatically download selected version at startup
  https://github.com/itzg/docker-minecraft-server
* jvm G1 垃圾收集器有什么缺点？ - SegmentFault 思否
  https://segmentfault.com/q/1010000021658061