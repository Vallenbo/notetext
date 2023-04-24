# 一、raft算法

​		一个 etcd 集群，通常会由 3 个或者 5 个节点组成，多个节点之间通过 Raft 一致性算法的完成分布式一致性协同，算法会选举出一个主节点作为 leader，由 leader 负责数据的同步与数据的分发。当 leader 出现故障后系统会自动地选取另一个节点成为 leader，并重新完成数据的同步。客户端在多个节点中，仅需要选择其中的任意一个就可以完成数据的读写，内部的状态及数据协同由 etcd 自身完成。

简单：基于HTTP+JSON的API让你用url命令就可以轻松使用。
安全：可选SSL客户认证机制。
快速：每个实例每秒支持一千次写操作。
可信：使用Raft算法充分实现了分布式。

leader选举的过程是：

> 1、增加term号；2、给自己投票；3、重置选举超时计时器；4、发送请求投票的RPC给其它节点。

Raft是一个用于管理日志一致性的协议。它将分布式一致性分解为多个子问题：Leader选举（Leader election）、日志复制（Log replication）、安全性（Safety）、日志压缩（Log compaction）等。同时，Raft算法使用了更强的假设来减少了需要考虑的状态，使之变的易于理解和实现。Raft将系统中的角色分为领导者（Leader）、跟从者（Follower）和候选者（Candidate）：

> `Leader`：接受客户端请求，并向Follower同步请求日志，当日志同步到大多数节点上后告诉Follower提交日志。
>
> `Follower`：接受并持久化Leader同步的日志，在Leader告之日志可以提交之后，提交日志。
>
> `Candidate`：Leader选举过程中的临时角色。

## 1、Raft分为哪几个部分？

  主要是分为leader选举、日志复制、日志压缩、成员变更等。

## 2、Raft中任何节点都可以发起选举吗？

  Raft发起选举的情况有如下几种：

刚启动时，所有节点都是follower，这个时候发起选举，选出一个leader；

当leader挂掉后，时钟最先跑完的follower发起重新选举操作，选出一个新的leader。

成员变更的时候会发起选举操作。

## 3、Raft中选举中给候选人投票的前提？

  Raft确保新当选的Leader包含所有已提交（集群中大多数成员中已提交）的日志条目。这个保证是在RequestVoteRPC阶段做的，candidate在发送RequestVoteRPC时，会带上自己的last log entry的term_id和index，follower在接收到RequestVoteRPC消息时，如果发现自己的日志比RPC中的更新，就拒绝投票。日志比较的原则是，如果本地的最后一条log entry的term id更大，则更新，如果term id一样大，则日志更多的更大(index更大)。

## 5、Raft数据一致性如何实现？
  主要是通过日志复制实现数据一致性，leader将请求指令作为一条新的日志条目添加到日志中，然后发起RPC 给所有的follower，进行日志复制，进而同步数据。

## 6、Raft的日志有什么特点？
  日志由有序编号（log index）的日志条目组成，每个日志条目包含它被创建时的任期号（term）和用于状态机执行的命令。

## 10、Raft日志压缩是怎么实现的？增加或删除节点呢？？
  在实际的系统中，不能让日志无限增长，否则系统重启时需要花很长的时间进行回放，从而影响可用性。Raft采用对整个系统进行snapshot来解决，snapshot之前的日志都可以丢弃（以前的数据已经落盘了）。

  snapshot里面主要记录的是日志元数据，即最后一条已提交的 log entry的 log index和term。
