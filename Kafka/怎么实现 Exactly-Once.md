https://blog.csdn.net/lzufeng/article/details/81906031

# Kafka Exactly Once

Kafka一致性级别：

名称	设置	一致性
At Least Once	acks=-1	保证数据不丢失，但不能保证数据不重复
At Most Once	acks=0	保证数据不重复，但不能保证数据不丢失
Exactly Once	acks=-1 + enable.idompotence=true	精准一致性，保证数据生产不丢失不重复



## 消息系统语义概述（Overview of messaging system semantics）

​    在一个分布式发布订阅消息系统中，组成系统的计算机总会由于各自的故障而不能工作。在Kafka中，一个单独的broker，可能会在生产者发送消息到一个topic的时候宕机，或者出现网络故障，从而导致生产者发送消息失败。根据生产者如何处理这样的失败，产生了不同的语义：

- **至少一次语义**（At least once semantics）：如果生产者收到了Kafka broker的确认（acknowledgement，ack），并且生产者的acks配置项设置为all（或-1），这就意味着消息已经被精确一次写入Kafka topic了。然而，如果生产者接收ack超时或者收到了错误，它就会认为消息没有写入Kafka topic而尝试重新发送消息。如果broker恰好在消息已经成功写入Kafka topic后，发送ack前，出了故障，生产者的重试机制就会导致这条消息被写入Kafka两次，从而导致同样的消息会被消费者消费不止一次。每个人都喜欢一个兴高采烈的给予者，但是这种方式会导致重复的工作和错误的结果。
- **至多一次语义**（At most once semantics）：如果生产者在ack超时或者返回错误的时候不重试发送消息，那么消息有可能最终并没有写入Kafka topic中，因此也就不会被消费者消费到。但是为了避免重复处理的可能性，我们接受有些消息可能被遗漏处理。
- **精确一次语义**（Exactly once semantics）：即使生产者重试发送消息，也只会让消息被发送给消费者一次。精确一次语义是最令人满意的保证，但也是最难理解的。因为它需要消息系统本身和生产消息的应用程序还有消费消息的应用程序一起合作。比如，在成功消费一条消息后，你又把消费的offset重置到之前的某个offset位置，那么你将收到从那个offset到最新的offset之间的所有消息。这解释了为什么消息系统和客户端程序必须合作来保证精确一次语义。

https://blog.csdn.net/w1992wishes/article/details/89502956



## 必须被处理的故障（Failures that must be handled）

​    为了描述为了支持精确一次消息投递语义而引入的挑战，让我们从一个简单的例子开始。
​    假设有一个单进程生产者程序，发送了消息“Hello Kafka“给一个叫做“EoS“的单分区Kafka topic。然后有一个单实例的消费者程序在另一端从topic中拉取消息，然后打印。在没有故障的理想情况下，这能很好的工作，“Hello Kafka“只被写入到EoS topic一次。消费者拉取消息，处理消息，提交偏移量来说明它完成了处理。然后，即使消费者程序出故障重启也不会再收到“Hello Kafka“这条消息了。
​    然而，我们知道，我们不能总认为一切都是顺利的。在上规模的集群中，即使最不可能发生的故障场景都可能最终发生。比如：

1. **broker可能故障**：Kafka是一个高可用、持久化的系统，每一条写入一个分区的消息都会被持久化并且多副本备份（假设有n个副本）。所以，Kafka可以容忍n-1个broker故障，意味着一个分区只要至少有一个broker可用，分区就可用。Kafka的副本协议保证了只要消息被成功写入了主副本，它就会被复制到其他所有的可用副本（ISR）。
2. **producer到broker的RPC调用可能失败**：Kafka的持久性依赖于生产者接收broker的ack。没有接收成功ack不代表生产请求本身失败了。broker可能在写入消息后，发送ack给生产者的时候挂了。甚至broker也可能在写入消息前就挂了。由于生产者没有办法知道错误是什么造成的，所以它就只能认为消息没写入成功，并且会重试发送。在一些情况下，这会造成同样的消息在Kafka分区日志中重复，进而造成消费端多次收到这条消息。
3. **客户端可能会故障**：精确一次交付也必须考虑客户端故障。但是我们如何知道一个客户端已经故障而不是暂时和brokers断开，或者经历一个程序短暂的暂停？区分永久性故障和临时故障是很重要的，为了正确性，broker应该丢弃僵住的生产者发送来的消息，同样，也应该不向已经僵住的消费者发送消息。一旦一个新的客户端实例启动，它应该能够从失败的实例留下的任何状态中恢复，从一个安全点开始处理。这意味着，消费的偏移量必须始终与生产的输出保持同步。





3.2.1、幂等性：每个分区中精确一次且有序（Idempotence: Exactly-once in order semantics per partition）
Kafka 在0.11.0.0之前的版本中只支持 At Least Once 和 At Most Once 语义，尚不支持 Exactly Once 语义。

Kafka 0.11.0.0版本引入了幂等语义。 一个幂等性的操作就是一种被执行多次造成的影响和只执行一次造成的影响一样的操作。

如果出现导致生产者重试的错误，同样的消息，仍由同样的生产者发送多次，将只被写到 Kafka broker 的日志中一次。

对于单个分区，幂等生产者不会因为生产者或 broker 故障而产生多条重复消息。

想要开启这个特性，获得每个分区内的精确一次语义，也就是说没有重复，没有丢失，并且有序的语义，只需要 producer 配置 enable.idempotence=true。

这个特性是怎么实现的呢？每个新的 Producer 在初始化的时候会被分配一个唯一的 PID，该PID对用户完全透明而不会暴露给用户。在底层，它和 TCP 的工作原理有点像，每一批发送到 Kafka 的消息都将包含 PID 和一个从 0 开始单调递增序列号。

Broker 将使用这个序列号来删除重复的发送。和只能在瞬态内存中的连接中保证不重复的 TCP 不同，这个序列号被持久化到副本日志，所以，即使分区的 leader 挂了，其他的 broker 接管了leader，新 leader 仍可以判断重新发送的是否重复了。这种机制的开销非常低：每批消息只有几个额外的字段。这种特性比非幂等的生产者只增加了可忽略的性能开销。

如果消息序号比 Broker 维护的序号大 1 以上，说明中间有数据尚未写入，也即乱序，此时 Broker 拒绝该消息。
如果消息序号小于等于 Broker 维护的序号，说明该消息已被保存，即为重复消息，Broker直接丢弃该消息。



**3.2.1、幂等性：**每个分区中精确一次且有序（Idempotence: Exactly-once in order semantics per partition）

Kafka 在0.11.0.0之前的版本中只支持 At Least Once 和 At Most Once 语义，尚不支持 Exactly Once 语义。

Kafka 0.11.0.0版本引入了幂等语义。 一个幂等性的操作就是一种被执行多次造成的影响和只执行一次造成的影响一样的操作。

如果出现导致生产者重试的错误，同样的消息，仍由同样的生产者发送多次，将只被写到 Kafka broker 的日志中一次。

对于单个分区，幂等生产者不会因为生产者或 broker 故障而产生多条重复消息。

想要开启这个特性，获得每个分区内的精确一次语义，也就是说没有重复，没有丢失，并且有序的语义，只需要 producer 配置 enable.idempotence=true。

这个特性是怎么实现的呢？每个新的 Producer 在初始化的时候会被分配一个唯一的 PID，该PID对用户完全透明而不会暴露给用户。在底层，它和 TCP 的工作原理有点像，每一批发送到 Kafka 的消息都将包含 PID 和一个从 0 开始单调递增序列号。

Broker 将使用这个序列号来删除重复的发送。和只能在瞬态内存中的连接中保证不重复的 TCP 不同，这个序列号被持久化到副本日志，所以，即使分区的 leader 挂了，其他的 broker 接管了leader，新 leader 仍可以判断重新发送的是否重复了。这种机制的开销非常低：每批消息只有几个额外的字段。这种特性比非幂等的生产者只增加了可忽略的性能开销。

如果消息序号比 Broker 维护的序号大 1 以上，说明中间有数据尚未写入，也即乱序，此时 Broker 拒绝该消息。

如果消息序号小于等于 Broker 维护的序号，说明该消息已被保存，即为重复消息，Broker直接丢弃该消息。

**总结来说，producer 端发送消息时，生成全局唯一自增pid，和broker中数据的pid进行对比，多则删除，少则通知producer端重新发送。**



————————————————

3.2.2、事务：跨分区原子写入（Transactions: Atomic writes across multiple partitions）
上述幂等设计只能保证单个 Producer 对于同一个 <Topic, Partition> 的 Exactly Once 语义。

Kafka 现在通过新的事务 API 支持跨分区原子写入。这将允许一个生产者发送一批到不同分区的消息，这些消息要么全部对任何一个消费者可见，要么对任何一个消费者都不可见。这个特性也允许在一个事务中处理消费数据和提交消费偏移量，从而实现端到端的精确一次语义。

为了实现这种效果，应用程序必须提供一个稳定的（重启后不变）唯一的 ID，也即Transaction ID 。 Transactin ID 与 PID 可能一一对应。区别在于 Transaction ID 由用户提供，将生产者的 transactional.id 配置项设置为某个唯一ID。而 PID 是内部的实现对用户透明。

另外，为了保证新的 Producer 启动后，旧的具有相同 Transaction ID 的 Producer 失效，每次 Producer 通过 Transaction ID 拿到 PID 的同时，还会获取一个单调递增的 epoch。由于旧的 Producer 的 epoch 比新 Producer 的 epoch 小，Kafka 可以很容易识别出该 Producer 是老的 Producer 并拒绝其请求。

下面是的代码片段演示了事务 API 的使用：

Producer<String, String> producer = new KafkaProducer<String, String>(props);
// 初始化事务，包括结束该Transaction ID对应的未完成的事务（如果有）
// 保证新的事务在一个正确的状态下启动
producer.initTransactions();
// 开始事务
producer.beginTransaction();
// 消费数据
ConsumerRecords<String, String> records = consumer.poll(100);
try{
    // 发送数据
    producer.send(new ProducerRecord<String, String>("Topic", "Key", "Value"));
    // 发送消费数据的Offset，将上述数据消费与数据发送纳入同一个Transaction内
    producer.sendOffsetsToTransaction(offsets, "group1");
    // 数据发送及Offset发送均成功的情况下，提交事务
    producer.commitTransaction();
} catch (ProducerFencedException | OutOfOrderSequenceException | AuthorizationException e) {
    // 数据发送或者Offset发送出现异常时，终止事务
    producer.abortTransaction();
} finally {
    // 关闭Producer和Consumer
    producer.close();
    consumer.close();
}
需要注意的是，上述的事务保证是从 Producer 的角度去考虑的



从 Consumer 的角度来看，该保证会相对弱一些。尤其是不能保证所有被某事务 Commit 过的所有消息都被一起消费，因为：

对于压缩的 Topic 而言，同一事务的某些消息可能被其它版本覆盖。
事务包含的消息可能分布在多个 Segment 中（即使在同一个 Partition内），当老的 Segment 被删除时，该事务的部分数据可能会丢失
Consumer 在一个事务内可能通过 seek 方法访问任意 Offset 的消息，从而可能丢失部分消息。
Consumer 可能并不需要消费某一事务内的所有 Partition，因此它将永远不会读取组成该事务的所有消息



https://bigdata.51cto.com/art/202104/656545.htm

**3.2.2、事务：跨分区原子写入**

上述幂等设计只能保证单个 Producer 对于同一个 <Topic, Partition> 的 Exactly Once 语义。

Kafka 现在通过新的事务 API 支持跨分区原子写入。这将允许一个生产者发送一批到不同分区的消息，这些消息要么全部对任何一个消费者可见，要么对任何一个消费者都不可见。这个特性也允许在一个事务中处理消费数据和提交消费偏移量，从而实现端到端的精确一次语义。

为了实现这种效果，应用程序必须提供一个稳定的（重启后不变）唯一的 ID，也即Transaction ID 。 Transactin ID 与 PID 可能一一对应。区别在于 Transaction ID 由用户提供，将生产者的 transactional.id 配置项设置为某个唯一ID。而 PID 是内部的实现对用户透明。

另外，为了保证新的 Producer 启动后，旧的具有相同 Transaction ID 的 Producer 失效，每次 Producer 通过 Transaction ID 拿到 PID 的同时，还会获取一个单调递增的 epoch。由于旧的 Producer 的 epoch 比新 Producer 的 epoch 小，Kafka 可以很容易识别出该 Producer 是老的 Producer 并拒绝其请求。

有了Transaction ID后，Kafka可保证：

- 跨Session的数据幂等发送。当具有相同Transaction ID的新的Producer实例被创建且工作时，旧的且拥有相同Transaction ID的Producer将不再工作。
- 跨Session的事务恢复。如果某个应用实例宕机，新的实例可以保证任何未完成的旧的事务要么Commit要么Abort，使得新实例从一个正常状态开始工作。

需要注意的是，上述的事务保证是从Producer的角度去考虑的。从Consumer的角度来看，该保证会相对弱一些。尤其是不能保证所有被某事务Commit过的所有消息都被一起消费，因为：

- 对于压缩的Topic而言，同一事务的某些消息可能被其它版本覆盖
- 事务包含的消息可能分布在多个Segment中（即使在同一个Partition内），当老的Segment被删除时，该事务的部分数据可能会丢失
- Consumer在一个事务内可能通过seek方法访问任意Offset的消息，从而可能丢失部分消息
- Consumer可能并不需要消费某一事务内的所有Partition，因此它将永远不会读取组成该事务的所有消息





**四、事务中Offset的提交**

许多基于Kafka的应用，尤其是Kafka Stream应用中同时包含Consumer和Producer，前者负责从Kafka中获取消息，后者负责将处理完的数据写回Kafka的其它Topic中。

为了实现该场景下的事务的原子性，Kafka需要保证对Consumer Offset的Commit与Producer对发送消息的Commit包含在同一个事务中。否则，如果在二者Commit中间发生异常，根据二者Commit的顺序可能会造成数据丢失和数据重复：

- 如果先Commit Producer发送数据的事务再Commit Consumer的Offset，即At Least Once语义，可能造成数据重复。
- 如果先Commit Consumer的Offset，再Commit Producer数据发送事务，即At Most Once语义，可能造成数据丢失。





**用于事务特性的控制型消息**

为了区分写入Partition的消息被Commit还是Abort，Kafka引入了一种特殊类型的消息，即`Control Message`。该类消息的Value内不包含任何应用相关的数据，并且不会暴露给应用程序。它只用于Broker与Client间的内部通信。

对于Producer端事务，Kafka以Control Message的形式引入一系列的`Transaction Marker`。Consumer即可通过该标记判定对应的消息被Commit了还是Abort了，然后结合该Consumer配置的隔离级别决定是否应该将该消息返回给应用程序。





 ![img](https://img2020.cnblogs.com/blog/434101/202006/434101-20200605105340716-1802126096.png) 









### **幂等的producer**

要介绍幂等的producer之前，得先了解一下幂等这个词是什么意思。幂等这个词最早起源于函数式编程，意思是一个函数无论执行多少次都会返回一样的结果。比如说让一个数加1就不是幂等的，而让一个数取整就是幂等的。因为这个特性所以幂等的函数适用于并发的场景下。

但幂等在分布式系统中含义又做了进一步的延申，比如在kafka中，幂等性意味着一个消息无论重复多少次，都会被当作一个消息来持久化处理。

kafka的producer默认是支持最少一次语义，也就是说不是幂等的，这样在一些比如支付等要求精确数据的场景会出现问题，在0.11.0后，kafka提供了让producer支持幂等的配置操作。即：

> props.put("enable.idempotence", ture)

在创建producer客户端的时候，添加这一行配置，producer就变成幂等的了。注意开启幂等性的时候，acks就自动是“all”了，如果这时候手动将ackss设置为0，那么会报错。

而底层实现其实也很简单，就是对每条消息生成一个id值，broker会根据这个id值进行去重，从而实现幂等，这样一来就能够实现精确一次的语义了。

但是！幂等的producery也并非万能。有两个主要是缺陷：

- 幂等性的producer仅做到单分区上的幂等性，即单分区消息不重复，多分区无法保证幂等性。
- 只能保持单会话的幂等性，无法实现跨会话的幂等性，也就是说如果producer挂掉再重启，无法保证两个会话间的幂等（新会话可能会重发）。因为broker端无法获取之前的状态信息，所以无法实现跨会话的幂等。

### **事务的producer**

当遇到上述幂等性的缺陷无法解决的时候，可以考虑使用事务了。事务可以支持多分区的数据完整性，原子性。并且支持跨会话的exactly once处理语义，也就是说如果producer宕机重启，依旧能保证数据只处理一次。

开启事务也很简单，首先需要开启幂等性，即设置enable.idempotence为true。然后对producer发送代码做一些小小的修改。

```text
//初始化事务
producer.initTransactions();
try {
    //开启一个事务
    producer.beginTransaction();
    producer.send(record1);
    producer.send(record2);
    //提交
    producer.commitTransaction();
} catch (KafkaException e) {
    //出现异常的时候，终止事务
    producer.abortTransaction();
}
```

但无论开启幂等还是事务的特性，都会对性能有一定影响，这是必然的。所以kafka默认也并没有开启这两个特性，而是交由开发者根据自身业务特点进行处理

# kafka事务性实现

https://blog.csdn.net/muyimo/article/details/91439222



## 事务工作原理

https://www.cnblogs.com/wangzhuxing/p/10125437.html

 ![img](https://img2018.cnblogs.com/blog/843808/201812/843808-20181216125439935-1003779001.png) 

