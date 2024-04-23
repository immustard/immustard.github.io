# Kafka生产者


&lt;!--more--&gt;



## 生产者消息发送流程

### 发送原理

在消息发送的过程中, 涉及到了**两个线程--`main线程`和`Sender线程`**. 在 main 线程中创建了一个`双端队列RecordAccumulator`. main 线程将消息发送个 RecordAccumulator, Sender 线程不断从 RecordAccumulator 中拉取消息发送到 Kafka Broker. 

&lt;center&gt;     &lt;img style=&#34;border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);&#34;      src=&#34;https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/202209041729645.png&#34; width = &#34;65%&#34; alt=&#34;&#34; onclick=&#34;window.open(this.src)&#34;/&gt;     &lt;br&gt;     &lt;div style=&#34;color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;&#34;&gt;       发送流程   	&lt;/div&gt; &lt;/center&gt;



### 生产者重要参数

| **参数名称**                           | **描述**                                                     |
| -------------------------------------- | ------------------------------------------------------------ |
| `bootstrap.servers`                    | 生产者连接集群所需的 boker 地址清单. 可以设置一个或多个, 中间用`,`隔开. **注意:** 并非需要所有的 broker 地址, 因为生产者从给定的 broker 里查找其他 broker 信息 |
| `key.serializer`和`value.serializer`   | 指定发送消息的 key 和 value 的序列化类型. **一定要写全类名!** |
| `buffer.memory`                        | RecordAccumulator 缓冲区总大小, `默认32m`.                   |
| `batch.size`                           | 缓冲区一批数据最大值, `默认16k`. (适当增加可以提高吞吐量; 但是如果过大的话, 会导致数据传输延迟增加). |
| `linger.ms`                            | 如果数据一直未到达 batch.size, sender 等待 linger.ms 之后就会发送数据, 单位是ms. `默认0ms`, 表示没有延迟. (生产环境一般 5-100ms 之间) |
| `acks`                                 | 0: 生产者发送过来的数据, 不需要等数据落盘应答; 1: 生产者发送过来的数据, Leader 收到数据后应答; -1(all): 生产者发送过来的数据, Leader&#43; 和 isr 队列里面的所有节点收齐数据后应答. `默认是-1`, -1和all是等价的. |
| `max.in.flight.request.per.connection` | 允许最多没有返回 ack 的次数, `默认为5`, 开启幂等性要保证该值是 1-5 的数字. |
| `retries`                              | 当消息发送出现错误的时候, 系统会重发消息. 该值表示重复次数. `默认为int最大值,2147483647`. 如果设置了重试, 还想保证消息的有序性, 需要设置 MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION=1 否则在重试此失败消息的时候, 其他的消息可能发送成功了. |
| `retry.backoff.ms`                     | 两次重试之间的时间间隔, `默认为100ms`.                       |
| `enable.idempotence`                   | 是否开启幂等性, `默认为true`, 开启幂等性.                    |
| `compression.type`                     | 生产者发送的所有数据的压缩方式. `默认为none`, 也就是不压缩. 支持压缩类型: none, gzip, snappy, lz4和zstd. |



## 生产者分区

### 分区的好处

1. **便于合理使用存储资源**, 每个 Partition 在一个 Broker 上存储, 可以吧海量的数据按照分区切割成小块存储在多台 Broker 上. 合理控制分区的任务, 可以实现**负载均衡**的效果. 
2. **提高并行度**, 生产者可以以分区为单位**发送数据**; 消费者可以以分区为单位**消费数据**. 



## 数据传递语义

* **至少一次**(At Least Once) = `ACK级别设置为-1` &#43; `分区副本大于等于2` &#43; `ISR里应答的最小副本数量大于等于2`
* **最多一次**(At Most Once) = `ACK级别设置为0`. 
* **精确一次**(Exactly Once): 对于一些非常重要的信息, 例如和钱相关的数据, 要求数据既不能重复也不能丢失. 



也就是说, At Least Once 可以保证数据不丢失, 但是**不能保证数据不重复**; At Most Once 可以保证数据不重复, 但是**不能保证数据不丢失**. 



## 幂等性

### 幂等性原理

幂等性就是指 Producer 不论向 Broker 发送多少次重复数据, Broker 端都只会持久化一条, 保证了不重复. 

所以上面的 Exactly Once = `幂等性` &#43; `At Least Once`(ack=-1 &#43; partitions&gt;=2 &#43; ISR最小副本数&gt;=2)



**重复数据的判断标准**: 具有`&lt;PID, Partition, SeqNumber&gt;`相同逐渐的消息提交时, Borker 只会持久化一条. 其中: 

* PID 是 Kafka 每次重启都会分配一个新的; 
* Partition 表示分区号
* Sequence Number 单调自增

**综上, 幂等性只能保证的是在单分区单会话内不重复**.



### 如何使用幂等性

开启参数`enable.idempotence`. 



## 生产者事务

&gt; 开启事务, 必须开启幂等性!

&lt;center&gt;     &lt;img style=&#34;border-radius: 0.3125em;     box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);&#34;      src=&#34;https://cdn.jsdelivr.net/gh/immustard/gallery/pictures/202209041812162.png&#34; width = &#34;65%&#34; alt=&#34;&#34; onclick=&#34;window.open(this.src)&#34;/&gt;     &lt;br&gt;     &lt;div style=&#34;color:orange; border-bottom: 1px solid #d9d9d9;     display: inline-block;     color: #999;     padding: 2px;&#34;&gt;       Kafka事务原理   	&lt;/div&gt; &lt;/center&gt;


---

> 作者:   
> URL: https://buli-home.cn/kafka_3/  

