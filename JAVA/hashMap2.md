https://blog.nowcoder.net/n/f43edaf2ec97412e84871bc5bf053a58



# HashMap

## 1.hashmap中的hash是什么？

把任意长度的输入（又叫做预映射pre-image）通过散列算法变换成固定长度的输出。hash算***可能碰到两个不同的key值经过hash算***算出同样的hash值，这就是hash冲突。
hash冲突可以避免吗？
理论上是不可能避免的，抽屉原理：有十个苹果但是有9个抽屉。最终一定会有一个抽屉的苹果数量大于1.
hashmap中的散列表是懒加载机制，只要第一次put数据才创建。
默认负载因子有啥用？
负载因子的作用是计算扩容阈值。比如默认16*0.75=12。
hash值是如何得到的？
是key的hashcode高16位异或低16位得到的新值，这是为了能散列化。



## 2. hashMap的组成结构

  答：数组+链表+红黑树
  每个数据单元都是一个node结构，node结构中有key字段、value字段、next字段还有hash字段。next字段就是发生hash冲突时候，当前slot中的node与冲突的node连成一个链表要用的字段。
  hashmap数组部分称为哈希桶，当链表长度达到8并且数组长度达到64才转为红黑树，长度未达到64则只会进行扩容操作，当红黑树节点数量小于等于6，转为链表。


## 3. 为什么要加入红黑树

  答：为了解决哈希冲突导致的链化严重的问题。jdk7的时候，当slot链化非常严重的时候，会严重影响get查询效率
本身散列列表的理想查询效率为O1，但是链化严重导致效率变成On。 



## 4. HashMap如何实现添加数据的(put)

  第一次调用putVal时会初始化hashmap中最耗费内存的散列表。先判断数组是否为空（table==null || table的length==0 ）为空进行初始化。这么做的目的是为了延迟初始化，减少内存消耗。因为如果hashmap new出来之后不放数据，就会浪费内存。只有放数据时才会初始化。
  第一种路由寻址找到的桶位slot = null，直接占用slot，然后把当前put方法传进来的key和value包装成一个node对象，扔进去就可以了。
  第二种路由寻址找到的桶位slot！=null，也就是里面已经有数据了，这时候又要分几种情况：
  （1） 引用的node还未链化，先对比一下这个node的key和当前put的key是否相等，如果相等就是替换操作，把新的替换旧的value，否则就是hash冲突，然后在slot的node后面追加一个node，1.8之后尾插法。
  （2） 桶slot内的nod已经链化了，首先迭代查找链表中的node和当前的key是否一致，一致就还是替换操作，替换当前node value新的value替换旧的，如果不是，那就是哈希冲突，把put包装的数据追加到链表尾部，再检查一下当前链表长度是否达到树化阈值，大于等于8或者元素个数大于64，如果达到就转换成红黑树。
  （3） hash冲突比较严重，链已经转化为红黑树了。创造树型节点直接插入红黑树。
  所有的插入完成之后，还需要判断当前节点数是否大于阈值，如果大于开始扩容为原数组的二倍。



## 5. HashMap如何实现获得数据(get)

  判断表或key是否是null，如果是直接返回null
  判断索引处第一个key与传入key是否相等，如果相等直接返回
  如果不相等，判断链表是否是红黑二叉树，如果是，直接从树中取值
如果不是树，就遍历链表查找



## 6. 什么是红黑树

它是一颗平衡二叉查找树
\- 每个节点非红即黑
\- 根节点总是黑色的
\- 如果节点是红色的，则它的子节点必须是黑色的（反之不一定）
\- 每个叶子节点都是黑色的空节点（NIL节点）

\- 从根节点到叶节点或空子节点的每条路径，必须包含相同数目的黑色节点（即相同的黑色高度）

## 7. HashMap中是如何实现扩容的

当数据量达到阈值之后触发扩容操作
table数组长度必须是2的次方，使用的左移移位运算，16--32--64

## 8. 为什么初始容量为16

  至于默认值为什么是16，而不是2 、4、8，或者32、64、1024等，我想应该就是个折中处理，过小会导致放不下几个元素，就要进行扩容了，而扩容是一个很消耗性能的操作。取值过大的话，无疑会浪费更多的内存空间。因此在日常开发中，如果可以预估HashMap会存入节点的数量，则应该在初始化时，指定其容量）
\9. 为什么负载因子为0.75.
  也是一个综合考虑，如果设置过小，HashMap的扩容频率会大大提升，而扩容操作会消耗大量的性能。如果设置过大的话，如果设成1，发生hash冲突的概率又会非常非常高。





## 12.HashMap，LinkedHashMap，TreeMap 有什么区别？



HashMap 参考其他问题；



LinkedHashMap 保存了记录的插入顺序，在用 Iterator 遍历时，先取到的记录肯定是先插入的；遍历比 HashMap 慢；



TreeMap 实现 SortMap 接口，能够把它保存的记录根据键排序（默认按键值升序排序，也可以指定排序的比较器）

## 13.HashMap & TreeMap & LinkedHashMap 使用场景？



一般情况下，使用最多的是 HashMap。



HashMap：在 Map 中插入、删除和定位元素时；



TreeMap：在需要按自然顺序或自定义顺序遍历键的情况下；



LinkedHashMap：在需要输出的顺序和输入的顺序相同的情况下。