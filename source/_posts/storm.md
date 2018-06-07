---
title: Storm
date: 2018-01-31 15:18:23
categories: 流式计算
tags:
  - centos
  - storm
  - zookeeper
---

## 安装zookeeper
> 安装过程查看文章[http://fenlan.github.io/2017/11/29/zookeeper-kafka/](http://fenlan.github.io/2017/11/29/zookeeper-kafka/)

## 安装Storm
1. 官网下载Storm
2. 配置`storm/conf/storm.yaml`

``` bash
 storm.zookeeper.servers:
     - "zookeeper-server1"
     - "zookeeper-server2"
     - "zookeeper-server3"
 nimbus.seeds: ["zookeeper-server1"]

 storm.local.dir: "/root/Downloads/storm/data"

 supervisor.slots.ports:
     - 6700
     - 6701
     - 6702
     - 6703
```

<!-- more -->
> 每一项配置顶格需要一个空格，所有配置避免使用Tab

3. 启动`nohup bin/storm nimbus &`
4. 启动`nohup bin/storm supervisor &`
5. 启动`nohup bin/storm ui &`
6. 查看`ui`: zookeeper-server1:8080
7. 运行`WordCount`
 - 下载maven : `wget http://www-eu.apache.org/dist/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz`
 - `tar -zxf apache-maven-3.5.2-bin.tar.gz`
 - 初始化安装storm所需依赖：`mvn clean install -DskipTests=true`
 - 使用Maven打包storm拓扑：`mvn package`
 - 运行`WordCount` : `bin/storm jar storm-starter-1.1.1.jar org.apache.storm.starter.WordCountTopology wordcount`
