---
title: Zookeeper Kafka 安装
date: 2017-11-29 15:44:18
categories: 分布式计算
tags:
  - Zookeeper
  - Kafka
---

## Zookeeper 安装

### 下载Zookeeper
官方下载地址[https://zookeeper.apache.org/releases.html](https://zookeeper.apache.org/releases.html)

### 环境搭建
1. 解压下载包，并将解压的目录重命名为zookeeper
2. 将 `zookeeper/conf/zoo_sample.cfg` 拷贝一份命名为 `zoo.cfg`
3. zoo.cfg配置

<!--more-->

``` bash
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=~/zookeeper/data
dataLogDir=~/zookeeper/logs
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

server.1=kafka-server1:2888:3888
server.2=kafka-server2:2888:3888
```
> 配置文件中的`dataDir`和`dataLogDir`最好设置在zookeeper目录下，并在集群中每台主机的这两个目录下写一个`myid`文件，文件内容为集群中每台主机对应的集群编号，编号与配置文件`zoo.cfg`最后两行对应，即kafka-server1这台主机的编号为`1`，kafka-server2`的编号为`2`。


4. 在每台主机上`zookeeper/bin`目录下运行`./zkSever.sh start`启动集群。
5. 在每台主机上`zookeeper/bin`目录下运行`./zkServer.sh status`查看状态。
> 启动成功为看到集群中一台主机为`leader`其他主机为`follower`，如果启动失败，会提示`not running`
6. 每台主机上`zookeeper/bin`目录下运行`./zkServer.sh stop`关闭集群


## kafka 安装

### 下载kafka
官方下载地址[https://kafka.apache.org/](https://kafka.apache.org/)

### 环境搭建
kafka依赖于zookeeper管理，因此安装kafka要先安装zookeeper并确保zookeeper安装成功，同样启动kafka要先启动zookeeper
1. 配置`server.properties`
`server.properties`中修改三个配置: `breker.id`设置为该主机在zookeeper集群中的编号; `listeners`监听该主机的9092端口，具体值在配置文件中有提示; `zookeeper.connect`设置为zookeeper集群，例如`kakfa-server1:2181,kafka-server2:2181`

2. 配置`producer.properties`
`producer.properties`中配置`bootstrap.servers`为`kafka-server1:9092,kafka-server2:9092`

3. 配置`consumer.properties`
`consumer.properties`中配置`zookeeper.connect`为`kakfa-server1:2181,kafka-server2:2181`
4. 每台主机启动kafka
``` bash
bin/kafka-server-start.sh -daemon config/server.properties
```
5. 每台主机关闭kafka
``` bash
bin/kafka-server-stop
```

## zookeeper and kafka
启动集群时，先启动zookeeper，再启动kafka；关闭集群时先关闭kafka，再关闭zookeeper

## 建议
集群是否搭建成功关键在于有没有比较清楚的了解zookeeper和kafka，网上教程很多，照搬网上教程是大忌，要充分理解配置的意义，诸如`kafka-server1``kafka-server2`在集群通信问题，以及部分教程要求修改`/etc/hosts`的目的。没有充分理解配置，配置集群是一条漫长而迷茫的路。如果在linux中配置过一些东西，会有相当多的经验，经验也很重要。