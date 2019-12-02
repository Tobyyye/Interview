![img](https://images0.cnblogs.com/blog/497634/201309/08171028-a5e372741b18431591bb577b1e1c95e6.jpg)

在“[Java 集合系列01之 总体框架](http://www.cnblogs.com/skywang12345/p/3308498.html)”中，介绍java集合的架构。主体内容包括Collection集合和Map类；而Collection集合又可以划分为List(队列)和Set(集合)。

**1. List的实现类主要有: LinkedList, ArrayList, Vector, Stack。**

(01) [LinkedList](http://www.cnblogs.com/skywang12345/p/3308807.html)是双向链表实现的双端队列；它不是线程安全的，只适用于单线程。
(02) [ArrayList](http://www.cnblogs.com/skywang12345/p/3308556.html)是数组实现的队列，它是一个动态数组；它也不是线程安全的，只适用于单线程。
(03) [Vector](http://www.cnblogs.com/skywang12345/p/3308833.html)是数组实现的矢量队列，它也一个动态数组；不过和ArrayList不同的是，Vector是线程安全的，它支持并发。
(04) [Stack](http://www.cnblogs.com/skywang12345/p/3308852.html)是Vector实现的栈；和Vector一样，它也是线程安全的。

 

**2. Set的实现类主要有: HastSet和TreeSet。**

(01) [HashSet](http://www.cnblogs.com/skywang12345/p/3311252.html)是一个没有重复元素的集合，它通过HashMap实现的；HashSet不是线程安全的，只适用于单线程。
(02) [TreeSet](http://www.cnblogs.com/skywang12345/p/3311268.html)也是一个没有重复元素的集合，不过和HashSet不同的是，TreeSet中的元素是有序的；它是通过TreeMap实现的；TreeSet也不是线程安全的，只适用于单线程。

 

**3.Map的实现类主要有: HashMap，WeakHashMap, Hashtable和TreeMap。**

(01) [HashMap](http://www.cnblogs.com/skywang12345/p/3310835.html)是存储“键-值对”的哈希表；它不是线程安全的，只适用于单线程。
(02) [WeakHashMap](http://www.cnblogs.com/skywang12345/p/3311092.html)是也是哈希表；和HashMap不同的是，HashMap的“键”是强引用类型，而WeakHashMap的“键”是弱引用类型，也就是说当WeakHashMap 中的某个键不再正常使用时，会被从WeakHashMap中被自动移除。WeakHashMap也不是线程安全的，只适用于单线程。
(03) [Hashtable](http://www.cnblogs.com/skywang12345/p/3310887.html)也是哈希表；和HashMap不同的是，Hashtable是线程安全的，支持并发。
(04) [TreeMap](http://www.cnblogs.com/skywang12345/p/3310928.html)也是哈希表，不过TreeMap中的“键-值对”是有序的，它是通过R-B Tree(红黑树)实现的；TreeMap不是线程安全的，只适用于单线程。
更多关于这些集合类的介绍，可以参考“Java 集合系列目录(Category)”。