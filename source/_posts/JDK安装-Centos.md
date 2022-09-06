---
title: JDK安装-Centos
date: 2022-09-06 20:19:27
tags:
- JDK安装
- Centos
categories:
- xieajiu
description: "Centos安装[JDK](https://www.oracle.com/java/technologies/downloads/), 当前选择的版本是8, 选择的是tar进行安装"
---

### Centos安装JDK

#### 1、安装前

i、检查并卸载系统带的JDK

``` shell
rpm -qa | grep java # 查询本地安装的jdk
rpm -e --allmatches --nodeps [查询到的内容] # 卸载jdk
```

ii、查看cpu位数，32或64

``` shell
lscpu

Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                2
On-line CPU(s) list:   0,1
Thread(s) per core:    1
Core(s) per socket:    2
```

其中x86_64就是64位
iii、下载安装包，直接去[Oracle官网](https://www.oracle.com/java/technologies/downloads/)下载相对应的安装包，我这里所下载如下图
{% asset_img 01.png JDK image %}

#### 2、安装

i、自己想办法把安装包上传到Linux中
ii、解压到自己喜欢的目录，就暂时叫`javaDir`
iii、开始配置jdk

``` shell
tar -zxvf 安装包名字 -C javaDir

# 开编辑profile文件，全局编辑 /etc下的profile，或者编辑用户目录下的.bash_profile文件

vim /etc/profile

# 在文件的底部添加下面代码
JAVA_HOME=javaDir
JRE_HOME=javaDir/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
```

> 记得将`javaDir`换成相对应的目录

最后重新加载文件，并验证JDK是否安装成功

``` shell
# 加载文件
source /etc/profile

# 验证JDK是否安装成功
java -version

java version "1.8.0_333"
Java(TM) SE Runtime Environment (build 1.8.0_333-b02)
Java HotSpot(TM) 64-Bit Server VM (build 25.333-b02, mixed mode)
# 有展示相关JDK的版本信息，就说说明JDK已经安装成功了
```
