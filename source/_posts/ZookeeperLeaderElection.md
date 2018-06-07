---
title: Zookeeper Leader选举源码解析
date: 2018-04-1８ 15:18:23
categories: 分布式计算
tags:
  - Zookeeper
  - FastLeaderElection
---

## Zookeeper简介
Zookeeper 是Apache Hadoop开源项目中的子项目，提供了一个分布式的协调服务框架。Zookeeper暴露了一组简单的操作原语(Primitive)集合，分布式应用能够基于这些原语实现更加高层的服务，包括同步机制、配置管理、服务器集群管理和统一命名服务等。

作为一个分布式的服务框架，Zookeeper主要解决分布式集群中应用系统的一致性问题，它采用类似文件系统目录的节点树的结构作为数据存储模型，并对已存储数据的状态变化进行维护和监控，通过监控这些数据状态的变化实现基于数据的集群管理。

Zookeeper 采用服务器集群的方式提供基本服务，服务器集群成为组，组中的成员具有两种角色，即一个唯一的领导者和若干个成员服务器，组能够为多个客户端提供服务。

<!--more-->
## Zookeeper角色
Zookeeper中的角色主要有以下几种

| 角色 | 角色说明 |
|--------|--------|
| 领导者(Leader) | Leader不接受客户端的请求，负责进行投票的发起和决议，最终更新状态      |
| 学习者(Learner)-跟随着(Follower) | Follower用于接收客户请求并向客户端返回结果，在选举过程中参与投票 |
| 学习者(Learner)-观察者(Obserber) | Observer可以接收客户端连接，将写请求转发给Leader节点。但它不参与投票过程，只同步Leader的状态。Observer的目的是为了扩展系统，提高读取速度 |
| 客户端(Client) | 请求发起方 |

## Zookeeper 工作原理
Zookeeper服务有两种不同的运行模式。一种是“独立模式”，即只有一个Zookeeper服务器。这种模式较为简单，比较适合测试环境，但不能保证高可用性和恢复性。在实际应用中，Zookeeper通常以“复制模式”运行在一个计算机集群上。Zookeeper通过复制来实现高可用性，只要集群中半数以上的机器处于可用状态，它就能提供服务。也就是说，在一个有2n+1节点的集群中，任意n台机器出现故障，都可以保证服务继续，因为剩下的n+1台超过了半数。出于这个原因，一个集群通常包含奇数台机器。

从概念上说，Zookeeper非常简单：它所做的就是确保对znode树的每一个修改都会被复制到集群中超过半数的机器上。如果少于半数的机器出现故障，则最少有一台机器会保存最新状态。其余的副本最终也会更新到这个状态。为了实现这个想法，Zookeeper使用了Azb协议。Zab协议包含两个可以无限重复的阶段：

`阶段一`: 当服务启动或者Leader崩溃后，Zab就进入了阶段1。当Leader被选举出来，且超过半数(或指定数量)的Learner完成了和Leader的状态同步以后，阶段1就结束了。状态同步保持了Leader和其他服务器具有相同的系统状态。
`阶段二`: 原子广播
所有的写请求都被转发给Leader，再有Leader将更新广播给Learner。当半数以上的Follower已经将修改持久化以后，Leader才会提交这个更新，然后客户端才会收到一个更新成功的响应。这个用来达成共识的协议被设计成具有原子性，因此每个修改要么成功，要么失败。

## Zookeeper Leader选举源码
Zookeeper 中QuorumPeer类出于源码“中心”位置，它与多个类关联，负责管理quorum协议。QuorumPeer继承自Thread，是集群环境下Zookeeper服务器的主线程类。它有4种状态： LOOKING、FOLLOWING、LEADING和OBSERVING，由成员state标识:
``` java
// src/java/main/org/apache/zookeeper/server/quorum/QuorumPeer.java
public enum ServerState {
        LOOKING, FOLLOWING, LEADING, OBSERVING;
    }
```

这4中状态决定了QuorumPeer的行为，即在集群中充当什么角色。state的初始状态是LOOKING，意思是寻找Leader服务器。一旦选举出Leader，QuorumPeer就切换自身状态，修改state的值，同时实例化对应的服务器控制类。服务器控制类有3种，Leader、Follower和Observer，它们作为QuorumPeer成员对象存在。

> 前面废话了这么多现在正式进入正题。。。。。。

QuorumPeer类有一个属性成员electionAlg，它是Election接口类型。服务器启动时，根据配置信息决定Election的具有实现类，指定选举算法。Election共有两个抽象方法
``` java
public interface Election {
    public Vote lookForLeader() throws InterruptedException;
    public void shutdown();
}
```
其中lookForLeader()是核心方法，返回一个Vote类型对象，标识被推荐服务器。Election的实现类有`LeaderEletcion`、`FastLeaderElection`、`AuthFastLeaderElection`。它们之间的区别在于通信机制以及lookForLeader的算法实现。由于源码中已经表示`LeaderElection`和`AuthFastLeaderElection`已经在3.4.0版本被弃用，因此在这里主要看`FastLeaderElection`。

`FastLeaderElection`有3个内部线程类: Listener、SendWorker、RecvWorker。Listener线程新建ServerSocketChannel，监听选举端口，一旦接收到连接请求，调用receiveConnection方法，启动发送线程SendWorker和接收线程RecvWorker。

`LookForLeader`详细源码
``` java
synchronized(this){
    logicalclock++;
    updateProposal(getInitId(), getInitLastLoggedZxid(), getPeerEpoch());
}

sendNotifications();
```
原子性操作，Leader选举开始每个节点投自己一票，其中`getInitId()`、`getInitLastLoggedZxid()`、`getPeerEpoch()`表示当前self节点的状态，然后将投票结果send给其他节点。接下来就循环交换投票信息，直到找到Leader节点。
``` java
while ((self.getPeerState() == ServerState.LOOKING) && (!stop)) {
    /*
     * 从投票消息队列中接收一条消息
     */
    Notification n = recvqueue.poll(notTimeout,
              TimeUnit.MILLISECONDS);
    if (n == null) {...}
    /*
     * 检查n节点是不是参与投票的节点，只有PeerType=PARTICIPANT的节点消息才会参与投票
     * Observers are not contained in this view, only nodes with
     * PeerType=PARTICIPANT.
     */
    else if(self.getVotingView().containsKey(n.sid)) {
        switch (n.state) {
            case LOOKING: 跟自己的投票比较。
            case OBSERVING: 没有操作
            case FOLLOWING:
            case LEADING: 当已经收到LEADING和FOLLOWING表示已经票选出Leader，然后投最后一票给Leader，结束投票
            default: 没有操作
    }
}
```
分别看`LOOKING`和`FOLLOWING`、`LEADING`干了什么
`LOOKING`在干的事情
``` java
/*
 * 表示投票轮次大于本节点记录的轮次，表示自己已经落后投票了，将自己的
 * 投票轮次设置为最新的，清空自己的票箱，这个票箱记录了集群中其他节点
 * 的投票结果
 */
if (n.electionEpoch > logicalclock) {
    logicalclock = n.electionEpoch;
    recvset.clear();
    /*
     * 将n节点的投票结果与自己的投票结果比较,如果投票比自己的投票合理，
     * 更新自己的投票，否则还是投自己
     */
    if(totalOrderPredicate(n.leader, n.zxid, n.peerEpoch,
    　　　　getInitId(), getInitLastLoggedZxid(), getPeerEpoch())) {
        updateProposal(n.leader, n.zxid, n.peerEpoch);
    } else {
    	updateProposal(getInitId(),
        	getInitLastLoggedZxid(),
        	getPeerEpoch());
    }
    // 发送自己的投票结果
    sendNotifications();
} else if (n.electionEpoch < logicalclock) {
    // 投票轮次比自己记录的轮次小，说明这个投票已经过时，不处理
    break;
} else if (totalOrderPredicate(n.leader, n.zxid, n.peerEpoch,
          proposedLeader, proposedZxid, proposedEpoch)) {
    // 如果是一个轮次，将n节点的投票与自己比较，如果投票更合理，更新投票
    updateProposal(n.leader, n.zxid, n.peerEpoch);
    sendNotifications();
}
// 将n节点的投票记录下来
recvset.put(n.sid, new Vote(n.leader, n.zxid, n.electionEpoch, n.peerEpoch));

/*
 * 在自己的票箱中查看自己投票的节点是否已经被集群中一半以上的
 * 节点认可了，如果已经有一半以上的节点认可，则结束选举
 */
if (termPredicate(recvset,
         new Vote(proposedLeader, proposedZxid,
         logicalclock, proposedEpoch))) {

    // Verify if there is any change in the proposed leader
    // 这里没太理解，
    while((n = recvqueue.poll(finalizeWait,
            TimeUnit.MILLISECONDS)) != null){
        if(totalOrderPredicate(n.leader, n.zxid, n.peerEpoch,
                 proposedLeader, proposedZxid, proposedEpoch)){
            recvqueue.put(n);
            break;
        }
    }

    /*
     * This predicate is true once we don't read any new
     * relevant message from the reception queue
     */
    if (n == null) {
        self.setPeerState((proposedLeader == self.getId()) ?
            ServerState.LEADING: learningState());

        Vote endVote = new Vote(proposedLeader,
            proposedZxid,
            logicalclock,
            proposedEpoch);
        leaveInstance(endVote);
        return endVote;
    }
}
```
其中没有说清楚接收的投票如何和自己的投票比较的，也就是`totalOrderPredicate`方法的实现
``` java
protected boolean totalOrderPredicate(long newId, long newZxid, long newEpoch, long curId, long curZxid, long curEpoch) {
    if(self.getQuorumVerifier().getWeight(newId) == 0){
        return false;
    }

    /*
     * We return true if one of the following three cases hold:
     * 1- New epoch is higher
     * 2- New epoch is the same as current epoch, but new zxid is higher
     * 3- New epoch is the same as current epoch, new zxid is the same
     *  as current zxid, but server id is higher.
     */

    return ((newEpoch > curEpoch) ||
            ((newEpoch == curEpoch) &&
            ((newZxid > curZxid) || ((newZxid == curZxid) && (newId > curId)))));
}
```
可以看出，比较的顺序是Epoch、zxid、Id，优先选投票轮次高的，投票轮次相同选Zxid高的，Zxid相同选id高的，因此在Zookeeper启动的时候，往往id高的获得Leader，但不绝对，比如在5个节点的集群中，启动顺序分别是1->2->3->4->5，当票选到节点3时已经票选超过半数，那么后面启动的4和5就直接成为follower

接下来再分析`FOLLOWING`和`LEADING`
``` java
if(n.electionEpoch == logicalclock){
    recvset.put(n.sid, new Vote(n.leader, n.zxid, n.electionEpoch, n.peerEpoch));
    // 如果推举的Leader是自己，把自己的状态改为LEADING
    if(ooePredicate(recvset, outofelection, n)) {
        self.setPeerState((n.leader == self.getId()) ?
        ServerState.LEADING: learningState());
    Vote endVote = new Vote(n.leader, n.zxid, n.electionEpoch, n.peerEpoch);
    leaveInstance(endVote);
    return endVote;
}
/*
 * 确定Leader之前要保证半数以上的节点已经成为follower
 * Before joining an established ensemble, verify
 * a majority is following the same leader.
 */
outofelection.put(n.sid, new Vote(n.version,
        n.leader,
        n.zxid,
        n.electionEpoch,
        n.peerEpoch,
        n.state));

if(ooePredicate(outofelection, outofelection, n)) {
        synchronized(this){
            logicalclock = n.electionEpoch;
            self.setPeerState((n.leader == self.getId()) ?
                ServerState.LEADING: learningState());
            }
        Vote endVote = new Vote(n.leader, n.zxid, n.electionEpoch, n.peerEpoch);
        leaveInstance(endVote);
        return endVote;
}
```

这里需要解释一下`ooePredicate`
``` java
/**
 * This predicate checks that a leader has been elected. It doesn't
 * make a lot of sense without context (check lookForLeader) and it
 * has been separated for testing purposes.
 * 有两个map，一个表示收到的投票集合，一个表示LEADING状态节点和FOLLOWING节点的投票
 * 集合，这里要确定两个条件：一个是在收到的LOOKING节点中半数认为n节点应该是Leader
 * 另一个是LEADING节点必须告诉其他节点自己是LEADING状态，避免一直投票
 *
 * @param recv  map of received votes
 * @param ooe   map containing out of election votes (LEADING or FOLLOWING)
 * @param n     Notification
 * @return
 */
protected boolean ooePredicate(HashMap<Long,Vote> recv,
                                HashMap<Long,Vote> ooe,
                                Notification n) {

    return (termPredicate(recv, new Vote(n.version,
                                         n.leader,
                                         n.zxid,
                                         n.electionEpoch,
                                         n.peerEpoch,
                                         n.state))
            && checkLeader(ooe, n.leader, n.electionEpoch));

}
```
再来看一下`checkLeader`
``` java
	/**
     * In the case there is a leader elected, and a quorum supporting
     * this leader, we have to check if the leader has voted and acked
     * that it is leading. We need this check to avoid that peers keep
     * electing over and over a peer that has crashed and it is no
     * longer leading.
     *
     * @param votes set of votes
     * @param   leader  leader id
     * @param   electionEpoch   epoch id
     */
    protected boolean checkLeader(
            HashMap<Long, Vote> votes,
            long leader,
            long electionEpoch){

        boolean predicate = true;

        /*
         * If everyone else thinks I'm the leader, I must be the leader.
         * The other two checks are just for the case in which I'm not the
         * leader. If I'm not the leader and I haven't received a message
         * from leader stating that it is leading, then predicate is false.
         */

        if(leader != self.getId()){
            if(votes.get(leader) == null) predicate = false;
            else if(votes.get(leader).getState() != ServerState.LEADING) predicate = false;
        } else if(logicalclock != electionEpoch) {
            predicate = false;
        }

        return predicate;
    }
```

`FastLeaderElection`源码分析完了，总结一下：首先，每个节点先给自己投票，然后将投票信息发送给集群中的其他节点；每个节点收到其他节点的投票，放在**投票队列**中，从队列中提取投票，跟自己的投票比较，如果投票更合理，就替换自己的投票，否则不改变自己的投票。然后检测自己当前的投票是否相同于集群半数以上的节点，如果相同于半数以上投票，则判断这个投票是不是投的自己，如果是投的自己，则改变自己的状态为LEADING，否则改为FOLLOWING或者OBSERVER，最后返回最后的投票，并结束`LookForLeader`。