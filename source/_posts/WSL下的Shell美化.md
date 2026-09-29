---
title: WSL下美化Shell
date: 2026-09-29 13:44:27
tags:
- WSL2
- Linux
- shell
categories:
- xieajiu
description: "通过zsh美化WSL中的Shell"
---

### zsh和oh-my-zsh安装

先检查是否已经安装了<code>git</code>，如果没有就先安装<code>git</code>，也可以<code>zsh</code>和<code>git</code>一起安装

```shell
sudo apt update && sudo apt install zsh git -y
```

{% asset_img zsh_01.png zsh安装 %}

这里选择 0 就好，之后安装Oh My Zsh会对相关配置进行覆盖

然后安装Oh My Zsh

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

然后安装Powerlevel10k

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

安装完成后会进入Powerlevel10k配置界面如下：

{% asset_img zsh_02.png Powerlevel10k配置 %}

按照提示属于即可，中间有些字符会遇到无法显示的字符，这个等下解决。

#### 乱码解决

在WSL中安装的Linux配置Powerlevel10k，字符无法显示是因为Windows Terminal字体不支持，先下载字体

```txt
https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf
https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf
https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf
https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf
```

复制字体链接进行下载，然后再Windows上进行安装，安装完成后在Windows Terminal的设置中设置WSL中Linux的字体为**MesloLGS NF**，如下：

{% asset_img zsh_03.png 字体设置 %}

然后关闭Windows Terminal再重新打开，登录Linux系统，执行下面命令重新配置Powerlevel10k

```shell
p10k configure
```

这个时候就能看到一些之前无法显示的字符如下：

{% asset_img zsh_04.png 特殊字符 %}

### 插件安装

- 命令行自动提示

  - ```shell
    git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
    ```

- 语法高亮

  - ```shell
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
    ```

安装好插件后编辑<code>.zshrc</code>

```shell
# 找到ZSH_THEME="robbyrussell",改成以下配置，这是配置主题
ZSH_THEME="powerlevel10k/powerlevel10k"

# 配置插件
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

最后执行source命令刷新配置文件

```shell
source ~/.zshrc
```

