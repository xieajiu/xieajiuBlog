---
title: 个人Mac软件配置
date: 2024-08-28 22:09:51
tags:
- Mac
categories:
- xieajiu
description: "Java开发环境简单配置"
---
### Mac的相关配置

#### homebrew

[homebrew官网](https://brew.sh/zh-cn/)

通过命令行安装brew

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

检查brew是否安装成功

```shell
brew --version

# 返回版本号则安装成功
Homebrew 4.3.7

#若返回一下提示，说明当前的sh环境找不到brew命令
zsh: command not found: brew
# 可执行下面名称，zsh的配置文件是【.zprofile】
(echo; echo 'eval "$(/opt/homebrew/bin/brew shellenv)"') >> /Users/${用户名}/${配置文件}
```

#### git

使用brew进行安装和管理git

```shell
brew install git
```

检查git是否安装成功

```shell
git --version
```

git的简易配置

```shell
git config --global core.editor emacs # 配置编辑器
git config --global color.ui true # 配置UI
git config --global user.name yourName # 配置用户名
git config --global user.email yourMail # 配置 邮箱
```

#### SDKMAN（软件开发工具包管理器）

[官网](https://sdkman.io/)，一个Java相关的软件管理工具，可以安装JDK，Maven，Groovy等软件的管理工具。还可以快捷切换本地的JDK版本

在 UNIX 上安装 SDKMAN! 轻而易举。它可轻松在 macOS、Linux 和 Windows（使用 WSL）上安装。此外，它还兼容 Bash 和 ZSH shell。

只需启动一个新终端并输入：

```shell
curl -s "https://get.sdkman.io" | bash
```

按照屏幕上的说明完成安装。然后，打开一个新终端或在同一 shell 中运行以下命令：

```shell
source "$HOME/.sdkman/bin/sdkman-init.sh"
```

最后，运行以下代码片段来确认安装成功：

```shell
sdk version
```

您应该看到包含最新脚本和本机版本的输出：

```shell
SDKMAN!
script: 5.18.2
native: 0.4.6
```

相关命令

```shell
# 查询查询Java可安装的版本
sdk list java 
# 安装Java
sdk install java [版本号(不填写默认最新版本)]
# 选择在当前终端中使用给定的版本
sdk use java 版本号
# 选择将给定版本设为默认版本
sdk default java 版本号
```

#### JDK

我这边使用**Eclipse Temurin**的JDK

```shell
sdk install java 11.0.23-tem
```

检测是否安装成功

```shell
java --version
```

#### Maven

```shell
sdk install maven
```

#### Idea插件

```txt
Chinese (Simplified) Language Pack【中文语言包】
GitToolBox
Maven Helper
Rainbow Brackets Lite - Free and OpenSource
MyBatis Log Free
MyBatisX
MapStruct Support
Translationw
```

