两者分析

RDB持久化方式能够在指定的时间间隔对内存中的数据进行快照存储；

AOF持久化方式记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以redis协议追加保存每次写的操作到文件末尾；

Redis还能够对AOF文件进行后台重写，使aof文件体积不至于过大；

如果你只是用作缓存，只希望数据在程序运行的时候存在，那么就可以不使用任何持久化方式；

两种方式同时开启：在两种方式同时开启的情况下，redis启动的时候会优先加载aof文件来恢复原始数据，因为在通常情况下aof文件保存的数据集比rdb文件保存的数据集要完整，rdb的数据会不实时。那要不要只使用aof呢？建议不要，因为rdb更适合用于备份数据库（aof在不断变化，不好备份），快速重启，而不会有aof可能存在的潜在bug，留着作为一个补救的手段；
性能建议

因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留save 900 1这条规则。

如果Enalbe AOF，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了。代价一是带来了持续的IO，二是AOF rewrite的最后将rewrite过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上。默认超过原大小100%大小时重写可以改到适当的数值。

如果不Enable AOF ，仅靠Master-Slave Replication 实现高可用性也可以。能省掉一大笔IO也减少了rewrite时带来的系统波动。代价是如果Master/Slave同时倒掉，会丢失十几分钟的数据，启动脚本也要比较两个Master/Slave中的RDB文件，载入较新的那个。新浪微博就选用了这种架构



### AOF和RDB的区别

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020050319263469.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FtcW0zMw==,size_16,color_FFFFFF,t_70) 

RDB和AOF的选择之感

    对数据非常敏感，建议使用默认的AOF持久化方案
    AOF持久化策略使用erverysecond，每秒钟fsync一次。该策略redis任然可以保持很好的处理性能，当出现问题时，最多丢失0-1秒中的数据。
    注意：由于AOF文件存储体积较大，且恢复数据较慢
    数据呈现阶段有效性，建议使用RDB持久化方案
    数据可以良好的做到阶段内无丢失（该阶段是开发者或运维人工手工维护的），且恢复速度较快，阶段点数据恢复通常采用RDB方案
    注意：利用RDB实现紧凑的数据持久化会使Redis降得很低
    综合对比
    
    RDB与AOF得选择实际上是在做一种权衡，每种都有利弊
    如不能承受数分钟以内得数据丢失，对业务数据非常敏感，选用AOF
    如能承受数分钟以内数据丢失，且追求大数据集得恢复速度，选用RDB
    灾难恢复选用RDB
    双保险策略，同时开启RDB和AOF，重启后，Redis优先使用AOF来恢复数据，降低丢失数据的量
