## 总结

对于线上出现的OOM，如何分析和解决可以大致分为三个步骤：

1. 充分挖掘特征。在挖掘特征时，需要多方面考虑，此过程更多的是猜测怀疑，所以可能的方面都要考虑到，包括但不限于代码改动、机器特征、时间特征等，必要时还需要做一定的统计分析。

2. 根据掌握的特征寻找稳定的复现的途径。一般需要做内存压力测试，这样比较容易达到OOM的临界值，只是简单的一些正常操作难以触发OOM。

3. 获取可分析的数据（内存dump文件）。利用MAT分析dump文件，MAT可以方便的按照大小排序实例，可以查看某些实例到GC ROOT的路径。

   

应用程序出现OOM异常，你是否仍然通过看日志的方式去排查问题（该方式定位解决问题是大概率的巧合而已）？正确的排查方案是进行dump文件分析，你知道为什么吗？

目录

- [OOM异常--intsmaze](https://www.cnblogs.com/intsmaze/p/9550256.html#oom异常--intsmaze)
- [正确姿势dump文件分析--intsmaze](https://www.cnblogs.com/intsmaze/p/9550256.html#正确姿势dump文件分析--intsmaze)
- [正确的姿势--intsmaze](https://www.cnblogs.com/intsmaze/p/9550256.html#正确的姿势--intsmaze)
- dump丢失打印--intsmaze
  - [查看/var/log/messages文件](https://www.cnblogs.com/intsmaze/p/9550256.html#查看varlogmessages文件)
- [哪些内存溢出会产生dump文件--intsmaze](https://www.cnblogs.com/intsmaze/p/9550256.html#哪些内存溢出会产生dump文件--intsmaze)



原文链接：https://blog.csdn.net/u012556994/article/details/81264962

四、如何解决这些OOM异常？

1、要解决OOM异常或heap space的异常，一般的手段是首先通过内存映像分析工具（如Eclipse Memory Analyzer）对dump 出来的堆转储快照进行分析，重点是确认内存中的对象是否是必要的，也就是要先分清楚到底是出现了内存泄漏（Memory Leak）还是内存溢出（Memory Overflow）。

2、如果是内存泄漏，可进一步通过工具查看泄漏对象到GC Roots 的引用链。于是就能找到泄漏对象是通过怎样的路径与GC Roots 相关联并导致垃圾收集器无法自动回收它们的。掌握了泄漏对象的类型信息，以及GC Roots 引用链的信息，就可以比较准确地定位出泄漏代码的位置。

3、如果不存在内存泄漏，换句话说就是内存中的对象确实都还必须存活着，那就应当检查虚拟机的堆参数（-Xmx 与-Xms），与机器物理内存对比看是否还可以调大，从代码上检查是否存在某些对象生命周期过长、持有状态时间过长的情况，尝试减少程序运行期的内存消耗







https://blog.csdn.net/fenglibing/article/details/82692169

# 线上故障排查(2) - Java应用故障之堆溢出OOM问题及排查方案



以下是用于测试OOM的测试代码：

public class HeapMemUseTest {
    public static void main(String[] args) {
        StringBuilder sb = new StringBuilder();
        while(true) {
            sb.append(System.currentTimeMillis());
        }
    }
}
这段代码非常简单，其目的就是为了模拟OOM，将其编译后，通过以下命令运行：

java -Xmx10m -Xms10m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./oom.out HeapMemUseTest



其中的参数代表的意义为：

-Xmx和-Xms分别是用于指定该Java进程初使化的最小堆内存以及可以使用的最大堆内存的，这里设置为10M

-XX:+HeapDumpOnOutOfMemoryError和-XX:HeapDumpPath参数分别用于指定发生OOM是否要导出堆以及导出堆的文件路径

该命令一执行，立即就会发生OOM，并打印如下的日志：

fenglibin@fenglibin-HP:~/eclipse_neon_workspace/Test/bin$ java -Xmx10m -Xms10m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./oom.out HeapMemUseTest
java.lang.OutOfMemoryError: Java heap space
Dumping heap to ./oom.out ...
Heap dump file created [5513523 bytes in 0.027 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3332)
	at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:124)
	at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:700)
	at java.lang.StringBuilder.append(StringBuilder.java:214)
	at HeapMemUseTest.main(HeapMemUseTest.java:13)
查看当前路径，oom.out文件已经生成了，该文件就是应用在发生OOM异常时自动导出的堆文件。那我们此时需要对该文件进行分析，因为其中记录了是什么对象导出了应用程OOM的发生。

分析OOM的工具推荐使用MAT，下载地址为https://projects.eclipse.org/projects/tools.mat，







https://blog.csdn.net/lkforce/article/details/60878295?utm_source=blogkpcl7

**使用jvisualvm来分析dump文件：**

jvisualvm是JDK自带的Java性能分析工具，在JDK的bin目录下，文件名就叫jvisualvm.exe。
jvisualvm可以监控本地、远程的java进程，实时查看进程的cpu、堆、线程等参数，对java进程生成dump文件，并对dump文件进行分析。
像我这种从服务器上dump下来文件也可以直接扔给jvisualvm来分析。

使用方式：直接双击打开jvisualvm.exe，点击文件->装入，在文件类型那一栏选择堆，选择要分析的dump文件，打开



# jvisualvm监控程序，使用IDEA连接相关配置

https://blog.csdn.net/sinat_34842630/article/details/103700368



Program arguments配置如下：

-Djava.rmi.server.hostname=127.0.0.1
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=1099
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false

**为了验证一下，我自己在本地模拟了一下堆内存溢出的情形，并用jvisualvm监控**

我本地设置的最大堆内存是800M左右，代码里面是往一个List里面疯狂加数据，直到撑爆堆内存，得到的监控内容是这样的：



![img](https://img-blog.csdn.net/20170308185634788?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGtmb3JjZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



设置10M我们可以发现list添加132各个元素时会发生OOM，这个时候我们向list添加131个元素，然后执行map添加，发现map添加一个元素就报错。此时的oom异常日志定位的是map添加元素导致的。
但是真实情况不是的，因为看代码也会发现map只添加了2个元素，怎么会是他造成的。map的添加只是刚好此时jvm内存达到容量上限了。
所以要找到根本问题，是需要通过dump文件分析OOM时，各个对象的容量状态。
而且实际情况中，map.put()的代码并不会向上面示例一样和list.add()代码放在一块，而是位于不同的包下，不同的业务流程中。这个时候看log日志去定位基本不可能了。
但是为什么大家出行OOM异常还是通过看log日志而且定位的位置是正确的。只是因为向list.add这种循环中，一直在执行，基本大概率是他触发的。



所以为了防患于未然，程序猿在开发的时候，一定要配置jvm启动参数HeapDumpOnOutOfMemoryError。



参数-XX：+HeapDumpOnOutOfMemoryError可以让虚拟机在出现内存溢出异常时Dump出当前的内存堆转储快照以便事后进行分析
![img](https://images2018.cnblogs.com/blog/758427/201808/758427-20180828185246502-1691551912.png)





#### 查看/var/log/messages文件

messages 日志是核心系统日志文件。它包含了系统启动时的引导消息，以及系统运行时的其他状态消息。在messages里会出现以下信息

```
out of memory:kill process 8398(java) score 699 or sacrifice child
killed process 8398,UID 505,(java) total-vm:2572232kB,anno-rss:1431292kB,file-rss:908kB
```

oom killer是linux系统的一个保护进程，当linux系统所剩的内存空间不足以满足系统正常运行时，会触发。oomkiller执行时，会找出系统所有线程的score值最高的那个pid，然后干掉。
这里我们可以看到，JAVA进程的确是被LINUX的oom killer干掉了。

我们的应用程序和日志都只能记录JVM内发生的内存溢出。如果JVM设置的堆大小超出了操作系统允许的内存大小，那么操作系统会直接杀死进程，这种情况JVM就无法记录本次操作。
Linux对于每个进程有一个OOM评分，这个评分在/proc/pid/oom_score文件中。例如/proc/8398/oom_score，如果不希望杀死这个进程，就将oom_adj内容改为-17。
更多关于linux的oom killer机制请自行百度检索。





# Java内存泄漏的排查总结

https://blog.csdn.net/fishinhouse/article/details/80781673





# 如何排查Java内存泄漏？

https://zhuanlan.zhihu.com/p/80255370



## **使用Java VisualVM远程分析堆**

https://www.cnblogs.com/zuxiaoyuan/p/10078588.html



![img](https://img2018.cnblogs.com/blog/953701/201812/953701-20181206190822478-1195680991.png)

