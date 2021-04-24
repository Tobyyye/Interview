**10、HashTable**

- 数组 + 链表方式存储

- 默认容量： 11(质数 为宜)

- put:

- - 索引计算 : （key.hashCode() & 0x7FFFFFFF）% table.length；
  - 若在链表中找到了，则替换旧值，若未找到则继续；
  - 当总元素个数超过容量*加载因子时，扩容为原来 2 倍并重新散列；
  - 将新元素加到链表头部。

- 对修改 Hashtable 内部共享数据的方法添加了 synchronized，保证线程安全。



**11、HashMap ，HashTable 区别**

- 默认容量不同，扩容不同；
- 线程安全性，HashTable 安全 ；
- 效率不同 HashTable 要慢因为加锁。