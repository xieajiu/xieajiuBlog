---
title: 使用Maven工具解决Jar包冲突
date: 2023-09-18 17:05:46
tags:
  - Maven
categories:
  - xieajiu
description: "使用Maven插件解决jar包冲突问题，以及相关介绍"
---

#### 使用场景

一般在Maven项目中遇到的`Jar包`冲突都是不同依赖引入了不同版本的相同依赖，通常的处理方式，通过Maven的`exclusion`标签进行依赖的排除。

当遇到必须需要引入不同版本的相同依赖时，就会导致执行加载一个依赖的类。这种场景比较少见，比如说数据库的驱动包。这个时候就可以使用Maven的插件`Apache Maven Shade Plugin`来解决这个问题。

#### 插件介绍

`Apache Maven Shade Plugin`提供了将依赖打包在 uber-jar 中的功能，还可以遮蔽（即重命名）某些依赖项的包，这里主要使用的它的重命名功能。

> `uber-jar`（有时也称为"Fat JAR"）是一个可执行的Java JAR文件，它包含了一个应用程序的所有依赖项和类文件，使得应用程序可以独立运行，而无需依赖外部的类库或JAR文件。

它还有非常丰富的功能，比如处理`MANIFEST`，`META-INF/services`或者`XML`。

#### 功能实现-Jar包隔离(重命名Jar)

以MySQL驱动包为例，JDBC的默认实现类是`com.mysql.cj.jdbc.Driver`，假设还有一个数据驱动的实现也是这个，它们就冲突了，需要修改包名加以区分。还发现MySQL的驱动依赖发生了变化，它的`groupId`和`artifactId`发生改变。

{% asset_img mysql.png mysql驱动依赖 %}

1、首先在POM文件中添加`Apache Maven Shade Plugin`插件。

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-shade-plugin</artifactId>
  <version>3.5.0</version>
  <executions>
    <execution>
      <phase>package</phase>
      <goals>
        <goal>shade</goal>
      </goals>
      <configuration>
        <relocations>
          <relocation>
            <!-- 包名匹配规则 -->
            <pattern>com.mysql</pattern>
            <!-- 要修改的包名 -->
            <shadedPattern>com.mysql8</shadedPattern>
          </relocation>
        </relocations>
        <transformers>
          <!-- 添加这个是为了将MySQL的依赖打包到Jar中，默认的package是不包含依赖的 -->
          <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
            <!-- Jar的运行入口 -->
            <mainClass>com.xieajiu.Main</mainClass> 
          </transformer>
        </transformers>
      </configuration>
  	</execution>
	</executions>
</plugin>
```

2、执行Maven的package

3、查看下添加该插件前后的对比，左边是修改后的，右边是原来的。我问将mysql的驱动包名进行了修改，然后上传的自己的或者公司的Maven仓库，编写自定义的依赖，然后引用就可以了。

{% asset_img mysql_01.png 打包前后对比 %}
