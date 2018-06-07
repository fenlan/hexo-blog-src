---
title: Kafka 简述
date: 2018-04-02 15:18:23
categories: 分布式计算
tags:
  - Kafka
  - 持久化
  - 备份
  - 日志压缩
---

## 持久化
`Don.t fear the filesystem`
Kafka非常依赖文件系统来存储和缓存消息，但人们在这里总是有一个错觉`disks are slow`。事实上，`disks`可以很慢，也可以很快，这取决于人们怎么用它，一个设计合理的磁盘结构通常可以和网络一样快。

这里有一个事实：磁盘读取快慢主要取决于寻道延时。`six 7200rpm SATA RAID-5 array`的磁盘`linear writes`的读取速度大概为600MB/sec，但`random writes`的读取速度为100k/sec，正因为现在操作系统通常采用随机存储的方式，导致人们对磁盘速度产生了错觉。

`持久化策略`
当我们保持消息队列的时候，快用完内存空间时，并不采用操作系统的策略(尽可能保持内存中的数据，将不常用的数据块替换出去)，而是将内存中的消息全部冲洗到文件系统中。个人理解Kafka能够高吞吐的原因在于`Batching`、`larger network packets`、`larger sequential disk operations`、`contiguous memory block`，所有的策略都为了保证Kafka将随机消息写转为线性写。
<!--more-->
> 个人翻译能力有限，留下英文原文

> This suggests a design which is very simple: rather than maintain as much as possible in-memory and flush it all out to the filesystem in a panic when we run out of space, we invert that. All data is immediately written to a persistent log on the filesystem without necessarily flushing to disk. In effect this just means that it is transferred into the kernel's pagecache.

此外Kafka集群保留所有publish的数据，无论这些消息有没有被消费。比如，保留策略为两天，那么两天之内，消息仍然在磁盘上，两天后为了腾出空间，就将数据移除。这样的好处在于，如果有多个消费者不会影响Kafka消息，也不会影响其他消费者。

## Producer
`负载均衡` : 生产者直接将数据发送给一个分区的`leader broker`, 它没有介入路由层。客户端控制它将消息发布到哪个分区。这可以随机完成，实现一种随机负载平衡，或者可以通过某种语义分区功能完成。Kafka公开了用于语义分区的接口，方法是允许用户指定要分区的密钥并使用它来hash分区（如果需要，还可以选择覆盖分区函数）。例如，如果选择的密钥是用户ID，则给定用户的所有数据都将被发送到同一分区。这反过来将允许消费者对他们的消费做出局部性假设。这种分区风格方便了那些对用户消息所在分区有严格要求的情况。
`异步发送` : 批次是效率的重要推动力之一，为了实现批量生产，Kafka producer将尝试在内存中积累数据，并在单个请求中发送更大批次的数据。批处理可以配置消息积累大小，并且不超过某个固定的延迟限制（比如64k或10ms）。这允许发送积累好的大块消息，并且在服务器上只进行几次较大的 I/O 操作。这种缓冲是可配置的，并提供了一种机制来折中少量额外的延迟以获得更好的吞吐量

`Message delivery semantics` : 0.11.0.0版本之前，如果`Producer`没有收到代表消息已经提交的响应，那么`Producer`只有重新发送消息，这样有个问题：在消息重发期间，上一次发送的消息已经提交成功，那么同一块消息可能被重复写入log。从0.11.0.0版本开始Kafka提供了一个选项保证消息不会被重复，`broker`分配给每个`Producer`一个ID，使用`Producer`发送来的序列码(每个消息都有一个序列码)删除重复的消息。另外从0.11.0.0开始`Producer`支持发送消息给多个`topic`分区，使用的是像合约一样的语义。
> 翻译能力有限，留下英文原文

> Also beginning with 0.11.0.0, the producer supports the ability to send messages to multiple topic partitions using transaction-like semantics: i.e. either all messages are successfully written or none of them are.

上述说讲的是如何保证消息提交，但不是所有的用例都需要这样强力的保证，对于延时敏感的用例，我们允许`Producer`指定其耐心等待程度，它可以是10ms量级的等待，甚至是完全的异步。

## Consumer
Kafka消费者通过向希望消费的分区的`leader`发出“获取”请求而工作。消费者在消费请求中会指定日志`offset`，然后接收`offset`之后的一大块数据`a chunk of log`，因此，消费者对这个位置具有重要的控制权，并且可以倒带它以在需要时重新消费数据。由于`offset`的控制权在消费者手里，所以理论上，消费者是可以按照自己想要的方式去消费消息，比如`reset to an older offset to reprocess data from the past`、`skip ahead to the most recent record and start consuming from "now"`。

每个消费者都有一个自己的消费组名称标示，每一个发布到topic上的消息会被投递到每个订阅了此topic的消费者组的某一个消费者（译者注：每组都会投递，但每组都只会投递一份到某个消费者）。这个被选中的消费者实例可以在不同的处理程序中或者不同的机器之上。

如果所有的消费者实例都有相同的消费组标示(consumer group),那么整个结构就是一个传统的消息队列模式，消费者之间负载均衡。
如果所有的消费者实例都采用不同的消费组，那么真个结构就是订阅模式，每一个消息将被广播给每一个消费者。

`Push还是Pull` : 最开始考虑的问题是消费值主动从`brokers` `pull`数据还是`brokers` `push`数据给消费者。Kafka采用了非常传统的设计，生产者`push`数据给`brokers`然后消费者从`brokers` `pull`数据。在其他的 logging-centric systems, such as Scribe and Apache Flume 中采用的是将数据推送给下游的方式。

它们各有优缺点。然而，`push-bases`的系统难以处理不同的消费者，因为`brokers`控制着数据传输的速度，我们的目标通常是消费者能够以最大可能的速度消费。不幸的是`push-based`当消费率低于生产速度时，消费者往往会不知所措。`pull-based system`有个很舒服的属性，消费者只需要简单地`falls behind`并尽可能的跟上生产者速率。还有一个优点是它适用于发送给消费者的大量批量数据。`push-based`必须发送一个请求给下游，同时积累数据然后发送数据给消费者，但它并不知道下游是否有能力处理这些数据。

`pull-based`的缺陷在于当`brokers`没有消息可以消费时，消费者就陷入轮询的`tight loop`。为了规避这个问题，Kafka在pull请求里面带有参数允许消费者请求阻塞在`long poll`直到数据到达(同时可选项是等到给定大小的数据时再消费以保证大块传输减少网络延时)

## 备份
Kafka为Topic的每个Partition日志进行备份，备份数量可以由用户进行配置。这保证了系统的自动容错，如果有服务器宕机，消息可以从剩余的服务器中读取。备份的单位是Topic的分区。在没有发生异常的情况下，Kafka中每个分区都会有一个Leader和0或多个Follower。备份包含Leader在内（也就是说如果备份数为3，那么有一个Leader Partition和两个Follower Partition）。所有的读写请求都落在Leader Partition上。通常情况下分区要比Broker多，Leader分区分布在Broker上。Follower上的日志和Leader上的日志相同，拥有相同的偏移量和消息顺序（当然，在特定时间内，Leader上日志会有一部分数据还没复制到Follower上）。

Follower作为普通的Consumer消费Leader上的日志，并应用到自己的日志中。Leader允许Follower自然的，成批的从服务端获取日志并应用到自己的日志中。大部分分布式系统都需要自动处理故障，需要对节点“alive”进行精确的定义。例如在Kafka中，节点存活需要满足两个条件：
- 节点需要保持它和ZooKeeper之间的Session（通过ZK的心跳机制）
- 如果是Follower，需要复制Leader上的写事件，并且复制进度没有“落后太多”

一条消息在被应用到所有的备份上之后被认为是“已经提交的”。只有提交了的消息会被Consumer消费。这意味着Consumer不需要担心Leader节点宕机后消息会丢失。另一方面，Producer可以配置是否等待消息被提交，这取决于他们在延迟和可用性上的权衡。这个可以通过Producer的配置项中设置。Kafka提供了一条消息被提交之后，只要还有一个备份可用，消息就不会丢失的保证

## 日志
包含两个partition，名称为“my_topic”的Topic的日志包含两个目录（名称为my_topic_0和my_topic_1）,其中包含该Topic的消息的数据文件。日志文件的格式是log entry的序列；每个log entry都是4字节的消息长度N加上后面N个字节的消息数据。每条消息都有一个64位的offset标识这条消息在这个Topic的Partition中的偏移量。消息在磁盘中的存储格式如下所示。每个日志文件都以它存储的第一条消息的offset命名。所以第一个文件会命名为00000000000.kafka，随后每个文件的文件名将是前一个文件的文件名加上S的正数，S是配置中指定的单个文件的大小

使用消息的Offset作为消息ID是不常见的。我们初始的想法是在Producer生成一个GUID作为Message ID，并在Broker上维持ID和Offset之间的映射关系。但是因为Consumer需要为每个Server维持一个ID，那么GUID的全局唯一性就变得没什么意义了。此外，维持一个随机的ID和Offset的映射关系将给索引的构建带来巨大的负担，本质上需要一个完整的持久化的随机存取的数据结构。因此，为了简化查找结构，我们决定使用每个分区的原子计数器，它可以和分区ID加上ServerID来唯一标识一条消息。一旦使用了计数器，直接使用Offset进行跳转是顺其自然的，两者都是分区内单调递增的整数。由于偏移量从消费者API中隐藏起来，因此这个决定是最终的实现细节，所以我们采用更有效的方法。

### Writes
日志允许连续追加到最后一个文件。当文件达到配置的大小时（如1GB）将滚动到一个新文件。日志采用两个配置：M，配置达到多少条消息后进行刷盘；S，配置多长时间之后进行刷盘。这个持久化策略保证最多只丢失M条消息或者S秒之内的消息。

### Reads
读取通过提供64位的offset和S-byte的chunk大小来实现。这将返回包含在S-byte的buffer的消息迭代。S比任意单条消息都大，但是如果在异常的超大消息的情况下，读取操作可以通过多次重试，每次都将buffer大小翻倍，直到消息被读取成功。最大消息大小和buffer大小可以配置，用于拒绝超过特定大小的消息，以限制客户端读取消息时需要拓展的buffer大小。buffer可能以不完整的消息作为结尾，这可以通过消息大小来轻松的检测到。
实际的读取操作首先需要定位offset所在的文件，再将offset转化为文件内相对的偏移量，然后从文件的这个偏移量开始读取数据。搜索操作通过内存中映射的文件的简单的二分查找完成。
日志提供了获取最近写入消息的能力以允许客户端从“当前时间”开始订阅。这在客户端无法在指定天数内消费掉消息的场景中非常有用。在这种情况下，如果客户端尝试消费一个不存在的offset将抛出OutOfRangeException异常并且可以根据场景重置或者失败

### Deletes
数据删除一次删除一个日志段。日志管理器允许通过插件的形式实现删除策略来选择那些文件是合适删除的。当前的删除策略是日志文件的修改时间已经过去N天，保留最近N GB数据的策略也是有用的。为了避免删除时锁定读取操作，我们采用copy-on-write的方式来实现，以保证一致性的视图。

## 日志压缩
日志压缩确保Kafka会为一个Topic分区数据日志中保留至少message key的最后一个值。在持久化那部分我们已经说明了在一断时间或达到特定大小的时候丢弃旧日志的简单方法。这适用于像日志这样每一条数据都是独立数据的情况。但是重要类别的数据是根据key处理的数据（例如DB中表的变更数据）。

让我们来讨论这样一个具体的流的例子。一个Topic包含了用户email address信息；每一次用户变更邮箱地址，我们都向这个topic发送一条消息，使用用户ID作为primay key。现在我们已经为用户ID为123的用户发送了一些消息，每条消息包含了email address的变更:
```
123 => bill@microsoft.com
        .
        .
        .
123 => bill@gatesfoundation.org
        .
        .
        .
123 => bill@gmail.com
```
日志压缩为我们提供了更精细的保留机制，至少保存每个key最后一个变更（如123 => bill@gmail.com）。这样做我们确保了这个日志包含了所有key最后一个值的快照。这样Consumer可以重建状态而不需要保留完成的变更日志。

假设我们有无限的日志，记录每次变更日志，我们从一开始就捕获每一次变更。使用这个完整的日志，我们可以通过回放日志来恢复到任何一个时间点的状态。这种假设的情况下，完整的日志是不实际的，对于那些每一行记录会变更多次的系统，即使数据集很小，日志也会无限的增长下去。丢弃旧日志的简单操作可以限制空间的增长，但是无法重建状态——因为旧的日志被丢弃，可能一部分记录的状态会无法重建（这写记录所有的状态变更都在就日志中）。

日志压缩机制是更细粒度的，每个记录都保留的机制，而不是基于时间的粗粒度。这个想法是选择性的删除哪些有更新的变更的记录的日志。这样最终日志至少包含每个key的记录的最后一个状态。压缩操作通过在后台周期性的拷贝日志段来完成。清除操作不会阻塞读取，并且可以被配置不超过一定IO吞吐来避免影响Producer和Consumer。

**补上几张图片**
![](http://kafka.apache.org/11/images/log_cleaner_anatomy.png)

![](http://kafka.apache.org/11/images/log_compaction.png)

## 参考链接
- [https://kafkadoc.beanmr.com/010_getting_started/01_introduction_cn.html](https://kafkadoc.beanmr.com/010_getting_started/01_introduction_cn.html)
- [http://kafka.apache.org/intro](http://kafka.apache.org/intro)
- [http://kafka.apache.org/documentation/](http://kafka.apache.org/documentation/)
- [备份](https://www.jianshu.com/p/7833e958fd0c)
- [日志压缩](https://www.jianshu.com/p/7abe0b0727fb)
- [日志](https://www.jianshu.com/p/d41935106b52)