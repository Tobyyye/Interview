**QPS（Query Per Second）：每秒请求数，就是说服务器在一秒的时间内处理了多少个请求。**



https://zhuanlan.zhihu.com/p/84012183

https://www.jianshu.com/p/b57fb501505c



# 并发量

并发我们都听的很多，但是他还有个哥哥叫并行。

- **并发：一段时间访问的大量用户的请求**
- **并行：同一时刻的大量用户的请求**

并发是最能体现你的代码和机器的性能。





# QPS每秒查询率

每秒查询率QPS是对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准。

每秒查询率

因特网上，经常用每秒查询率来衡量域名系统服务器的机器的性能，其即为QPS。

对应fetches/sec，即每秒的响应请求数，也即是最大吞吐能力。



https://blog.csdn.net/zsj777/article/details/80408884



一．系统吞度量要素：

  一个系统的吞度量（承压能力）与request对CPU的消耗、外部接口、IO等等紧密关联。

单个reqeust 对CPU消耗越高，外部系统接口、IO影响速度越慢，系统吞吐能力越低，反之越高。

系统吞吐量几个重要参数：QPS（TPS）、并发数、响应时间

     QPS（Query Per Second），QPS 其实是衡量吞吐量（Throughput）的一个常用指标，就是说服务器在一秒的时间内处理了多少个请求 —— 我们通常是指 HTTP 请求，显然数字越大代表服务器的负荷越高、处理能力越强。作为参考，一个有着简单业务逻辑（包括数据库访问）的程序在单核心运行时可以提供 50 - 100 左右的 QPS，即每秒可以处理 50 - 100 个请求。

   TPS(Transaction Per Second) 每秒钟系统能够处理的事务的数量。
        QPS（TPS）：每秒钟 request/事务 的数量  （此处 / 表示 或的意思）

        并发数： 系统同时处理的request/事务数  （此处 / 表示 或的意思）
    
        响应时间：  一般取平均响应时间

理解了上面三个要素的意义之后，就能推算出它们之间的关系：

QPS（TPS）= 并发数/平均响应时间

        一个系统吞吐量通常由QPS（TPS）、并发数两个因素决定，每套系统这两个值都有一个相对极限值，在应用场景访问压力下，只要某一项达到系统最高值，系统的吞吐量就上不去了，如果压力继续增大，系统的吞吐量反而会下降，原因是系统超负荷工作，上下文切换、内存等等其它消耗导致系统性能下降。
    
     并发用户数和QPS两个概念没有直接关系，但是如果要说QPS时，一定需要指明是多少并发用户数下的QPS，否则毫无意义，因为单用户数的400QPS和200并发用户数下的400QPS是两个不同的概念。前者说明该应用可以在一秒内串行执行400个请求，而后者说明在并发200个请求的情况下，一秒内该应用能处理400个请求，当QPS相同时，越大的并发用户数，代表了网站并发处理能力越好。对于当前的web服务器，其处理单个用户的请求肯定戳戳有余，这个时候会存在资源浪费的情况（一方面该服务器可能有多个cpu，但是只处理单个进程，另一方面，在处理一个进程中，有些阶段可能是IO阶段，这个时候会造成CPU等待，但是有没有其他请求进程可以被处理）。而当并发数设置的过大时，每秒钟都会有很多请求需要处理，会造成进程（线程）频繁切换，反正真正用于处理请求的时间变少，每秒能够处理的请求数反而变少，同时用户的请求等待时间也会变大，甚至超过用户的心理底线。所以在最小并发数和最大并发数之间，一定有一个最合适的并发数值，在该并发数下，QPS能够达到最大。但是，这个并发并非是一个最佳的并发，因为当QPS到达最大时的并发，可能已经造成用户的等待时间变得超过了其最优值，所以对于一个系统，其最佳的并发数，一定需要结合QPS，用户的等待时间来综合确定。



决定系统响应时间要素

系统一次调用的响应时间有一条关键路径，这个关键路径是就是系统影响时间；

关键路径是有CPU运算、IO、外部系统响应等等组成。


