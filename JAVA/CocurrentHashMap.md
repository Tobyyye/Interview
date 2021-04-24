讲一下 concurrentHashMap 原理。

头插法还是尾插法？

扩容怎么做？



https://zhuanlan.zhihu.com/p/138562977



面试官:看你简历里写了 HashMap，那你说说它存在什么缺点？

1. 线程不安全
2. 迭代时无法修改值
3. 那你有用过线程安全的 Map 吗？
   1. 有，回答在哪用过。
   2. 没有，不过我了解过。



ConcurrentHashMap 专门是给多个线程访问的。举个例子：

```
// 在线用户管理类
public class UserManager {
    private Map<String, User> userMap = new ConcurrentHashMap<>();
    
    // 当用户登入时调用
    public void onUserSignIn(String sessionId, User user) {
        this.userMap.put(sessionId, user);
    }
    
    // 当用户登出或超时时调用
    public void onUserSignOut(String sessionId) {
        this.userMap.remove(sessionId);
    }
    
    public getUser(String sessionId) {
        return this.userMap.get(sessionId);
    }
}
```

当有很多用户同时登入和登出时，`onUserSignIn()` 和 `onUserSignOut()` 就会有很多线程同时调用。





**面试官:**那你说说它们的实现。

1. Hashtable

Hashtable 本身比较低效，因为它的实现基本就是将 put、get、size 等各种方法加上 synchronized 锁。这就导致了所有并发操作都要竞争同一把锁，一个线程在进行同步操作时，其他线程只能等待，大大降低了并发操作的效率。





在 JDK1.7 中，ConcurrentHashMap 是由 Segment 数组结构和 HashEntry 数组结构组成。Segment 继承了 ReentrantLock，是一种可重入锁。HashEntry 则用于存储键值对数据。一个 ConcurrentHashMap 里包含一个 Segment 数组，一个 Segment 里包含一个 HashEntry 数组 ，每个 HashEntry 是一个链表结构的元素，因此 JDK1.7 的 ConcurrentHashMap 是一种数组+链表结构。当对 HashEntry 数组的数据进行修改时，必须首先获得与它对应的 Segment 锁，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全（分段锁）。

![img](https://pic1.zhimg.com/80/v2-75035a6388d0a57061f36fd6eac07a18_720w.jpg)

在 JDK1.8 中，ConcurrentHashMap 选择了与 HashMap 相同的**数组**+**链表**+**红黑树**结构，在锁的实现上，采用 CAS 操作和 synchronized 锁实现更加低粒度的锁，将锁的级别控制在了更细粒度的 table 元素级别，也就是说只需要锁住这个链表的首节点，并不会影响其他的 table 元素的读写，大大提高了并发度。

![img](https://pic3.zhimg.com/80/v2-a8248fdcfeae5f188d242acae89e1ff2_720w.jpg)

**面试官**：那为什么 JDK1.8 要使用 synchronized 锁而不是其他锁呢？

我认为有以下两个方面：

- Java 开发人员从未放弃过 synchronized 关键字，而且一直在优化，在 JDK1.8 中，synchronized 锁的性能得到了很大的提高，并且 synchronized 有多种锁状态，会从无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁一步步转换，因此并非是我们认为的重量级锁。
- 在粗粒度加锁中像 ReentrantLock 这种锁可以通过 Condition 来控制各个低粒度的边界，更加的灵活。而在低粒度中，Condition 的优势就没有了，此时 synchronized 不失为一种更好的选择。

**面试官**：你提到了 synchronized 的锁状态升级，能具体说下每种锁状态在什么情况下升级吗？

1. **无锁**

无锁没有对资源进行锁定，所有的线程都能访问并修改同一个资源，但同时只有一个线程能修改成功。

2.**偏向锁**

当一段同步代码一直被同一个线程所访问，无锁就会升级为偏向锁，以后该线程访问该同步代码时会自动获取锁，降低了获取锁的代价。

3.**轻量级锁**

当锁是偏向锁的时候，被另外的线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞，从而提高性能。

**4.重量级锁**

若当前只有一个等待线程，则该线程通过自旋进行等待。但是当自旋超过一定的次数，或者一个线程在持有锁，一个在自旋，又有第三个来访时，轻量级锁升级为重量级锁。

**面试官**：为什么 ConcurrentHashMap 的 key 和 value 不能为 null？

- 这是因为当通过 get(k) 获取对应的 value 时，如果获取到的是 null 时，无法判断，它是 put(k,v) 的时候 value 为 null，还是这个 key 从来没有添加。
- 假如线程 1 调用 map.contains(key) 返回 true，当再次调用 map.get(key) 时，map 可能已经不同了。因为可能线程 2 在线程 1 调用 map.contains(key) 时，删除了 key，这样就会导致线程 1 得到的结果不明确，产生多线程安全问题，因此，ConcurrentHashMap 的 key 和 value 不能为 null。
- 其实这是一种安全失败机制（fail-safe），这种机制会使你此次读到的数据不一定是最新的数据。

**面试官**：那你谈谈快速失败（fail-fast）和安全失败（fail-safe）的区别。

**快速失败(fail—fast)和安全失败(fail—safe)**都是 Java 集合中的一种机制。

如果采用快速失败机制，那么在使用迭代器对集合对象进行遍历的时候，如果 A 线程正在对集合进行遍历，此时 B 线程对集合进行增加、删除、修改，或者 A 线程在遍历过程中对集合进行增加、删除、修改，都会导致 A 线程抛出 ConcurrentModificationException 异常。

**面试官**：为什么在用迭代器遍历时，修改集合就会抛异常时？

- 原因是迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个 modCount 变量。集合在被遍历期间如果内容发生变化，就会改变 modCount 的值。

- 每当迭代器使用 hashNext()/next() 遍历下一个元素之前，都会检测 modCount 变量是否为 expectedModCount 值，是的话就返回遍历；否则抛出异常，终止遍历。

- java.util 包下的集合类都是快速失败的。

- **如果采用安全失败机制**，那么在遍历时不是直接在集合内容上访问，而是先复制原有集合内容，在拷贝的集合上进行遍历。

  由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，故不会抛 ConcurrentModificationException 异常。

  java.util.concurrent 包下的并发容器都是安全失败的。



**面试官：**ConcurrentHashMap 的 get 方法为什么不用加锁，会不会出现数据读写不一致情况呢？

- 不会出现读写不一致的情况。
- get 方法逻辑比较简单，只需要将 key 通过 hash 之后定位到具体的 Segment ，再通过一次 hash 定位到具体的元素上。
- 由于变量 value 是由 volatile 修饰的，根据 JMM 中的 happen before 规则保证了对于 volatile 修饰的变量始终是写操作先于读操作的，并且 volatile 的内存可见性保证修改完的数据可以马上更新到主存中，所以能保证在并发情况下，读出来的数据是最新的数据。

**面试官：**说下 ConcurrentHashMap 的 put 方法执行逻辑。

JDK1.7：

先尝试自旋获取锁，如果自旋重试的次数超过 64 次，则改为阻塞获取锁。获取到锁后：

1. 将当前 Segment 中的 table 通过 key 的 hashcode 定位到 HashEntry。
2. 遍历该 HashEntry，如果不为空则判断传入的 key 和当前遍历的 key 是否相等，相等则覆盖旧的 value。
3. 不为空则需要新建一个 HashEntry 并加入到 Segment 中，同时会先判断是否需要扩容。
4. 释放 Segment 的锁

JDK1.8：

先定位到 Node，拿到首节点 first，判断是否为：

1. 如果为 null ，通过 CAS 的方式把数据 put 进去。
2. 如果不为 null ，并且 first.hash = MOVED = -1 ，说明其他线程在扩容，参与一起扩容。
3. 如果不为 null ，并且 first.hash != -1 ，synchronized 锁住 first 节点，判断是链表还是红黑树，遍历插入。



JDK1.8：

由于没有 segment 的概念，所以只需要用一个 baseCount 变量来记录 ConcurrentHashMap 当前节点的个数。

- 先尝试通过CAS 更新 baseCount 计数。
- 如果多线程竞争激烈，某些线程 CAS 失败，那就 CAS 尝试将 cellsBusy 置 1，成功则可以把 baseCount 变化的次数暂存到一个数组 counterCells 里，后续数组 counterCells 的值会加到 baseCount 中。
- 如果 cellsBusy 置 1 失败又会反复进行 CAS baseCount 和 CAS counterCells 数组。

**面试官**：如何提高 ConcurrentHashMap 的插入效率？

主要从以下两个方面入手：

- 扩容操作。主要还是要通过配置合理的容量大小和负载因子，尽可能减少扩容事件的发生。
- 锁资源的争夺，在 put 方法中会使用 synchonized 对首节点进行加锁，而锁本身也是分等级的，因此我们的主要思路就是尽可能的避免锁升级。我们可以将数据通过 ConcurrentHashMap 的 spread 方法进行预处理，这样我们可以将存在哈希冲突的数据放在一个桶里面，每个桶都使用单线程进行 put 操作，这样的话可以保证锁仅停留在偏向锁这个级别，不会升级，从而提升效率。\





**面试官**：ConcurrentHashMap 是否存在线程不安全的情况？如果存在的话，在什么情况下会出现？如何解决？



我想起来了，存在这种情况，请看如下代码

```text
public class ConcurrentHashMapNotSafeDemo implements Runnable {
    private static ConcurrentHashMap<String, Integer> scores = new ConcurrentHashMap<>();
    public static void main(String[] args) throws InterruptedException {
        scores.put("John", 0);
        Thread t1 = new Thread(new ConcurrentHashMapNotSafeDemo());
        Thread t2 = new Thread(new ConcurrentHashMapNotSafeDemo());
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(scores);
    }
    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            Integer score = scores.get("John");
            Integer newScore = score + 1;
            scores.put("John", newScore);
        }
    }
}
```