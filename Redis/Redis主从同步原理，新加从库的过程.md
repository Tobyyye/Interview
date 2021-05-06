新加从库的过程：

## **概述**

在现有企业中80%公司大部分使用的是redis单机服务，在实际的场景当中单一节点的redis容易面临风险。

### **面临问题**

\1. 机器故障。我们部署到一台 Redis 服务器，当发生机器故障时，需要迁移到另外一台服务器并且要保证数据是同步的。而数据是最重要的，如果你不在乎，基本上也就不会使用 Redis 了。

\2. 容量瓶颈。当我们有需求需要扩容 Redis 内存时，从 16G 的内存升到 64G，单机肯定是满足不了。当然，你可以重新买个 128G 的新机器。

### **解决办法**

要实现分布式数据库的更大的存储容量和承受高并发访问量，我们会将原来集中式数据库的数据分别存储到其他多个网络节点上。

> Redis 为了解决这个单一节点的问题，也会把数据复制多个副本部署到其他节点上进行复制，实现 Redis的高可用，实现对数据的冗余备份，从而保证数据和服务的高可用。

## **主从复制**

### **什么是主从复制**

![img](https://pic3.zhimg.com/80/v2-b2757dd9bfa4b8d408e4f3342cc489ca_720w.jpg)

主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点(master)，后者称为从节点(slave),数据的复制是单向的，只能由主节点到从节点。

默认情况下，每台Redis服务器都是主节点；且一个主节点可以有多个从节点(或没有从节点)，但一个从节点只能有一个主节点。

### **主从复制的作用**

\1. 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。

\2. 故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。

\3.  负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。

\4. 读写分离：可以用于实现读写分离，主库写、从库读，读写分离不仅可以提高服务器的负载能力，同时可根据需求的变化，改变从库的数量；

\5. 高可用基石：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。



## **主从复制启用**

从节点开启主从复制，有3种方式：

\1. 配置文件： 在从服务器的配置文件中加入：slaveof <masterip> <masterport>

\2. 启动命令： redis-server启动命令后加入 --slaveof <masterip> <masterport>

\3. 客户端命令： Redis服务器启动后，直接通过客户端执行命令：slaveof <masterip>
<masterport>，则该Redis实例成为从节点。

通过  info replication 命令可以看到复制的一些信息



https://zhuanlan.zhihu.com/p/112213386





## **主从复制过程

主从复制过程大体可以分为3个阶段：连接建立阶段（即准备阶段）、数据同步阶段、命令传播阶段。

在从节点执行 slaveof 命令后，复制过程便开始运作，下面图示大概可以看到，
从图中可以看出复制过程大致分为6个过程

![img](https://pic1.zhimg.com/80/v2-15ab2188df03b122bc9d5bee53874494_720w.jpg)

主从配置之后的日志记录也可以看出这个流程

1）保存主节点（master）信息。
执行 slaveof 后 Redis 会打印如下日志：

![img](https://pic1.zhimg.com/80/v2-ffc514a3de8e1d54c49e2323887694c0_720w.png)

2）从节点（slave）内部通过每秒运行的定时任务维护复制相关逻辑，当定时任务发现存在新的主节点后，会尝试与该节点建立网络连接

![img](https://pic2.zhimg.com/80/v2-49e5a86b9ae1ac0aa6814576bf1be5c1_720w.jpg)

从节点与主节点建立网络连接

从节点会建立一个 socket 套接字，从节点建立了一个端口为51234的套接字，专门用于接受主节点发送的复制命令。从节点连接成功后打印如下日志：

![img](https://pic4.zhimg.com/80/v2-0278db6dec9a5ba9fdc4ec92c79c9d9b_720w.png)

如果从节点无法建立连接，定时任务会无限重试直到连接成功或者执行 slaveof no one 取消复制

关于连接失败，可以在从节点执行 info replication 查看 master_link_down_since_seconds 指标，它会记录与主节点连接失败的系统时间。从节点连接主节点失败时也会每秒打印如下日志，方便发现问题：

\# Error condition on socket **for** SYNC: {socket_error_reason}

3）发送 ping 命令。
连接建立成功后从节点发送 ping 请求进行首次通信，ping 请求主要目的如下：
·检测主从之间网络套接字是否可用。
·检测主节点当前是否可接受处理命令。
如果发送 ping 命令后，从节点没有收到主节点的 pong 回复或者超时，比如网络超时或者主节点正在阻塞无法响应命令，从节点会断开复制连接，下次定时任务会发起重连。

![img](https://pic2.zhimg.com/80/v2-8ea529b55295292de2a02e109d0926a5_720w.jpg)

![img](https://pic3.zhimg.com/80/v2-bbacb788301fada16c374480ba98abc2_720w.jpg)

从节点发送的 ping 命令成功返回，Redis 打印如下日志，并继续后续复制流程：

![img](https://pic4.zhimg.com/80/v2-4189291485c0128eb6003b7827a5d933_720w.png)

4）权限验证。如果主节点设置了 requirepass 参数，则需要密码验证，从节点必须配置 masterauth 参数保证与主节点相同的密码才能通过验证；如果验证失败复制将终止，从节点重新发起复制流程。

5）同步数据集。主从复制连接正常通信后，对于首次建立复制的场景，主节点会把持有的数据全部发送给从节点，这部分操作是耗时最长的步骤。

6）命令持续复制。当主节点把当前的数据同步给从节点后，便完成了复制的建立流程。接下来主节点会持续地把写命令发送给从节点，保证主从数据一致性。











https://blog.csdn.net/qq_41724691/article/details/86616266

2.初次全量同步

当一个redis服务器初次向主服务器发送salveof命令时，redis从服务器会进行一次全量同步，同步的步骤如下图所示：

在这里插入图片描述

        slave服务器向master发送psync命令（此时发送的是psync ? -1），告诉master我需要同步数据了。
        master接收到psync命令后会进行BGSAVE命令生成RDB文件快照。
        生成完后，会将RDB文件发送给slave。
        slave接收到文件会载入RDB快照，并且将数据库状态变更为master在执行BGSAVE时的状态一致。
        master会发送保存在缓冲区里的所有写命令，告诉slave可以进行同步了
        slave执行这些写命令。


 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190124133223141.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzI0Njkx,size_16,color_FFFFFF,t_70) 



**3.命令传播**

slave已经同步过master了，那么如果后续master进行了写操作，比如说一个简单的set name redis，那么master执行过当前命令后，会将当前命令发送给slave执行一遍，达成数据一致性。







 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190123192757690.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzI0Njkx,size_16,color_FFFFFF,t_70) 





