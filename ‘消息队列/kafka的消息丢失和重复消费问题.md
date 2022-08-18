# kafka如何保证消息的顺序性

https://www.cnblogs.com/shujiying/p/14540753.html?ivk_sa=1024320u

- # kafka 中单个 分区 内部的数据 ，一定是有序的

- 一个topic中多个分区不一定有序

### kafka如何保证单个partition中是有序的

producer发消息到队列时，通过加锁保证有序。

### 那么是否有这样一个问题呢？

先后两条消息发送时，前一条消息发送失败，后一条消息发送成功，然后失败的消息重试后发送成功，造成乱序。

为了解决重试机制引起的消息乱序为实现Producer的幂等性，Kafka引入了Producer ID（即PID）和Sequence Number。

对于每个PID，该Producer发送消息的每个<Topic, Partition>都对应一个单调递增的Sequence Number。

同样，Broker端也会为每个<PID, Topic, Partition>维护一个序号，并且每Commit一条消息时将其对应序号递增。

对于接收的每条消息，如果其序号比Broker维护的序号大一，则Broker会接受它，否则将其丢弃

如果消息序号比Broker维护的序号差值比一大，说明中间有数据尚未写入，即乱序，此时Broker拒绝该消息

如果消息序号小于等于Broker维护的序号，说明该消息已被保存，即为重复消息，Broker直接丢弃该消息

发送失败后会重试，这样可以保证每个消息都被发送到broker

#### 每个Partition保证有效了，但是在多线程消费的时候又存在问题

消费者从 partition 中取出来数据的时候，也一定是有顺序的。到这里，顺序还是 ok 的，没有错乱。

但是消费者这里还是可能会有多个线程来并发来处理消息，因为如果消费者是单线程消费数据，那么这个吞吐量太低了。而多个线程并发的话，顺序可能就乱掉了。



<img src="https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/aHR0cHM6Lny9tbWJpei5xcGljLmNuL21tYml6X3BuZy9YQ0VUTG9YelRyaWNHbEtSTDc1OE45azVBTjdKS1A5c2ZtaWJpYWZabGlhdjRyNnVmU01meFZSckFpYU83bFN4S1lBTkF6aWFyQlQxcVhyVmt2dGhHdkVtN1FqZy82NDA_d3hfZm10PXBuZw" alt="img" style="zoom:80%;" />

#### 2. 解决方案

- 一个 topic，一个 partition，一个 consumer，内部单线程消费，单线程吞吐量太低，一般不会用这个。
- **写 N 个内存 queue，具有相同 key 的数据都到同一个内存 queue；然后对于 N 个线程，每个线程分别消费一个内存 queue 即可，这样就能保证顺序性**。

<img src="https://cdn.jsdelivr.net/gh/Jason-Wu-1999/blog.img/imgs/aHR0cHM6Ly9tbWJlei5xcGljLmNuL21tYml6X3BuZy9YQ0VUTG9YelRyaWNHbEtSTDc1OE45azVBTjdKS1A5c2ZEZU0zQWhpYVVScGZTS2hOSk5iZklwbFpzY210aWJmZlcySUxpYVloWFM2Z0hmSWNhOWlidHNoVWx3LzY0MD93eF9mbXQ9cG5n" alt="img" style="zoom: 80%;" />





> 一个topic中的一个分区只能被一个消费组中的一个消费者消费，可以防止消息的重复消费
>
> 如果消费组中的消费者个数大于分区的话，会浪费掉
>



# kafka消息丢失和重复消费的问题

### 生产方消息丢失

#### 问题：不同**ACK机制**下的问题

 **0**：Leader收到消息后立刻返回给Producer，消息可能还没刷盘，也还没有同步给Follower。此时如果Leader挂掉，消息就丢失了。

**1**：Leader将消息写入磁盘后，立刻返回ACK给Producer，消息可能还没同步Follower。此时如果Leader挂掉，选举新的Follower成为主分区，因为刚才没同步成功，所以消息丢失。

**all**：Leader将消息写入磁盘，并且同步给所有Follower，此时再返回ACK。这个方案也存在一个问题，如果在Leader准备返回ACK的时候挂掉，Producer没收到ACK认定为发送失败。此时又有2种情况

> （1）开启重试：会导致队列收到2条重复的消息，此时**需要Consumer端保证消费幂等**，后面会介绍；
>
> （2）不开启重试(不推荐)：网络问题、超时等原因导致发送失败，不会重新发送，**消息容易丢失**。

#### 解决方法：

- ACK参数设置为all 并开启重试
- 对性能要求不是极高的情况下，建议将ACK设置-1，我们只需要在 Consumer 端结合幂等性避免消息重复消费。



### 生产方重复

####  原因分析：

因为避免生产者消息丢失，在`all`情况下设置ACK没收到就重传，但是 实际上数据已经写入，只是ack在返回过程中出现问题，导致重复发送数据

#### 解决方法

在消费方保证幂等性即可



### 消费方消息丢失

 默认情况下，消费消息的步骤如下：

- 消费者主动调用`poll()`方法拉取消息
- 消费者拉取消息后，主动上报目前的消费进度到`Broker`端
- `Broker`端收到消费进度后，将该**分区**的消费进度持久化

一般来说，消费者所调用的`poll()`方法默认情况下是自动提交，通过参数`enable.auto.commit`配置，并==且并不是每`poll()`一次就提交一次==，而是每个时间段统一提交，通过参数`auto.commit.interval.ms`控制。

> 这里带来一个问题，如果消息不是每次`poll()`都提交，那么第一次`poll()`和第二次`poll()`不会拉到相同的数据么？因为`offset`并没有提交。
>
> 但是实际并不会这样，因为提交`offset`一是为了再均衡后将消费位移共享给其他客户端，二是为了消费者重启后，能获取上一次的消费进度。
>
> 而平时，消费者都会在内存中保留了每次`poll()`后的位移，因此每次调用`poll()`返回后，都会更新此位移。

#### **原因分析**：

如果当消息还没有消费完，就已经提交offset，然后在消费的过程中出问题了，就会导致消息丢失问题

#### 解决方法：

将自动提交改为**手动提交**，等业务处理完成之后再去手动提交这个offset



### 消费方重复消费

#### 原因分析

1. **Kafka异常重启**

kafka每个消息写进去，都有一个offset，代表他的序号。consumer消费数据之后，每隔一段时间，会把自己消费过的消息的offset提交一下提交到zookeeper，代表我已经消费过了。

下次要是重启，而offset未及时提交，则会从上次消费到的offset来继续消费。

2. **Kafka消息消费设置时长小于实际业务执行时长**

   Kafka消费者有两个配置参数：

   ```
   max.poll.interval.ms    //两次poll(拉取)操作允许的最大时间间隔。单位毫秒。默认值300000（5分钟）。
   ```

   两次poll超过此时间间隔，Kafka服务端会进行rebalance操作，导致客户端连接失效，无法提交offset信息，从而引发重复消费。

   ```
   max.poll.records   // 一次poll操作获取的消息数量。默认值50。
   ```

   如果每条消息处理时间超过60秒，那么一批消息处理时间将超过5分钟，从而引发poll超时，最终导致重复消费。

3. **再均衡导致消息重复消费**

   **再均衡触发的时机**

   - 消费者数量变化（增加、减少都会引起）
   - 分区数量减少
   - 主题创建  并且消费者订阅主题时使用的是正则表达式

   `Kafka`提供了消费者组的概念，然而对于消费者组来说，消费者客户端的数量是会动态变化的，而每个消费者则是对应着每个分区，同时在再均衡过程中，整个消费者组会变得不可用。

   在再均衡发生的时候，如果某个分区拉取了并处理了消息，但是还没有来得及提交，此时消费者客户端会丢失这些状态，并变得不可用，当再均衡完成后，又会重新拉取消息进行消费，也就发生了消息重复消费的情况。

   `Kafka`专门提供了再均衡监听器接口(`ConsumerRebalanceListener`)，用来解决这个问题。再均衡监听器主要在调用`subscribe()`方法的时候设置。其主要包含两个方法：

   ```java
   onPartitionsRevoked(Collection<TopicPartition> partitions);  //在再均衡开始之前和消费者停止读取消息之后被调用，可以通过这个回调来处理消费位移的提交,partitions 表示再均衡之前分配到的分区
   
   opPartitionsAssigend(Collection<TopicPartition> partitions): //在重新分配分区之后和消费者开始读取消费之前被调用,partitions 表示再均衡之后分配到的分区
   ```

#### 解决方法

在消费者方保证消费的**幂等性**

其实还是得结合业务来思考，我这⾥给⼏个思路：

- 比如你拿个数据要写库，你先根据主键查⼀下，如果这数据都有了，你就别插⼊了，update⼀下好吧
- 比如你是写redis，那没问题了，反正每次都是set，天然幂等性
- **比如你不是上⾯两个场景，那做的稍微复杂⼀点，你需要让⽣产者发送每条数据的时候，⾥⾯加⼀个全局唯⼀的id，类似订单id之类的东西，然后你这⾥消费到了之后，先根据这个id去⽐如redis⾥查⼀下，之前消费过吗？如果没有消费过，你就处理，然后这个id写redis。如果消费过了，那你就别处理了，保证别重复处理相同的消息即可。**

