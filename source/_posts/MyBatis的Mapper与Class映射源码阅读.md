---
title: MyBatis的Mapper与Class映射源码阅读
date: 2022-11-04 14:42:27
tags:
- MyBatis
categories:
- xieajiu
description: "从`MyBatis的配置文件`到与`MapperClass`映射的整个流程"
---

#### MyBatis的配置读取的脑图

{% asset_img MyBatis.png MyBatis读取配置思维导图 %}

[点击下载思维导图](/download/MyBatis的Class与Mapper映射.xmind)

#### MyBatis的版本

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.11</version>
</dependency>
```

#### MyBatis的Mapper配置

```xml
<mappers>
  <package name="com.xie.mapper"/>
</mappers>
```

#### 开始源码调试

##### 1、读取`MyBatis配置`

```java
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

##### 2、创建`SqlSessionFactory`

进行单步调试可以看到：

```java
XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
return build(parser.parse());
```

这个时候创建了`XMLConfigBuilder`，执行`parser.parse()`	创建MyBatis的`Configuration`对象。

##### 3、创建MyBatis的Configuration对象

```java
public Configuration parse() {
  // 判断是否已经解析过
  if (parsed) {
    throw new BuilderException("Each XMLConfigBuilder can only be used once.");
  }
  parsed = true;
  // 读取MyBatis的配置文件，<configuration> MyBatis配置文件的跟标签
  // 解析配置文件中的配置，包括数据源信息、事务、Mapper等
  parseConfiguration(parser.evalNode("/configuration"));
  return configuration;
}
```

> 代码总是那么清晰明了

##### 4、解析MapperClass

```java
private void parseConfiguration(XNode root) {
  try {
    propertiesElement(root.evalNode("properties"));
    // ...
    mapperElement(root.evalNode("mappers"));
  } catch (Exception e) {
    throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
  }
}
```

方法`mapperElement`用来处理`mappers`标签下的字标签`mapper/package`以及属性（`resource`,`class`,`url`），其中可以分为两类，一类是通过Class去匹配对应的XML，一类是通过指定XML读取`namespace`属性找到对应的Class。

> 通过Class去寻找对应的XML，需要XML和Class在同一资源目录下。

且看`mapperElement`方法：

```java
private void mapperElement(XNode parent) throws Exception {
  if (parent != null) {
    // 遍历 mappers 下的字标签
    for (XNode child : parent.getChildren()) {
      if ("package".equals(child.getName())) {
        String mapperPackage = child.getStringAttribute("name");
        configuration.addMappers(mapperPackage);
      } else {
        String resource = child.getStringAttribute("resource");
        String url = child.getStringAttribute("url");
        String mapperClass = child.getStringAttribute("class");
        if (resource != null && url == null && mapperClass == null) {
          try(InputStream inputStream = Resources.getResourceAsStream(resource)) {
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(xxxx);
            mapperParser.parse();
          }
        } else if (resource == null && url != null && mapperClass == null) {
          // 同 resource
        } else if (resource == null && url == null && mapperClass != null) {
          // 同 package，后续会说明的
        } else {
          throw new BuilderException("xxxx");
        }
      }
    }
  }
}
```

> 这里的`package`处理和`resource`殊途同归

##### 5、configuration的addMappers方法

```java
public void addMappers(String packageName, Class<?> superType) {
  ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<>();
  resolverUtil.find(new ResolverUtil.IsA(superType), packageName);
  // 遍历包下的 Class 找到符合要求的MapperClass
  Set<Class<? extends Class<?>>> mapperSet = resolverUtil.getClasses();
  for (Class<?> mapperClass : mapperSet) {
    // 这里的 addMapper 和 resource 的 class 调用的同一个方法
    addMapper(mapperClass);
  }
}
```

##### 6、addMapper方法

```java
public <T> void addMapper(Class<T> type) {
  if (type.isInterface()) {
    if (hasMapper(type)) {
      // 重复的 Class 
      throw new BindingException("");
    }
    boolean loadCompleted = false;
    try {
      // 标记已经解析过的 Class
      knownMappers.put(type, new MapperProxyFactory<>(type));
      MapperAnnotationBuilder parser = new MapperAnnotationBuilder(config, type);
      parser.parse();
      loadCompleted = true;
    } finally {
      if (!loadCompleted) {
        knownMappers.remove(type);
      }
    }
  }
}
```

##### 7、MapperAnnotationBuilder的parse方法

```java
public void parse() {
  String resource = type.toString();
  // 防止重复加载 XML
  if (!configuration.isResourceLoaded(resource)) {
    loadXmlResource();
    configuration.addLoadedResource(resource);
    assistant.setCurrentNamespace(type.getName());
    // ... 
  }
}
```

然后重点看一下`loadXmlResource`方法。

```java
private void loadXmlResource() {
  // 一系列判断
  // 获取 inputStream
  if (inputStream != null) {
    XMLMapperBuilder xmlParser = new XMLMapperBuilder(xxx);
    xmlParser.parse();
  }
}
```

> 这段代码就是最开始处理`mappers`的字标签`mapper`的`resource`属性的代码

剩下的就是通过`XMLMapperBuilder`解析Mapper中的SQL、各种Handle等等

#### 总结

MyBatis使用package配置标签，会先获取到包路径下符合要求的Class，准确的说应该是接口、Mapper接口，通过接口的全路径去寻找MapperClass与之对应的MapperXML，所以在使用Mybatis的时候，有些同学会有MapperClass找不到对应的MapperXML的经历。

当使用`<mapper resource="com/BaseMapper.xml" />`去配置Mapper时，就不需要关注MapperClass和MapperXML是否在同一个文件下。MyBatis会读取Mapper XML的`namespace`属性去找到对应的MapperClass。

> 这里的同一文件夹下，指的是打包之后的Class文件与MapperXml文件在同一文件夹下。所有在开发的时候Class与Mapper是分开的