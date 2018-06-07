---
title: JVM 小记
date: 2018-03-12 15:18:23
categories: Java
tags:
  - JVM
  - GC
---

## 类加载机制

JVM 的类加载是通过ClassLoader 及其子类完成的，加载分为三类加载
### Bootstrap ClassLoader
负责加载`$JAVA_HOME`中`jre/lib/rt.jar`里所有的class或者 -Xbootclasspath选项指定的jar包，`rt.jar`由C++实现，而不是ClassLoader子类

### Extension ClassLoader
负责加载Java平台中扩展功能的一些jar包，包括`$JAVA_HOME`中`jre/lib/*.jar`或`-Djava.ext.dirs`指定目录下的jar包

### App ClassLoader
负责加载classpath中指定的jar包及目录中的class

### Custom ClassLoader
属于应用程序根据自身需要定义的ClassLoader，如Tomcat、jboss都会根据J2EE规范自行实现ClasLoader。

加载过程中会先检查类是否已加载，检查顺序是自`Custom`-->`Bootstrap`，只要某个Classloader已加载就视为已加载此类，保证此类只加载一次，而加载的顺序是自`Bootstrap`-->`Custom`

### 双亲委托模型
未完待续。。。。