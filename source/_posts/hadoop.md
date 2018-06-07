---
title: hadoop之旅
date: 2017-09-22 15:18:23
categories: 分布式计算
tag:
  - hadoop
  - Hbase
  - Zookeeper
---

## hadoop介绍
**以下是 Hadoop 的几种定义，每种定义都针对的是企业内的不同受众：**
- 对于高管：Hadoop 是 Apache 的一个开源软件项目，目的是从令人难以置信的数量/速度/多样性等有关组织的数据中获取价值。使用数据，而不是扔掉大部分数据。
- 对于技术管理人员：一个开源软件套件，挖掘有关您的企业的结构化和非结构化大数据。Hadoop 集成您现有的商业智能生态系统。
- 工程：大规模并行、无共享、基于 Java 的 map-reduce 执行环境。打算使用数百台到数千台计算机处理相同的问题，具有内置的故障恢复能力。Hadoop 生态系统中的项目提供了数据加载、更高层次的语言、自动化的云部署，以及其他功能。
- 安全性：由 Kerberos 保护的软件套件。

<!-- more -->

## hadoop组件
**下图显示了Hadoop生态系统各种组件**
![Hadoop](http://cdn.guru99.com/images/Big_Data/061114_0803_LearnHadoop4.png)

**Apache Hadoop 由两个子项目组成**
1. Hadoop MapReduce : MapReduce 是一种计算模型及软件架构，编写在Hadoop上运行的应用程序。这些MapReduce程序能够对大型集群计算节点并行处理大量的数据。
2. HDFS (Hadoop Distributed File System): HDFS 处理 Hadoop 应用程序的存储部分。 MapReduce应用使用来自HDFS的数据。 HDFS创建数据块的多个副本，并集群分发它们到计算节点。这种分配使得应用可靠和极其迅速的计算。

**如果还是不清楚，再解释一下**
- HDFS：如果您希望有 4000 多台电脑处理您的数据，那么最好将您的数据分发给 4000 多台电脑。HDFS 可以帮助您做到这一点。HDFS 有几个可以移动的部件。Datanodes 存储数据，Namenode 跟踪存储的位置。还有其他部件，但这些已经足以使您开始了。
- MapReduce：这是一个面向 Hadoop 的编程模型。有两个阶段，毫不意外，它们分别被称为 Map 和 Reduce。如果希望给您的朋友留下深刻的印象，那么告诉他们，Map 和 Reduce 阶段之间有一个随机排序。JobTracker 管理您的 MapReduce 作业的 4000 多个组件。TaskTracker 从 JobTracker 接受订单。如果您喜欢 Java，那么用 Java 编写代码。如果您喜欢 SQL 或 Java 以外的其他语言，您的运气仍然不错，您可以使用一个名为 Hadoop Streaming 的实用程序。

**需要根本性理解这两个东西，否则配置的时候跟着教程走会遇到问题**
**附上[参考地址1](https://wizardforcel.gitbooks.io/tutorialspoint-db/hadoop/39.html) [参考地址2](https://www.ibm.com/developerworks/cn/data/library/techarticle/dm-1209hadoopbigdata/index.html)**

## 说说联机

### 首先准备ssh
**为了使用Hadoop的时候免于密码登录其他Slaves，需要对ssh进行设置。**
**生成登录钥匙,分为公钥和私钥**
```
ssh-keygen -t rsa
```
**一直敲回车(如果之前进行过这种操作，会提示是否覆盖之前的内容，输入y回车)，进行完这一步后系统会在`~/.ssh/`目录下生成两个文件，一个是私钥，一个是公钥，需要把公钥放在联机主机上**
```
ssh-copy-id -p port username@remote-server
```

**登录测试一下，通常第一次登录需要密码，之后不需要，如果碰到**
```
connect to host localhost port 22: connection refused
```

**有两种可能，一种是你的ssh服务没有打开通过一下命令打开**
```
sudo service sshd restart
```

**另一种可能是你的ssh—agent和ssh-server等设置不一致，需要将`/etc/ssh/ssh_config`和`/etc/ssh/sshd_config`两个文件里面的端口设置相同**

**如果需要这种情况**
```
sign_and_send_pubkey: signing failed: agent refused operation
```

**需要敲`ssh-add`就解决问题**

### 联机
**联机设置主要在`hadoop/etc/hadoop/masters`和`hadoop/etc/hadoop/slaves`里面,在所有机子上设置`masters`为主机ip地址,而只在主机上添加其他所有联机设备的ip地址(2.x版本不需要masters文件，因此不必配置masters文件)**
> 说明：slaves里面同样可以写主机hostname，如果写hostname, 需要配合`/etc/hosts`一起配置

![masters and slaves](/images/hadoop-master-slaves-conf.png)

**然后在主机上启动hadoop,并通过`localhost:50070`查看Hadoop信息和连接情况**

**如果在网页上没有出现我们的datanode，可能是需要进入没有出现的slaves，找到之前`hdfs.site.xml`配置的目录，将其目录下的所有文件删掉就ok**

## Hadoop下载安装

### 进入`Hadoop`官网找到最新的稳定版下载
![下载界面](/images/hadoopdownloads.png)

**下载完成后将压缩包解压并重命名为`hadoop`(为了方便)**

### 配置
**编辑`hadoop/etc/hadoop/core-site.xml`,指定NameNode的主机名和端口**
``` xml
<configuration>
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://hadoop-master:9000</value>
	</property>
	<property>
		<name>io.file.buffer.size</name>
		<value>131072</value>
	</property>
	<property>
		<name>hadoop.tmp.dir</name>
		<value>file:/home/fenlan/Downloads/hadoop/tmp</value>
	</property>
</configuration>
```

> 需要说明属性`hadoop.tmp.dir`的值是一个你电脑上的目录，为了方便，最好将它设置在`hadoop`目录下。而`fs.defaultFS`的值是主机+端口,`hadoop-master`可以在`/etc/hosts`里面设置，一般本机设置默认为`localhost`，`fs.default.name`的值统一为`masters`下的主机名称.

**编辑`hadoop/etc/hadoop/hdfs-site.xml`,指定HDFS的默认副本数**
``` xml
<configuration>
	<property>
		<name>dfs.namenode.name.dir</name>
		<value>file:/home/fenlan/Downloads/hadoop/hdfs/namenode</value>
	</property>
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>file:/home/fenlan/Downloads/hadoop/hdfs/datanode</value>
	</property>
	<property>
		<name>dfs.namenode.checkpoint.dir</name>
		<value>file:/home/fenlan/Downloads/hadoop/hdfs/namesecondary</value>
	</property>
	<property>
		<name>dfs.replication</name>
		<value>2</value>
	</property>
	<property>
		<name>dfs.block.size</name>
		<value>134217728</value>
	</property>
</configuration>
```

**编辑`hadoop/etc/hadoop/mapred-site.xml`,指定JobTracker的主机名和端口(如果没有，直接复制`mapred-site.xml.template`)**
``` xml
<configuration>
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
	<property>
		<name>mapreduce.jobhistory.address</name>
		<value>hadoop-master:10020</value>
	</property>
	<property>
		<name>mapreduce.jobhistory.webapp.address</name>
		<value>hadoop-master:19888</value>
	</property>
	<!--
	<property>
		<name>yarn.app.mapreduce.am.staging-dir</name>
		<value>file:/home/fenlan/Downloads/hadoop/app</value>
	</property>
	-->
</configuration>
```

**编辑`hadoop/etc/hadoop/yarn-site.xml`**
``` xml
<configuration>

<!-- Site specific YARN configuration properties -->
	<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>hadoop-master</value>
	</property>
    <!--
	<property>
		<name>yarn.resourcemanager.bind-host</name>
		<value>0.0.0.0</value>
	</property>
	<property>
		<name>yarn.nodemanager.bind-host</name>
		<value>0.0.0.0</value>
	</property>
    -->
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
	<property>
		<name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
		<value>org.apache.hadoop.mapred.ShuffleHandler</value>
	</property>
	<property>
		<name>yarn.log-aggregation-enable</name>
		<value>true</value>
	</property>
	<property>
		<name>yarn.nodemanager.local-dirs</name>
		<value>file:/home/fenlan/Downloads/hadoop/yarn/local</value>
	</property>
	<property>
		<name>yarn.nodemanager.log-dirs</name>
		<value>file:/home/fenlan/Downloads/hadoop/yarn/log</value>
	</property>
	<!--
	<property>
		<name>yarn.nodemanager.remote-app-log-dir</name>
		<value>hdfs://hadoop-master:9000/home/fenlan/Downloads/hadoop/yarn-log/apps</value>
	</property>
	-->
</configuration>

```

**修改`hadoop/etc/hadoop/hadoop-env.sh`,将里面的JAVA_HOME设置为JAVA安装根目录**
``` shell
export HAVA_HOME＝/home/fenlan/Downloads/jdk1.8.0_60
```
> 想知道本机JAVA_HOME，敲个`which java`就知道了

**编辑`hadoop/etc/hadoop/master`,添加namenode主机,`hadoop/etc/hadoop/slaves`,添加datanode主机**
```
hadoop-master
```
```
hadoop-slave1
hadoop-slave2
```

> 需要说明，hadoop-slave hadoop-master为主机的hostname, 为了方便管理，需要将主机的hostname修改, 比如我的电脑开始hostname是`fenlan-K401UQ`, 在我的教程里面就需要改成`hadoop-slave1`(相当重要，否则在运行程序是会报错找不到主机),这里我个人感激仍然没有讲清楚，下面有一个youtube视频教程，辅助理解

**格式化HDFS**
``` shell
bin/hdfs namenode -format
```

**启动Hadoop的单节点集群**
``` shell
sbin/start-dfs.sh
sbin/start-yarn.sh
sbin/mr-jobhistory-daemon.sh start historyserver
sbin/start-all.sh
```

**停止Hadoop**
``` shell
sbin/stop-dfs.sh
sbin/stop-yarn.sh
sbin/mr-jobhistory-daemon.sh stop historyserver
sbin/stop-all.sh
```

### 加载HDFS
```
bin/hadoop fs -mkdir /fenlan/input
bin/hadoop fs -mkdir /fenlan/output
bin/hadoop fs -put files /fenlan/input
```

**运行一个单词统计实例**
```
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.1.jar wordcount /fenlan/input /fenlan/output/result
```

**查看结果**
```
bin/hadoop fs -cat /fenlan/output/result/*
```

## 推荐教程
[hadoop搭建教程](https://dwbi.org/etl/bigdata/183-setup-hadoop-cluster)
[hadoop分布式集群搭建](http://www.ityouknow.com/hadoop/2017/07/24/hadoop-cluster-setup.html)
[视频教程(youtube)](https://www.youtube.com/watch?v=-YEcJquYsFo)

## HBase 安装
1. 官网下载HBase,解压包并重命名为`hbase`
2. 编辑`hbase/conf/hbase-env.sh`的`JAVA_HOME`
3. 编辑`hbase/conf/hbase-site.xml`如下
``` xml
<configuration>
	<property>
		<name>hbase.rootdir</name>
		<value>hdfs://hadoop-master:9000/hbase</value>
	</property>
	<property>
		<name>hbase.cluster.distributed</name>
		<value>true</value>
	</property>
	<property>
		<name>hbase.master</name>
		<value>hadoop-master:60000</value>
	</property>
	<property>
		<name>hbase.zookeeper.quorum</name>
		<value>hadoop-master,hadoop-slave1,hadoop-slave2</value>
	</property>
	<property>
		<name>hbase.master.info.port</name>
		<value>60010</value>
	</property>
	<property>
		<name>hbase.master.port</name>
		<value>60000</value>
	</property>
</configuration>
```

4. 编辑`hbase/conf/regionservers`添加slaves主机
5. 启动HBase(先启动hadoop)
```
bin/start-hbase.sh
```

6. 管理HBase
```
bin/hbase shell
```

## 问题总结
**1.当无法启动datanode时将`hadoop/hdfs/datanode/current/`文件夹删掉重新启动**

**教程如果有错误，欢迎留言!**