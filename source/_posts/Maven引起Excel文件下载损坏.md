---
title: Maven引起Excel文件下载损坏
date: 2022-09-16 16:43:27
tags:
- 遇到的问题
- Maven
- Excel下载
categories:
- xieajiu
description: "由于项目配置使Maven导致Excel下载后文件损坏解决方法"
---
#### 问题场景

大家在开发中都会有一些`静态模板`下载功能开发，有幸遇到一个问题。下载后的Excel模板打开提示，文件损坏。

如下：

{% asset_img 001.png excel文件损坏示意图 %}

#### 问题定位

网上有好多解决办法，但是对我这个情况并不适用。后来经过排查发现是`maven`执行`package`后生成的模板文件和原文件不一致造成的，如下：

{% asset_img 002.png 文件对比示意图 %}

可以看到`src/main/resource/`下的模板文件大小是`34KB`，经过打包后`target/classes/`下的模板文件变成了`59KB`，所以下载的就是这个错误的模板。

#### 原因分析

上边可知，是在打包的时候损坏了模板文件。重点观察`pom.xml`文件，查看打包执行了哪些动作。

```xml
<resources>
    <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
    </resource>
</resources>
```

就发现了项目对资源文件开启了过滤，对文件进行了读写，由于Excel文件不同于普通的文本文件，导致读写后Excel遭到了破坏。

#### 解决方案

`方法一`：

```xml
<resources>
    <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
        <includes>
            <include>a.properties</include>
        </includes>
    </resource>
    <resource>
        <directory>src/main/resources</directory>
        <excludes>
            <exclude>a.properties</exclude>
        </excludes>
    </resource>
</resources>
```

> 第一个`resource`标签标识开启文件读写，只读写和打包`include`标签中文件
>
> 第二个`resource`标签标识只打包不是`exclude`中的文件，但不进行读写（因为`filtering`默认是`false`）

`方法二`：

这个方法不是通用方法，需要看具体项目的`pom.xml`文件。如下，我遇到的`pom.xml`

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <version>2.6</version>
    <configuration>
        <!-- 二进制文件不做过滤处理 -->
        <nonFilteredFileExtensions>
            <nonFilteredFileExtension>xlsx</nonFilteredFileExtension>
        </nonFilteredFileExtensions>
    </configuration>
</plugin>
```

我这边引用了`maven-resources-plugin`插件，并有`nonFilteredFileExtension`标签可以配置不进行处理的资源文件的后缀名。
相关文档：[Maven的pom.xml中resources标签的用法](https://blog.csdn.net/wenonepiece/article/details/112721380)