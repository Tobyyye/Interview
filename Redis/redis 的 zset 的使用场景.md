redis 的 zset 的使用场景？底层实现？为什么要用跳表？



# 1. 延时队列

zset 会按 score 进行排序，如果 score 代表想要执行时间的时间戳。在某个时间将它插入 zset 集合中，它变会按照时间戳大小进行排序，也就是对执行时间前后进行排序。



2. 排行榜

经常浏览技术社区的话，应该对 “1小时最热门” 这类榜单不陌生。如何实现呢？如果记录在数据库中，不太容易对实时统计数据做区分。我们以当前小时的时间戳作为 zset 的 key，把贴子ID 作为 member ，点击数评论数等作为 score，当 score 发生变化时更新 score。利用 ZREVRANGE 或者 ZRANGE 查到对应数量的记录。

记录回复数

代码如下：

    /**
      * 模拟每次针对贴子的回复数加 1
      *
      * @param id
    */
    public void post(long id) {
        Jedis client = jedisPool.getResource();
        client.zincrby(POSTLIST, 1, String.format(MEMBER_PREFIX, id));
        client.close();
    }

 

获取列表

代码如下：

    /**
      * 获取 Top 的贴子列表 ID
      *
      * @param size
      * @return
    */
    public List<Integer> getTopList(int size) {
        List<Integer> result = new ArrayList<>();
        if (size <= 0 || size > 100) {
            return result;
        }
        Jedis client = jedisPool.getResource();
        Set<Tuple> tupleSet = client.zrevrangeWithScores(POSTLIST, 0, size - 1);
        client.close();
        for (Tuple tuple : tupleSet) {
            String t = tuple.getElement().replaceAll("[^0-9]", "");
            result.add(Integer.valueOf(t));
        }
        return result;
    }




3. 限流

滑动窗口是限流常见的一种策略。如果我们把一个用户的 ID 作为 key 来定义一个 zset ，member 或者 score 可以都为访问时的时间戳。我们只需统计某个 key 下在指定时间戳区间内的个数，就能得到这个用户滑动窗口内访问频次，与最大通过次数比较，来决定是否允许通过。

滑动窗口

 代码如下：

    /**
      *
      * @param userId
      * @param period 窗口大小
      * @param maxCount 最大频次限制
      * @return
    */
    public boolean isActionAllowed(String userId, int period, int maxCount) {
        String key = String.format(KEY, userId);
        long nowTs = System.currentTimeMillis();
        Jedis client = jedisPool.getResource();
        Pipeline pipe = client.pipelined();
        pipe.multi();
        pipe.zadd(key, nowTs, String.format(MEMBER, userId, nowTs));
        pipe.zremrangeByScore(key, 0, nowTs - period * 1000);
        Response<Long> count = pipe.zcard(key);
        pipe.expire(key, period + 1);
        pipe.exec();
        pipe.close();
        client.close();
        return count.get() <= maxCount;
    }

思路是每一个请求到来时，将时间窗口外的记录全部清理掉，只保留窗口内的记录。zset 中只有 score 值非常重要，value 值没有特别的意义，只需要保证它是唯一的就可以了

问题

    需要清理额外的数据
    
    限制的请求量过大时，会占用大量内存


**底层实现**



### 1. 编码

zset的编码有**ziplist**和**skiplist**两种。
 底层分别使用**ziplist（压缩链表）**和**skiplist（跳表）**实现。

- ##### 什么时候使用ziplist什么时候使用skiplist？

当zset满足以下两个条件的时候，使用ziplist：

> 1. 保存的元素少于128个
> 2. 保存的所有元素大小都小于64字节

**不满足这两个条件则使用skiplist。**
 （注意：这两个数值是可以通过`redis.conf`的`zset-max-ziplist-entries` 和 `zset-max-ziplist-value`选项 进行修改。）

------

### 2. 实现

- ##### ziplist编码

> 关于什么是ziplist（压缩链表），可以参见这篇文章：[Redis源码分析-压缩列表ziplist](https://www.jianshu.com/p/afaf78aaf615)

ziplist 编码的有序集合对象使用压缩列表作为底层实现，每个集合元素使用两个紧挨在一起的压缩列表节点来保存，第一个节点保存元素的成员，第二个节点保存元素的分值。并且压缩列表内的集合元素按分值从小到大的顺序进行排列，小的放置在靠近表头的位置，大的放置在靠近表尾的位置。



![img](https:////upload-images.jianshu.io/upload_images/1038472-ea9ab3a8c73124f7.png?imageMogr2/auto-orient/strip|imageView2/2/w/542)

从小到大排列



![img](https:////upload-images.jianshu.io/upload_images/1038472-6f812227fb90b78d.png?imageMogr2/auto-orient/strip|imageView2/2/w/771)

ziplist结构





