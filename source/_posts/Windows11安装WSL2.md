---
title: Windows11安装WSL
date: 2026-09-29 09:48:27
tags:
- WSL2
- Linux
categories:
- xieajiu
description: "安装WSL2，以及进行相关配置"
---

### WSL介绍

[**WSL(Windows Subsystem Linux)**](https://learn.microsoft.com/zh-cn/windows/wsl/about)是适用于 Linux 的 Windows 子系统（WSL）是 Windows 的一项功能，可用于在 Windows 计算机上运行 Linux 环境，而无需单独的虚拟机或双重启动。 WSL 旨在为想要同时使用 Windows 和 Linux 的开发人员提供无缝高效的体验。

### 安装WSL2

> 前提条件：必须运行 Windows 10 版本 2004 及更高版本（内部版本 19041 及更高版本）或 Windows 11 才能使用以下命令

```powershell
wsl --install
```

这个命令会安装WLS，目前默认版本就是版本2，如果网络比较慢的话，建议使用以下命令

```powershell
wsl --install --web-download
```

在安装好后需要进行重启，重启完成后，可执行一下命令进行检查

```powershell
 # 检查wsl版本
 wsl -v
```

{% asset_img 01.png WLS版本 %}

### 安装Linux系统

在安装之前可以使用以下命令获取已经安装的Linux版本

```powershell
wsl --list --verbose
```

{% asset_img 03.png 已安装的Linux %}

第一次安装正常是没有的，可以使用以下命令获取可安装的Linux发行版

```powershell
wsl --list --online
```

{% asset_img 04.png 可安装的Linux发行版本 %}

我这边选择Ubuntu-26版本进行安装，使用以下命令

```powershell
wsl --install --distribution Ubuntu-26.04 --location D:\xxx --web-download
```

参数介绍：

- <code>--distribution</code>：指定要安装的 Linux 分发版。 可以通过运行 `wsl --list --online`来查找可用的分发版

- `--web-download`：从联机源安装，而不是使用 Microsoft Store。

- `--location`：指定要将 WSL 分发版安装到哪个文件夹。

  - > 在最初版本无法指定安装目录，只能通过导入导出的方式进行迁移，相关命令请到：https://learn.microsoft.com/zh-cn/windows/wsl/basic-commands

等安装完成后，在命令行会进行Linux用户创建，正常输入用户名和密码即可，登录系统后可以进行下更新

```shell
sudo apt update && sudo apt upgrade
```

### WSL基础配置

{% asset_img 02.png 配置界面 %}

在这里可以进行内存、磁盘和网络配置，我这边建议网络使用镜像模式，这样在Linux系统中启动的端口会被映射到Windows上，方便访问。

当然还可以运行Linux GUI，仅支持Windows11。Windows官网的文档也很清晰https://learn.microsoft.com/zh-cn/windows/wsl/

