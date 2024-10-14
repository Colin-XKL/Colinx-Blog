---
title:  Mac 平台配置 C/Python/Java 学习环境
date: 2020-12-18
lastmod: 2020-12-19
description: Mac 平台配置 C/Python/Java 学习环境
categories:
- 技术
- 指南
tags:
- C
- Python
- Java
- Mac
- 环境配置
---





<!-- # Mac 平台配置 C/Python/Java 学习环境 -->

> 本系列教程旨在为刚入门的编程语言学习者做好指南工作，开始编码，本应很简单
>
> The PAINLESS way to start coding!





## 配置 C 语言环境

下面以 clang+VSCode+CodeRunner 为例，搭建一个简单的 C 语言学习环境。

### 检查编译器支持

C 语言的编译需要编译器，一般可以选择 gcc 或是 clang。Mac 系统默认安装了 clang 同时兼容了 gcc 的指令。在终端中进行查看：

![screenshot](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015258.png)

输入`gcc -v`指令测试 gcc 命令是否可用并查看其版本，由上图输出可以看到，该命令可用，版本信息显示的则是 clang 的信息。一般来说，刚学习 C 语言无需关注两者的异同。

### 配置 VSCode

<span id="vscode">VSCode</span>是由微软主导开发的一款开源免费、轻巧简单、功能强大的代码编辑器。配合各式各样的插件可以方便地实现各种你想得到和你想不到的功能。

前往[VSCode 官网](https://code.visualstudio.com/)下载 Mac 版 VScode。并按照安装程序的指引进行安装。

如果下载速度太慢，可以参考[这篇文章](https://zhuanlan.zhihu.com/p/112215618)。

![截屏 2020-12-18 22.20.00](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015416.png)

安装完毕后，打开会看到如下图所示的界面。默认界面为英文，下面对其进行汉化并安装一些必要的插件。

![截屏 2020-12-18 22.21.59](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015546.png)

单击方形图标，打开扩展面板。

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015621.png" alt="image-20201218223234296" style="zoom:50%;" />



搜索`chinese`安装汉化插件。

![image-20201218222747511](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015622.png)

搜索`code runner`安装 Code Runner 插件。并按指示重启应用（Reload 字样）

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015620.png" alt="image-20201218222511011" style="zoom:50%;" />（截图中因为本地已安装故只显示了 Uninstall 卸载按钮。未安装的情况下会显示 Install 按钮可点击安装）

其他插件可根据需要安装。此处推荐安装 C/C++ 插件以实现 C 代码的高亮和补全等功能。

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015624.png" alt="image-20201218223432859" style="zoom:50%;" />

重启后进入应用，界面自动切换到中文。再点击扩展图标，展开扩展列表，在已安装扩展中找到**Code Runner**，点击齿轮图标展开菜单，点击进入扩展设置。<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015627.png" alt="image-20201218224215676" style="zoom:50%;" />



在输入栏中，在当前选项卡为**用户**的情况下，输入`run`并按回车进行搜索。修改`Run in Terminal`和 `Save File Before Run`的设置项。<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015626.png" alt="image-20201218224059150" style="zoom:50%;" />



完成了上述的设置，我们就可以来编写 C 语言的程序了。

### Hello，C！

点击左侧第一个按钮，来到文件管理面板。点击打开文件夹按钮，打开一个空白的文件夹（任意文件夹都可以，只是保存你代码的地方，一般一个干净整洁的新文件夹为宜）

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015623.png" alt="image-20201218230901008" style="zoom:50%;" />

打开文件夹后，在空白处单击右键，新建一个文件，文件名输入为`hello.c`

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015629.png" alt="image-20201218224536077" style="zoom:50%;" />

键入代码。

![image-20201218224343393](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015628.png)

点击右上角的三角形按钮即可自动编译运行你的 C 语言代码。在窗口下方的终端即可看到输出的`Hello,C!`字样。

![image-20201219021624927](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219021631.png)

### 完成✅

现在，开始你的 C 语言学习之旅吧！



## 配置 Python 语言学习环境





### 检查 Python 环境

Mac 系统会自带有 Python 环境。在终端中输入 python 并回车。

![截屏 2020-12-18 22.54.54](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015630.png)

可以看到系统已经安装有 Python，但是显示版本为 2.7。现在的主流版本是 Python3，Python2 与 Python3 的语法并不兼容。

在`>>>`后面键入`exit()`退出 python2 的交互程序。再次在终端输入`python3`

![image-20201218231954777](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015634.png)



此时显示的版本为 3.8.6。

### Hello, Python!

在交互式命令行界面，我们可以直接输入 python 语句并执行。

![截屏 2020-12-18 23.00.54](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219021016.png)

一句一句地输入代码再执行显然太低效了。我们需要一个趁手的代码编辑器。

下面以 VSCode 为例。



**VSCode 的下载和基本配置**请参考上文 C 语言环境配置中 VSCode 的配置环节。

完成基本的设置后，来安装 Python 的专属插件，以实现 Python 代码的高亮、补全等功能。



![image-20201218230442930](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015633.png)



点击左侧第一个按钮，来到文件管理面板。点击打开文件夹按钮，打开一个空白的文件夹（任意文件夹都可以，只是保存你代码的地方，一般一个干净整洁的新文件夹为宜）

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015623.png" alt="image-20201218230901008" style="zoom:50%;" />

打开文件夹后，在空白处单击右键，新建一个文件，文件名输入为`hello.py`

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015629.png" alt="image-20201218224536077" style="zoom:50%;" />

键入代码。

![截屏 2020-12-18 22.58.44](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015631.png)

点击右上角的三角形按钮开始运行我们编写的代码

![截屏 2020-12-18 23.00.54](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015632.png)



可以看到，窗口下方的终端中已经出现了`Hello,Python!`字样。代码运行成功。

### 完成✅

现在，开始你的 Python 语言学习之旅吧！



## 配置 Java 语言学习环境





### 安装 JDK

Mac OS 并没有内置 Java 语言的支持，需要我们另外安装。

直接在终端输入`java`，系统会提示 Java 未安装并跳转到 Oracle 官方的下载页面。在官网下载会强制要求你注册并登陆 Oracle 账户，网站连接缓慢且步骤繁琐。此处我们到清华大学的 TUNA 开源软件镜像站下载相关软件。

首先到 TUNA 开源软件镜像站，来到[AdoptOpenJDK 的下载页面](https://mirrors.tuna.tsinghua.edu.cn/AdoptOpenJDK/)。在列表中找到你需要的版本，点击进入后依次选择处理器架构和平台。

此处以 Java 11 为例，其 Mac OS 下安装程序的下载地址为[https://mirrors.tuna.tsinghua.edu.cn/AdoptOpenJDK/11/jdk/x64/mac/OpenJDK11U-jdk_x64_mac_hotspot_11.0.9.1_1.pkg](https://mirrors.tuna.tsinghua.edu.cn/AdoptOpenJDK/11/jdk/x64/mac/OpenJDK11U-jdk_x64_mac_hotspot_11.0.9.1_1.pkg)。需要其他版本的也可以自行选择。

> 自行选择时请注意：
>
> * jre 只是 java 运行环境，并不包括对 java 代码的编译功能。下载时请认准**jdk**
>
> * 选择处理器架构时，传统 Intel 内核 Mac 请选择**x86**，对于 M1 内核的 Mac，截止本文写作，暂无相关支持，请关注后续更新。
> * 最内层下载资源的列表中，jdk 的资源文件名会有`hotspot`和 `openj9`两种字样。Java 入门请选择`hotspot`，文件后缀名请选择`.pkg`

下载完成后点击打开，出现如下的安装界面。根据指引完成安装。

![image-20201218232011039](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015635.png)



安装完成后，在终端中输入`java -version`并回车确认。看到如下输出说明已经正确安装且 Java 的版本为`11.0.9.1`即 Java 11.

![image-20201218235233365](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015640.png)

下面来为 Java 的学习配置一个趁手的代码编辑器，以 VSCode 为例。



### 为 Java 语言学习配置 VSCode

**VSCode 的下载和基本配置**请参考上文 C 语言环境配置中 VSCode 的配置环节。

完成基本的设置后，来安装 Java 的专属插件，以实现 Java 代码的高亮、补全等功能。

在左侧扩展面板的搜索栏中搜索`java`，选择`Language Support for Java`并安装。

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015641.png" alt="image-20201218235514041" style="zoom:50%;" />

点击左侧第一个按钮，来到文件管理面板。点击打开文件夹按钮，打开一个空白的文件夹（任意文件夹都可以，只是保存你代码的地方，一般一个干净整洁的新文件夹为宜）

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015623.png" alt="image-20201218230901008" style="zoom:50%;" />

打开文件夹后，在空白处单击右键，新建一个文件，文件名输入为`hello.java`

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015629.png" alt="image-20201218224536077" style="zoom:50%;" />

键入代码。

![image-20201218235213233](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015639.png)

可以看到，在窗口下方的终端面板中，已经成功出现了**Hello, Java!**字样，代码运行成功。

![截屏 2020-12-19 00.05.33](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015637.png)

### 完成✅

至此，Mac 平台下的 Java 语言学习环境配置完毕，开始你的 Java 学习之旅吧！





## 常见问题

### 1. Java 扩展安装弹出提示 JDK 版本过低



<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201219015638.png" alt="image-20201218234917588" style="zoom:50%;" />

这是因为插件内置的一些功能需要 Java 来运行，而这些功能又依赖于一些较新的特性，这些特性最早出现在 Java 11 中。为了更好地学习和使用 Java，这里推荐安装 Java 11 或 Java 12。网站教程或学校授课常常以 Java 8 为例，但对于初学者来说，几者无太大区别，Java 8 的代码都可以被很好地支持。

