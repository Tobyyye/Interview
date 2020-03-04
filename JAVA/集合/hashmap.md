HashMap基于哈希表的 Map 接口的实现。HashMap中，null可以作为键，这样的键只有一个；可以有一个或多个键所对应的值为null。HashMap中hash数组的默认大小是16，而且一定是2的指数。Hashtable、HashMap都使用了 Iterator。而由于历史原因，Hashtable还使用了Enumeration的方式 。HashMap 实现 Iterator，支持fast-fail。

哈希表是由数组+链表组成的，它是通过把key值进行hash来定位对象的，这样可以提供比线性存储更好的性能。

![img](https:////upload-images.jianshu.io/upload_images/14326004-62a3c57daa0dcab2?imageMogr2/auto-orient/strip|imageView2/2/w/1024/format/webp)

image



HashMap不是线程安全的。