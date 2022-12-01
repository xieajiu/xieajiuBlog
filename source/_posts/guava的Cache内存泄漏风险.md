---
title: guava的Cache内存泄漏风险
date: 2022-12-01 11:16:51
tags:
- 遇到的问题
categories:
- xieajiu
description: "google的guave的cache存在引起内存泄漏风险"
---

#### 环境描述

- Linux服务器：Centos7.6  4核8G

- guave版本

  ```xml
  <dependency>
      <groupId>com.google.guava</groupId>
      <artifactId>guava</artifactId>
      <version>27.1.jre</version>
  </dependency>
  ```

- JDK版本：1.8

#### 问题的发现

在进行接口测试时，发现频繁请求接口会导致服务器的CPU居高不下，Java的GC无明显日志信息（`可能抢占不到CPU，日志输出不了`），通过`jmap`获取堆栈信息，然后使用[ha457.jar](/download/ha457.jar)查看，如下图：

{% asset_img 001.png 堆栈分析图 %}

发现`com.google.common.cache.LocalCache$Segment`占了大量内存，GC无法清理，导致频繁GC然后CPU升高。仔细观察是`java.util.concurrent.ConcurrentlinkedQueue$Node`引起的。然后去百度了下，看到这个类存在内存泄漏的风险（[文章地址](https://zhuanlan.zhihu.com/p/328416427)），但是这个文章说的是`remove`方法，观看了google的cache源码发现`ConcurrentlinkedQueue`的实列只调用了`add(),poll(),offer()`方法。百思不得其解，就去了GitHub，看到了关于[`Guava LocalCache recencyQueue is 223M entires dominating 5.3GB of heap`](https://github.com/google/guava/issues/2408)的文章。

#### 最后的解决方案

{% asset_img 002.png 解决办法 %}

人家都说了用[Caffeine](https://github.com/ben-manes/caffeine)，就乖乖用吧。

#### 疑点

> In synthetic tests this is easy to reproduce, but was viewed as not viable in realistic usages to handle beyond a safety threshold (per CLHM in your reference). The long-term was to switch to a [ring buffer](https://github.com/google/guava/issues/2063#issuecomment-107169736), but that kept being put off as not seen as overly critical. Without a real-world error, it was seen as a GC + perf optimization which is less important given Google's impressive infrastructure (e.g. less application cache centric by having fast data stores).

为什么综合测试中很容易复现，但实际中是不可行的？

