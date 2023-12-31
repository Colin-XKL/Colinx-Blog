---
title:  Win 10 配置 C 语言环境的正确姿势
date: 2020-12-27
lastmod: 2020-12-27
description: VC6.0 太古老，Dev C++没补全不友好，MinGW 安装太烦恼？你用着最新的电脑，最新的系统，却在用着上个世纪的软件开始你人生第一门编程课？你需要这篇指南：在现代化的硬件和平台上使用现代化工具学习 C 语言
categories:
- 技术
- 指南
tags:
- C
- Win10
- 环境配置
---

<!-- # Win 10 配置 C 语言环境的正确姿势 -->

> 本系列教程旨在为刚入门的编程语言学习者做好指南工作，开始编码，本应很简单
>
> The PAINLESS way to start coding!

VC6.0 太古老，Dev C++没补全不友好，MinGW 安装太烦恼？

你用着最新的电脑，最新的系统，却在用着上个世纪的软件开始你人生第一门编程课？

你需要这篇指南：**在现代化的硬件和平台上使用现代化工具学习 C 语言**

*aka*：**Win10 配置 C 语言环境的正确姿势**

>截止 2020 年末，Win10 配置 C 语言环境的常见方案有：
>
>* 使用 scoop 来便捷地安装所需的环境
>* 使用 Winget 来安装所需的环境 *[不成熟]*
>* 使用国内镜像 Cygwin+VSCode 配置 C 语言环境【快速】
>* 使用 WSL+VSCode
>* 使用 WSL+Clion

综合考虑可行性与小白友好性，我们推荐的方案是：



## 使用国内镜像 Cygwin+VSCode 快速配置 C 语言环境

C 语言编写的`.c`源代码文件需要通过编译器编译生成可执行文件`.exe`后才能运行。Windows 系统默认没有自带 C 语言的编译器，这里我们需要手动下载配置才行。

C 语言编译器在 Win 平台下的选择多种多样，但要么配置麻烦，使用门槛高；要么就是下载源在国外，国内下载慢如龟，甚至直接下不成。这里给出基于国内镜像 Cygwin+VSCode 配置 C 语言环境的方案，**实测可用，步骤清晰易懂，国内网络也可在几分钟内配置好！**



### 编译器的下载与配置

Cygwin 的下载安装分为两个部分：

1. Cygwin 安装程序的下载（～1MB）
2. 安装器来完整后续的编译器核心部分的安装（～100MB）

#### Cygwin 的下载安装

Cygwin 的安装程序可以[从其官方站点下载](https://cygwin.com/install.html)。不过站点为全英文且国内访问速度堪忧，这里给出快速下载链接

> Cygwin 安装程序**快速下载**（不限速，免登陆）
>
> https://wws.lanzous.com/iUpUKjojq5i

下载完以后，双击打开，进入下图的界面



<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227164821.png" alt="image-20201224235413884" style="zoom:50%;" />



点击下一步，选择第一项，从网络下载并安装



<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227164829.png" alt="image-20201224235436360" style="zoom:50%;" />



如果让你选择安装路径的话，可以不用改，不过默认在 C 盘。自己改的话，一个尽量避免使用中文路径，还有一个就是要记住你自己自定义的路径，后面会用到。

网络好的话，点击下一步会出来一个可用的国内镜像列表，那样的话随便选一个都可以直接进行后面的步骤。

网络条件不好的情况下，半分钟以内就会报错说网络连接失败。不过不用担心，接下来可以自己填写国内镜像地址。

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227164833.png" alt="image-20201224235644342" style="zoom:50%;" />

在 User URL 的输入框输入`https://mirrors.tuna.tsinghua.edu.cn/cygwin/`并点击 Add 添加。之后点击下一步继续。



<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227164836.png" alt="image-20201224235719679" style="zoom:50%;" />





之后会进入一个选择界面。这里选择要安装的组件。我们只需要 C 语言的编译器，这里在搜索框内输入`gcc`，然后在下方找到`gcc-core`  和   `gcc-g++`，点击右侧的三角形打开下拉菜单，选择 9 开头的版本。如下图所示。

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227164839.png" alt="image-20201225000518139" style="zoom:50%;" />



选择完成后，一路点击下一步安装。如果出现如下的警告信息，直接用默认的设置，点下一步继续就可以了。

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227164842.png" alt="image-20201225000604625" style="zoom:50%;" />



一般一到两分钟之内就可以下载完毕。如果过了很久还没有装完，要么是网络太垃圾，要么就是不小心勾了其他的软件，一直在安装。。。

#### 配置 Cygwin

下面要更改环境变量。如果你之前没有自定义安装目录的话，默认路径`C:\cygwin64\bin`。否则下文对应的地方使用你自定义的目录。

在开始菜单中找到`Windows系统` - 控制面板。如果这里没有的话，按`Windows徽标键`+`S`可以呼出搜索面板，可以在此搜索控制面板菜单项。

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227164856.png" alt="image-20201227152139118" style="zoom:50%;" />

在控制面板主页找到系统与安全。

![image-20201227152347064](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227164859.png)

或者如果你的控制面板打开不是上面的布局而是下面这种布局的话，找到系统菜单。

![image-20201227152420458](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227164903.png)

进入如下的页面，点击高级系统设置。

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227164910.png" alt="image-20201225001127874" style="zoom:50%;" />

切换到高级选项卡，点击下方的环境变量菜单

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227164913.png" alt="image-20201225000925342" style="zoom:50%;" />

在用户变量中，点击`Path`，点击下方编辑按钮进行编辑

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227164916.png" alt="image-20201225000949291" style="zoom:50%;" />

输入 Cygwin 安装路径下的 bin 目录。如果你之前没有自定义安装路径，直接设置如图即可。否则设置为你自定义的路径。

**注意：这一步只要添加这一个就好**，不要看我截图里面的很干净，就把其他的都删了

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227164919.png" alt="image-20201225001045854" style="zoom:50%;" />

一路点击确定。修改完之后重启下电脑确保改动生效。

在 PowerShell 中或者命令提示符中输入 gcc 并回车。如果显示 no input files 则表示安装成功。

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227164921.png" alt="image-20201225001234917" style="zoom:50%;" />

### 配置编辑器

安装完了编译器，可以先来 Hello world 了

![image-20201225001723920](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227164924.png)



按住 Shift 键，右键点击文件夹空白处，会出现在此处打开 Powershell 窗口的选项。

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227164932.png" alt="image-20201225001744160" style="zoom:50%;" />



`gcc hello.c` 命令就会在当前目录下生成一个 exe 可执行文件。如果要指定文件名，可以`gcc hello.c -o hello.exe`

输入`./hello.exe`即可执行该 exe 文件。

![image-20201225002338213](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227164937.png)



如果直接双击 exe 也可以，不过运行窗口会一闪而过，解决方案是在 main 函数末尾，return 语句前，加一句 getchar()。



## 配置 VSCode 的作为 C 语言学习环境

VSCode 是由微软主导开发的一款开源免费、轻巧简单、功能强大的代码编辑器。配合各式各样的插件可以方便地实现各种你想得到和你想不到的功能。

前往[VSCode 官网](https://code.visualstudio.com/)下载 Windows 版 VScode。并按照安装程序的指引进行安装。

直接下载链接

https://code.visualstudio.com/sha/download?build=stable&os=win32-x64-user

> 如果下载速度太慢可以考虑如下方案。
>
> 原下载链接：
>
> https://az764295.vo.msecnd.net/stable/ea3859d4ba2f3e577a159bc91e3074c5d85c0523/VSCodeUserSetup-x64-1.52.1.exe
>
> 将开头的 az764295.vo.msecnd.net 替换掉，如下。
>
> https://vscode.cdn.azure.cn/stable/ea3859d4ba2f3e577a159bc91e3074c5d85c0523/VSCodeUserSetup-x64-1.52.1.exe
>
> 速度直接起飞！

打开安装包进行安装，无脑下一步即可。不顾最后的页面可以考虑全部钩上。



<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227165016.png" alt="image-20201227155805507" style="zoom:50%;" />



<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227165019.png" alt="image-20201227155937465" style="zoom:50%;" />





![image-20201227160036085](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227165022.png)





![image-20201227160129001](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227165026.png)



![image-20201227160209263](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227165029.png)



<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227165032.png" alt="image-20201227160939375" style="zoom:50%;" />



<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227165035.png" alt="image-20201227161003478" style="zoom:50%;" />



在输入栏中，在当前选项卡为**用户**的情况下，输入`run`并按回车进行搜索。修改`Run in Terminal`和 `Save File Before Run`的设置项。

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227165038.png" alt="image-20201227161205762" style="zoom:50%;" />



完成了上述的设置，我们就可以来编写 C 语言的程序了。

### Hello，C！

点击左侧第一个按钮，来到文件管理面板。点击打开文件夹按钮，打开一个空白的文件夹（任意文件夹都可以，只是保存你代码的地方，一般一个干净整洁的新文件夹为宜）

<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227165059.png" alt="image-20201227163528187" style="zoom:50%;" />


打开文件夹后，在空白处单击右键，新建一个文件，文件名输入为`hello.c`


![image-20201227164010460](https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227165107.png)



点击右上角的三角形按钮即可自动编译运行你的 C 语言代码。在窗口下方的终端即可看到输出的`Hello,C!`字样。


<img src="https://blog-1301127393.file.myqcloud.com/BlogImgs/20201227165112.png" alt="image-20201227164041803" style="zoom:50%;" />



### 完成✅

现在，开始你的 C 语言学习之旅吧！
