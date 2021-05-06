# kafka架构内部原理 工作流程



https://blog.csdn.net/qq_41594698/article/details/90728255

1.使用场景
Kafka是一个分布式消息队列。在流式计算中，Kafka一般用来缓存数据，Storm通过消费Kafka的数据进行计算。
Kafka对消息保存时根据Topic进行归类，发送消息者称为Producer，消息接受者称为Consumer，此外kafka集群有多个kafka实例组成，每个Server实例称为broker。

2.成份解析
其作用的位置就是下图的队列：

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190601130148272.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNTk0Njk4,size_16,color_FFFFFF,t_70) 

可以看到，其有两种模式：

1.点对点模式（一对一，消费者主动拉取数据，消息收到后消息清除）
点对点模型通常是一个基于拉取或者轮询的消息传送模型，这种模型从队列中请求信息，而不是将消息推送到客户端。这个模型的特点是发送到队列的消息被一个且只有一个接收者接收处理，即使有多个消息监听者也是如此。
2.发布/订阅模式（一对多，数据生产后，推送给所有订阅者）
发布订阅模型则是一个基于推送的消息传送模型。发布订阅模型可以有多种不同的订阅者，临时订阅者只在主动监听主题时才接收消息，而持久订阅者则监听主题的所有消息，即使当前订阅者不可用，处于离线状态。

3.工作流程分析
总流程图：

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190601130403434.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNTk0Njk4,size_16,color_FFFFFF,t_70) 


3.1 生产过程分析
3.1.1 写入方式
producer采用推（push）模式将消息发布到broker，每条消息都被追加（append）到分区（patition）中，属于顺序写磁盘（顺序写磁盘效率比随机写内存要高，保障kafka吞吐率）。

3.1.2 分区（Partition）
消息发送时都被发送到一个topic，其本质就是一个目录，而topic是由一些Partition Logs(分区日志)组成，其组织结构如下图所示：

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/2019060113045417.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNTk0Njk4,size_16,color_FFFFFF,t_70) 

我们可以看到，每个Partition中的消息都是有序的，生产的消息被不断追加到Partition log上，其中的每一个消息都被赋予了一个唯一的offset值。

3.1.2.1 分区的原因
（1）方便在集群中扩展，每个Partition可以通过调整以适应它所在的机器，而一个topic又可以有多个Partition组成，因此整个集群就可以适应任意大小的数据了；
（2）可以提高并发，因为可以以Partition为单位读写了。不同consumer可以读取不同的partition

3.1.2.2 分区的原则
即消息怎么被决定放在哪个分区：
（1）指定了patition，则直接使用；
（2）未指定patition但指定key，通过对key的value进行hash出一个patition
（3）patition和key都未指定，使用轮询选出一个patition。如按0、1、2顺序放置
其源码如下：
DefaultPartitioner类：

public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
    List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
    int numPartitions = partitions.size();
    if (keyBytes == null) {//3
        int nextValue = nextValue(topic);
        List<PartitionInfo> availablePartitions = cluster.availablePartitionsForTopic(topic);
        if (availablePartitions.size() > 0) {
            //对分区数量取模
            int part = Utils.toPositive(nextValue) % availablePartitions.size();
            return availablePartitions.get(part).partition();
        } else {
            // no partitions are available, give a non-available partition
            return Utils.toPositive(nextValue) % numPartitions;
        }
    } else {//2
        // hash the keyBytes to choose a partition
        return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
    }
}

3.1.3 写入流程



 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190601130820651.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNTk0Njk4,size_16,color_FFFFFF,t_70) 

1）producer先从zookeeper的 "/brokers/…/state"节点找到该partition的leader
2）producer将消息发送给该leader
3）leader将消息写入本地log
4）followers从leader pull消息，写入本地log后向leader发送ACK
5）leader收到所有ISR中的replication的ACK后，增加HW（high watermark，最后commit 的offset）并向producer发送ACK(默认为all，也可以配置为其他模式，如1，则leader完成就行，为0则发送了之后就算完成，不管是否收到)

3.2 broker保存消息
3.2.1 存储方式
物理上把topic分成一个或多个patition（对应 server.properties 中的num.partitions=3配置），每个patition物理上对应一个文件夹（该文件夹存储该patition的所有消息和索引文件）

3.2.2 存储策略
无论消息是否被消费，kafka都会保留所有消息。有两种策略可以删除旧数据：
1）基于时间：log.retention.hours=168
2）基于大小：log.retention.bytes=1073741824
需要注意的是，因为Kafka读取特定消息的时间复杂度为O(1)，即与文件大小无关，所以这里删除过期文件与提高 Kafka 性能无关。

3.3 消费过程分析
kafka提供了两套consumer API：高级Consumer API和低级API。

3.3.1 高级API
1）高级API优点
写起来简单
不需要去自行去管理offset，系统通过zookeeper自行管理
不需要管理分区，副本等情况，系统自动管理
消费者断线会自动根据上一次记录在zookeeper中的offset去接着获取数据（默认设置1分钟更新一下zookeeper中存的的offset）
可以使用group来区分对同一个topic 的不同程序访问分离开来（不同的group记录不同的offset，这样不同程序读取同一个topic才不会因为offset互相影响）
2）高级API缺点
不能自行控制offset（对于某些特殊需求来说）
不能细化控制如分区、副本、zk等

3.3.2 低级API
1）低级 API 优点
能够开发者自己控制offset，想从哪里读取就从哪里读取。
自行控制连接分区，对分区自定义进行负载均衡
对zookeeper的依赖性降低（如：offset不一定非要靠zk存储，自行存储offset即可，比如存在文件或者内存中）
2）低级API缺点
太过复杂，需要自行控制offset，连接哪个分区，找到分区leader 等。
具体代码可参考资料

3.3.3 消费者组
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190601131128983.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNTk0Njk4,size_16,color_FFFFFF,t_70) 

消费者是以consumer group消费者组的方式工作，由一个或者多个消费者组成一个组，共同消费一个topic。每个partition在同一时间只能由group中的一个消费者读取，但是 多个group可以同时消费这个partition。
在图中，有一个由三个消费者组成的group，有一个消费者读取主题中的两个分区，另外两个分别读取一个分区。某个消费者读取某个分区，也可以叫做某个消费者是某个分区的拥有者。
在这种情况下，消费者可以通过水平扩展的方式同时读取大量的消息。另外，如果一个消费者失败了，那么其他的group成员会自动负载均衡读取之前失败的消费者读取的分区。

3.3.4 消费方式
consumer采用pull（拉）模式从broker中读取数据。
push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由broker决定的。它的目标是尽可能以最快速度传递消息，但是这样很容易造成consumer来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而pull模式则可以根据consumer的消费能力以适当的速率消费消息。
对于Kafka而言，pull模式更合适，它可简化broker的设计，consumer可自主控制消费消息的速率，同时consumer可以自己控制消费方式——即可批量消费也可逐条消费，同时还能选择不同的提交方式从而实现不同的传输语义。
