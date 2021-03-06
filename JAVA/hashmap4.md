https://zhuanlan.zhihu.com/p/62854712



## 







1、对Key求Hash值，然后再计算下标；

2、如果没有碰撞，直接放入桶中（碰撞的意思是计算得到的Hash值相同，需要放到同一个bucket中）；

3、如果碰撞了，以链表的方式链接到后面；

4、如果链表长度超过阀值( TREEIFY THRESHOLD==8)，就把链表转成红黑树，链表长度低于6，就把红黑树转回链表；

5、如果节点已经存在就替换旧值；

6、如果桶满了(容量16*加载因子0.75)，就需要 resize（扩容2倍后重排）。



- 以下是具体get过程(考虑特殊情况如果两个键的hashcode相同，你如何获取值对象？)



当我们调用get()方法，HashMap会使用键对象的hashcode找到bucket位置，找到bucket位置之后，会调用keys.equals()方法去找到链表中正确的节点，最终找到要找的值对象。

![img](https://pic1.zhimg.com/80/v2-7a1594967098357f3855365d8b222698_720w.jpg)



**3、有什么方法可以减少碰撞？**

- 扰动函数可以减少碰撞，原理是如果两个不相等的对象返回不同的hashcode的话，那么碰撞的几率就会小些，这就意味着存链表结构减小，这样取值的话就不会频繁调用equal方法，这样就能提高HashMap的性能。（扰动即Hash方法内部的算法实现，目的是让不同对象返回不同hashcode。）
- 使用不可变的、声明作final的对象，并且采用合适的equals()和hashCode()方法的话，将会减少碰撞的发生。不可变性使得能够缓存不同键的hashcode，这将提高整个获取对象的速度，使用String，Interger这样的wrapper类作为键是非常好的选择。为什么String, Interger这样的wrapper类适合作为键？因为String是final的，而且已经重写了equals()和hashCode()方法了。不可变性是必要的，因为为了要计算hashCode()，就要防止键值改变，如果键值在放入时和获取时返回不同的hashcode的话，那么就不能从HashMap中找到你想要的对象。

**4、HashMap中hash函数是怎么实现的?**

我们可以看到在hashmap中要找到某个元素，需要根据key的hash值来求得对应数组中的位置。如何计算这个位置就是hash算法。前面说过hashmap的数据结构是数组和链表的结合，所以我们当然希望这个hashmap里面的元素位置尽量的分布均匀些，尽量使得每个位置上的元素数量只有一个，那么当我们用hash算法求得这个位置的时候，马上就可以知道对应位置的元素就是我们要的，而不用再去遍历链表。 所以我们首先想到的就是把hashcode对数组长度取模运算，这样一来，元素的分布相对来说是比较均匀的。但是，“模”运算的消耗还是比较大的，能不能找一种更快速，消耗更小的方式，我们来看看JDK1.8的源码是怎么做的（被楼主修饰了一下）

```text
static final int hash(Object key) {
    if (key == null){
        return 0;
    }
     int h;
     h=key.hashCode()；返回散列值也就是hashcode
      // ^ ：按位异或
      // >>>:无符号右移，忽略符号位，空位都以0补齐
      //其中n是数组的长度，即Map的数组部分初始化长度
     return  (n-1)&(h ^ (h >>> 16));
}
```



![img](https://pic2.zhimg.com/80/v2-e4c228d45fe727d9df3b3d263be3a005_720w.jpg)

简单来说就是

1、高16bt不变，低16bit和高16bit做了一个异或(得到的HASHCODE转化为32位的二进制，前16位和后16位低16bit和高16bit做了一个异或)

2、(n·1)&hash=->得到下标



**5、拉链法导致的链表过深问题**

为什么不用二叉查找树代替，而选择红黑树？为什么不一直使用红黑树？

之所以选择红黑树是为了解决二叉查找树的缺陷，二叉查找树在特殊情况下会变成一条线性结构（这就跟原来使用链表结构一样了，造成很深的问题），遍历查找会非常慢。而红黑树在插入新数据后可能需要通过左旋，右旋、变色这些操作来保持平衡，引入红黑树就是为了查找数据快，解决链表查询深度的问题，我们知道红黑树属于平衡二叉树，但是为了保持“平衡”是需要付出代价的，但是该代价所损耗的资源要比遍历线性链表要少，所以当长度大于8的时候，会使用红黑树，如果链表长度很短的话，根本不需要引入红黑树，引入反而会慢。



**5、拉链法导致的链表过深问题**

为什么不用二叉查找树代替，而选择红黑树？为什么不一直使用红黑树？

之所以选择红黑树是为了解决二叉查找树的缺陷，二叉查找树在特殊情况下会变成一条线性结构（这就跟原来使用链表结构一样了，造成很深的问题），遍历查找会非常慢。而红黑树在插入新数据后可能需要通过左旋，右旋、变色这些操作来保持平衡，引入红黑树就是为了查找数据快，解决链表查询深度的问题，我们知道红黑树属于平衡二叉树，但是为了保持“平衡”是需要付出代价的，但是该代价所损耗的资源要比遍历线性链表要少，所以当长度大于8的时候，会使用红黑树，如果链表长度很短的话，根本不需要引入红黑树，引入反而会慢。



**6、说说你对红黑树的见解？**

![img](https://pic2.zhimg.com/80/v2-0786ff28e09d4b6a468163b5807e1315_720w.jpg)

1、每个节点非红即黑；

2、根节点总是黑色的；

3、如果节点是红色的，则它的子节点必须是黑色的（反之不一定）；

4、每个叶子节点都是黑色的空节点（NIL节点）；

5、从根节点到叶节点或空子节点的每条路径，必须包含相同数目的黑色节点（即相同的黑色高度）。

**7、解决hash 碰**



**8、HashMap的大小**

超过了负载因子(load factor)定义的容量，怎么办？

默认的负载因子大小为0.75，也就是说，当一个map填满了75%的bucket时候，和其它集合类(如ArrayList等)一样，将会创建原来HashMap大小的两倍的bucket数组，来重新调整map的大小，并将原来的对象放入新的bucket数组中。这个过程叫作rehashing，因为它调用hash方法找到新的bucket位置。这个值只可能在两个地方，一个是原下标的位置，另一种是在下标为<原下标+原容量>的位置。　　



**9、重新调整HashMap大小存在什么问题吗？**

- 当重新调整HashMap大小的时候，确实存在条件竞争，因为如果两个线程都发现HashMap需要重新调整大小了，它们会同时试着调整大小。在调整大小的过程中，存储在链表中的元素的次序会反过来，因为移动到新的bucket位置的时候，HashMap并不会将元素放在链表的尾部，而是放在头部，这是为了避免尾部遍历(tail traversing)。如果条件竞争发生了，那么就死循环了。(多线程的环境下不使用HashMap）
- 为什么多线程会导致死循环，它是怎么发生的？

2.在JDK1.8中，在并发执行put操作时会发生数据覆盖的情况。

其中第六行代码是判断是否出现hash碰撞，假设两个线程A、B都在进行put操作，并且hash函数计算出的插入下标是相同的，当线程A执行完第六行代码后由于时间片耗尽导致被挂起，而线程B得到时间片后在该下标处插入了元素，完成了正常的插入，然后线程A获得时间片，由于之前已经进行了hash碰撞的判断，所有此时不会再进行判断，而是直接进行插入，这就导致了线程B插入的数据被线程A覆盖了，从而线程不安全。

除此之前，还有就是代码的第38行处有个++size，我们这样想，还是线程A、B，这两个线程同时进行put操作时，假设当前HashMap的zise大小为10，当线程A执行到第38行代码时，从主内存中获得size的值为10后准备进行+1操作，但是由于时间片耗尽只好让出CPU，线程B快乐的拿到CPU还是从主内存中拿到size的值10进行+1操作，完成了put操作并将size=11写回主内存，然后线程A再次拿到CPU并继续执行(此时size的值仍为10)，当执行完put操作后，还是将size=11写回内存，此时，线程A、B都执行了一次put操作，但是size的值只增加了1，所有说还是由于数据覆盖又导致了线程不安全。

hashMap进行put操作时的流程是：先计算hashCode找到桶，然后遍历桶内的链表找到插入位置插入。即先查后写。但凡先查后写的，在并发环境下都会有并发查询问题

举个例子，比如有两个待写入的键值对{"a": 1, "b": 2}，刚好落到同一个桶里，假设这个桶是0号桶，这个桶里有一个元素c。

单线程环境下，我们先查a准备写入的位置，查到是0号桶的c后面的位置，之后写入a。然后查b准备写入的位置，查到是0号桶的a后面的位置，之后写入b。两个键值对都能正常被写入。

多线程环境下，a的查询可能和b是同步进行的。线程t1的查询里，a的写入位置是c后面，c.next=a；线程t2的查询里，b的写入位置也是c后面（因为此刻a尚未插入）, c.next=b。最终先插入c后面的会被后写的覆盖，只有后写的那个会被实际成功写入。

ConcurrentHashMap是怎么解决这样的并发问题的呢？其实很简单，就是在写入时对0号桶加锁，加锁期间别的线程需要等待锁释放后才能写入。比如线程t1写入a时加了锁，那么线程t2必须等待a写完才能写入b，这时b的写入位置是a后面，a.next=b，a和b都能正常被写入。（这是jdk1.7的做法，1.8改为CAS了）

是存在size()问题,++size不是原子操作







JDK1.8中的线程不安全
根据上面JDK1.7出现的问题，在JDK1.8中已经得到了很好的解决，如果你去阅读1.8的源码会发现找不到transfer函数，因为JDK1.8直接在resize函数中完成了数据迁移。另外说一句，JDK1.8在进行元素插入时使用的是尾插法。

为什么说JDK1.8会出现数据覆盖的情况喃，我们来看一下下面这段JDK1.8中的put操作代码：



HashMap的容量是有限的。当经过多次元素插入，使得HashMap达到一定饱和度时，Key映射位置发生冲突的几率会逐渐提高。这时候，HashMap需要扩展它的长度，也就是进行Resize。1.扩容：创建一个新的Entry空数组，长度是原数组的2倍。2.ReHash：遍历原Entry数组，把所有的Entry重新Hash到新数组。



1.JDK8以前是头插法，JDK8后是尾插法

## HashMap在1.8之前插入元素采用头插法的危害性