# skiplist



https://www.jianshu.com/p/e5a516831ac2



https://zhuanlan.zhihu.com/p/113227225



https://blog.csdn.net/ict2014/article/details/17394259



跳表（Skiplist）是一个特殊的链表，相比一般的链表，有更高的查找效率，可比拟二叉查找树，平均期望的查找、插入、删除时间复杂度都是O(logn)，许多知名的开源软件（库）中的数据结构均采用了跳表这种数据结构。

- Redis中的有序集合zset
- LevelDB、RocksDB、HBase中Memtable
- ApacheLucene中的TermDictionary、Posting List

**跳表结构描述**

跳表可视为水平排列（Level）、垂直排列（Tower）的位置（Position，对Entry的访问抽象）的二维集合。每个Level是一个列表Si，每个Tower包含存储连续列表中相同Entry的位置，跳表的各个位置可以通过如下方式进行遍历。

- After(P)：返回和P在同一Level的后面的一个位置，若不存在则返回NULL。
- Before(P)：返回和P在同一Level的前面的一个位置，若不存在则返回NULL。
- Below(P):返回和P在同一Tower的下面的一个位置，若不存在则返回NULL。
- Above(P)：返回和P在同一Tower的上面的一个位置，若不存在则返回NULL。



 ![img](https://pic4.zhimg.com/80/v2-1cc963c19eb34db31c47ef69a15f6863_720w.jpg) 





## 六.Redis为什么用skiplist而不用平衡树？

**这里从内存占用、对范围查找的支持和实现难易程度这三方面总结的原因。**

 1) 也不是非常耗费内存，实际上取决于生成层数函数里的概率 p，取决得当的话其实和平衡树差不多。 

 2) 因为有序集合经常会进行 ZRANGE 或 ZREVRANGE 这样的范围查找操作，跳表里面的双向链表可以十分方便地进行这类操作。 

 3) 实现简单，ZRANK 操作还能达到 o(logn)的时间复杂度O

## 四.skiplist与平衡树、哈希表的比较

- skiplist和各种平衡树（如AVL、红黑树等）的元素是有序排列的，而哈希表不是有序的。因此，在哈希表上只能做单个key的查找，不适宜做范围查找。所谓范围查找，指的是查找那些大小在指定的两个值之间的所有节点。
- 在做范围查找的时候，平衡树比skiplist操作要复杂。在平衡树上，我们找到指定范围的小值之后，还需要以中序遍历的顺序继续寻找其它不超过大值的节点。如果不对平衡树进行一定的改造，这里的中序遍历并不容易实现。而在skiplist上进行范围查找就非常简单，只需要在找到小值之后，对第1层链表进行若干步的遍历就可以实现。
- 平衡树的插入和删除操作可能引发子树的调整，逻辑复杂，而skiplist的插入和删除只需要修改相邻节点的指针，操作简单又快速。
- 从内存占用上来说，skiplist比平衡树更灵活一些。一般来说，平衡树每个节点包含2个指针（分别指向左右子树），而skiplist每个节点包含的指针数目平均为1/(1-p)，具体取决于参数p的大小。如果像Redis里的实现一样，取p=1/4，那么平均每个节点包含1.33个指针，比平衡树更有优势。
- 查找单个key，skiplist和平衡树的时间复杂度都为O(log n)，大体相当；而哈希表在保持较低的哈希值冲突概率的前提下，查找时间复杂度接近O(1)，性能更高一些。所以我们平常使用的各种Map或dictionary结构，大都是基于哈希表实现的。
- 从算法实现难度上来比较，skiplist比平衡树要简单得多。







# Redis要使用跳表来实现zset

https://cloud.tencent.com/developer/article/1666793

分治思想检索的一般都是对数级别的效率，这里就不展开了。

解决了这两个问题这个跳表完全可以替代平衡搜索树来来用。其实个人理解跳表跟树在思想上也有共同之处。只是相同道之下的不同术。

至于为什么Redis不用平衡搜索树来做，结合Redis作者的话和我自己的理解，首先我认为这么做挺好的，确实在保证底线（最差）的情况下在某些时候还有亮点。

1）内存占用更少，自定义参数化决定使用多少内存

2）有序集合很多时候用zrange 或者zrevrange 。性能至少不比平衡树差

3）简单更容易实现和维护

这个跳表其实小结下来不是什么新鲜东西，但是有很多值得学习和借鉴的地方。

#### 一句话总结什么是跳表：跳表就是在有序链表的基础上通过增加额外的指针节点来解决查询效率，通过随机插入来提高变更效率的一种数据结构。

一句话总结为什么Redis要用跳表来实现zset：内存使用更少，简单易维护，性能不比搜索树差