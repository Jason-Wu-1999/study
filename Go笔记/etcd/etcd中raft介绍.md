

etcd是一个分布式、一致性的键值存储系统，主要用于配置共享和服务发现。

### 架构图

![img](https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/3914706545-5dabc6002dd98_articlex)

**从etcd的架构图中我们可以看到，etcd主要分为四个部分。**

- HTTP Server： 用于处理用户发送的API请求以及其它etcd节点的同步与心跳信息请求。
- Store：用于处理etcd支持的各类功能的事务，包括数据索引、节点状态变更、监控与反馈、事件处理与执行等等，是etcd对用户提供的大多数API功能的具体实现。
- ==Raft：Raft强一致性算法的具体实现，是etcd的核心。==
- WAL：Write Ahead Log（预写式日志），是etcd的数据存储方式。除了在内存中存有所有数据的状态以及节点的索引以外，etcd就通过WAL进行持久化存储。WAL中，所有的数据提交前都会事先记录日志。Snapshot是为了防止数据过多而进行的状态快照；Entry表示存储的具体日志内容。

通常，一个用户的请求发送过来，会经由HTTP Server转发给Store进行具体的事务处理，如果涉及到节点的修改，则交给Raft模块进行状态的变更、日志的记录，然后再同步给别的etcd节点以确认数据提交，最后进行数据的提交，再次同步。

# 如何保证强一致性？

使用raft协议 包括两个过程  选主过程、主从数据同步    ==在etcd中数据只有一个流向，leader到follower==

## 主从数据同步

1. client连接follower或者leader，如果连接的是follower则，follower会把client的请求(写请求，读请求则自身就可以直接处理)转发到leader
2. leader接收到client的请求，将该请求转换成entry，写入到自己的日志中，得到在日志中的index，会将该entry发送给所有的follower(实际上是批量的entries)
3. follower接收到leader的AppendEntriesRPC请求之后，会将leader传过来的批量entries写入到文件中（通常并没有立即刷新到磁盘），然后向leader回复OK,leader收到过半的OK回复之后，就认为可以提交了，然后应用到leader自己的状态机中，leader更新commitIndex，应用完毕后回复客户端
4. 在下一次leader发给follower的心跳中，会将leader的commitIndex传递给follower，follower发现commitIndex更新了则也将commitIndex之前的日志都进行提交和应用到状态机中

**总结：**在leader收到数据操作的请求，先不着急更新本地数据（数据是持久化在磁盘上的），而是生成对应的log，然后把生成log的请求广播给所有的follower，每个follower在收到请求之后听从leader的命令，也写入log，然后返回success回去。leader收到过半的OK回复之后，就认为可以提交了，**进行二次提交**，正式写入数据（持久化），然后再告诉follower，他们也持久化

**其中log可以用来恢复数据**

## 选举过程

1. 当集群初始化时候，每个节点都是Follower角色；都维护一个随机的timer，如果timer时间到了还没有收到leader的消息，自己就会变成candidate，竞选leader，
2. 当Follower在一定时间内没有收到来自主节点的心跳，会将自己角色改变为Candidate，并发起一次选主投票；当收到包括自己在内超过半数节点赞成后，选举成功；当收到票数不足半数选举失败，或者选举超时。若本轮未选出主节点，将进行下一轮选举（出现这种情况，是由于多个节点同时选举，所有节点均为获得过半选票）。
3. 集群中存在至多1个有效的主节点，**通过心跳与其他节点同步数据**
4. Candidate节点收到来自主节点的信息后，会立即终止选举过程，进入Follower角色。为了避免陷入选主失败循环，每个节点未收到心跳发起选举的时间是一定范围内的随机值，这样能够避免2个节点同时发起选主。
   





## etcd分布式锁实现的基础机制

### Lease机制

租约机制（**TTL**，Time To Live），etcd 可以为存储的 **key-value** 对设置租约，**当租约到期，key-value 将失效删除**；

同时也支持**续约**，通过客户端可以在租约到期之前续约， 以避免 **key-value** 对过期失效。

Lease 机制可以**保证分布式锁的安全性**，为锁对应的 key 配置租约， **即使锁的持有者因故障而不能主动释放锁，锁也会因租约到期而自动释放**。

### Revision机制

每个 key 带有一个 Revision 号，每进行一次事务便+1，它是全局唯一的， 通过 Revision 的大小就可以知道进行写操作的顺序。

在实现分布式锁时，多个客户端同时抢锁， 根据 Revision 号大小依次获得锁，可以避免 “羊群效应” ，实现**公平锁**。

> 羊群效应：羊群是一种很散乱的组织，平时在一起也是盲目地左冲右撞，但一旦有一只头羊动起来，其他的羊也会不假思索地一哄而上，全然不顾旁边可能有的狼和不远处更好的草。
>
> etcd的Revision机制，可以根据Revision号的大小顺序进行写操作，因而可以避免“羊群效应”。
>
> 这和zookeeper的临时顺序节点+监听机制可以避免羊群效应的原理是一致的。

### Prefix机制

即前缀机制。

例如，一个名为 /etcd/lock 的锁，两个争抢它的客户端进行写操作， 实际写入的 key 分别为：key1="/etcd/lock/UUID1"，key2="/etcd/lock/UUID2"。

其中，UUID 表示全局唯一的 ID，确保两个 key 的唯一性。

写操作都会成功，但返回的 Revision 不一样， 那么，如何判断谁获得了锁呢？通过前缀 /etcd/lock 查询，返回包含两个 key-value 对的的 KeyValue 列表， 同时也包含它们的 Revision，通过 Revision 大小，客户端可以判断自己是否获得锁。

#### 具体步骤

- 步骤 1: 准备

客户端连接 Etcd，以 /lock/mylock 为前缀创建全局唯一的 key，假设第一个客户端对应的 key="/lock/mylock/UUID1"，第二个为 key="/lock/mylock/UUID2"；客户端分别为自己的 key 创建租约 - Lease，租约的长度根据业务耗时确定，假设为 15s；

- 步骤 2: 创建定时任务作为租约的“心跳”

当一个客户端持有锁期间，其它客户端只能等待，**为了避免等待期间租约失效**，**客户端需创建一个定时任务作为“心跳”进行续约**。此外，如果持有锁期间客户端崩溃，心跳停止，key 将因租约到期而被删除，从而锁释放，避免死锁。

- 步骤 3: 客户端将自己全局唯一的 key 写入 Etcd

进行 put 操作，将步骤 1 中创建的 key 绑定租约写入 Etcd，根据 Etcd 的 Revision 机制，假设两个客户端 put 操作返回的 Revision 分别为 1、2，客户端需记录 Revision 用以接下来判断自己是否获得锁。

- 步骤 4: 客户端判断是否获得锁

客户端以前缀 /lock/mylock 读取 keyValue 列表（keyValue 中带有 key 对应的 Revision），判断自己 key 的 Revision 是否为当前列表中最小的，如果是则认为获得锁；否则监听列表中前一个 Revision 比自己小的 key 的删除事件，一旦监听到删除事件或者因租约失效而删除的事件，则自己获得锁。

- 步骤 5: 执行业务

获得锁后，操作共享资源，执行业务代码。

- 步骤 6: 释放锁

完成业务流程后，删除对应的key释放锁。

