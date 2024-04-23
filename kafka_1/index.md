# Kafka概述


<!--more-->



## 定义

* **Kafka传统定义**: Kafka 是一个**分布式**的基于**发布/订阅模式**的**消息队列**(Message Queue)
* **发布/订阅**: 消息的发布者不会讲消息直接发送给特定的订阅者, 而是**将发布的消息分为不同的类型, 订阅者只接受感兴趣的消息**. 
* **Kafka最新定义**: Kafka是一个开元的**分布式事件流平台**(Event Streaming Platform), 被数千家公司用于高性能**数据管道**、**流分析**、**数据集成**和**关键任务应用**. 



## 消息队列

目前比较常见的消息队列产品主要有Kafka、ActiveMQ、RabbitMQ、RocketMQ等. 

大数据场景主要采用 Kafka, JavaEE 开发中主要采用 ActiveMQ、RabbitMQ、RocketMQ . 



### 传统消息队列的应用场景

* **缓存/消峰**: 有助于控制和优化数据流经过系统的速度, 解决生产消息和消费消息的处理速度不一致的情况. 
* **解耦**: 允许独立的扩展或修改两边的处理过程, 只要确保它们遵守同样的接口约束. 
* **异步通信**: **允许用户把一个消息放入队列, 但并不立即处理它**, 然后**在需要的时候再去处理它们**. 



### 消息队列的两种模式

1. **点对点模式**: 消费者主动拉取数据, 消息收到后清除消息
2. **发布/订阅模式**: 
   * 可以有多个 topic(主题) (浏览、点赞、收藏、评论等)
   * 消费者消费数据之后, 不能删除数据
   * 每个消费者相互独立, 都可以消费到数据



## Kafka基础框架

<center>     <img style="border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"      src="https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/202207261603195.png" width = "65%" alt="" onclick="window.open(this.src)"/>     <br>     <div style="color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;">       Kafka基础框架   	</div> </center>

1. **Producer**: 消息生产者, 向 Kafka broker 发消息的客户端. 
2. **Consumer**: 消息消费者, 向 Kafka broker 取消息的客户端. 
3. **Consumer Group(CG)**: 消费者组, 由多个 consumer 组成. 消费者组内每个消费者负责消费不同分区的数据, 一个分区只能由一个组内消费者消费; 消费者组之间互不影响. 所有的消费者都属于某个消费者组, 即消费者组是逻辑上的一个订阅者. 
4. **Broker**: 一台 Kafka 服务器就是一个 broker. 一个集群由多个 broker 组成. 一个 broker 可以容纳多个topic. 
5. **Topic**: 可以理解为一个队列, 生产者和消费者面向的都是一个 topic. 
6. **Partition**: 为了实现扩展性, 一个非常大的 topic 可以分布到多个 broker(即服务器) 上, 一个 topic 可以分为多个 partition, 每个 partition 是一个有序的队列. 
7. **Replica**: 副本. 一个 topic 的每个分区都有若干个副本, 一个 Leader 和若干个 Follower. 
8. **Leader**: 每个分区多个副本的"主", 生产者发送数据的对象, 以及消费者消费数据的对象都是Leader. 
9. **Follower**: 每个分区多个副本中的"从", 实时从 Leader 中同步数据, 保持和 Leader 数据的同步. Leader 发生故障时, 某个 Follower 会成为新的 Leader. 

