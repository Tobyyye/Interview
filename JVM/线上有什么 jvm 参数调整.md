**https://blog.csdn.net/a724888/article/details/78367780**    





# JVM线上常用参数、常用工具以及异常排查



下面只列举其中的几个常用和容易掌握的配置选项

| 配置参数                        | 功能                                                         |
| :------------------------------ | :----------------------------------------------------------- |
| -Xms                            | 初始堆大小。如：-Xms256m                                     |
| -Xmx                            | 最大堆大小。如：-Xmx512m                                     |
| -Xmn                            | 新生代大小。通常为 Xmx 的 1/3 或 1/4。新生代 = Eden + 2 个 Survivor 空间。实际可用空间为 = Eden + 1 个 Survivor，即 90% |
| -Xss                            | JDK1.5+ 每个线程堆栈大小为 1M，一般来说如果栈不是很深的话， 1M 是绝对够用了的。 |
| -XX:NewRatio                    | 新生代与老年代的比例，如 –XX:NewRatio=2，则新生代占整个堆空间的1/3，老年代占2/3 |
| -XX:SurvivorRatio               | 新生代中 Eden 与 Survivor 的比值。默认值为 8。即 Eden 占新生代空间的 8/10，另外两个 Survivor 各占 1/10 |
| -XX:PermSize                    | 永久代(方法区)的初始大小                                     |
| -XX:MaxPermSize                 | 永久代(方法区)的最大值                                       |
| -XX:+PrintGCDetails             | 打印 GC 信息                                                 |
| -XX:+HeapDumpOnOutOfMemoryError | 让虚拟机在发生内存溢出时 Dump 出当前的内存堆转储快照，以便分析用 |



Xms：初始堆大小

Xmx：最大堆大小

Xss：Java 每个线程的Stack大小

XX：NewSize=n：设置年轻代大小

XX：NewRatio=n：设置年轻代和年老代的比值。如：为 3，表示年轻代与年老代比值为 1:3，年轻代占整个年轻代年老代和的 1/4。

XX：SurvivorRatio=n：年轻代中 Eden 区与两个 Survivor 区的比值。注意 Survivor区有两个。如：3，表示 Eden：Survivor=3：2，一个 Survivor 区占整个年轻代的 1/5。

XX：MaxPermSize=n：设置持久代大小。

## 收集器设置



XX：+UseSerialGC：设置串行收集器

XX：+UseParallelGC:：设置并行收集器

XX：+UseParalledlOldGC：设置并行年老代收集器

XX：+UseConcMarkSweepGC：设置并发收集器

XX：+UseG1GC：G1收集器，Java9默认开启，无需设置



https://www.cnblogs.com/jiataoq/p/9512498.html

## JVM 提供的常用工具



1、jps：用来显示本地的 Java 进程，可以查看本地运行着几个 Java 程序，并显示他们的进程号。

命令格式：jps



2、jinfo：运行环境参数：Java System 属性和 JVM 命令行参数，Java class path 等信息。

命令格式：jinfo 进程 pid



3、jstat：监视虚拟机各种运行状态信息的命令行工具。

命令格式：jstat -gc 123 250 20



4、jstack：可以观察到 JVM 中当前所有线程的运行情况和线程当前状态。

命令格式：jstack 进程 pid



5、jmap：观察运行中的 JVM 物理内存的占用情况（如：产生哪些对象，及其数量）。

命令格式：jmap [option] pid

-dump:[live,]format=b,file= 使用hprof二进制形式,输出jvm的heap内容到文件, live子选项是可选的，假如指定live选项,那么只输出活的对象到文件.

-finalizerinfo 打印正等候回收的对象的信息.

-heap 打印heap的概要信息，GC使用的算法，heap的配置及wise heap的使用情况.

-histo[:live] 打印每个class的实例数目,内存占用,类全名信息. VM的内部类名字开头会加上前缀”*”. 如果live子参数加上后,只统计 活的对象数量.

-permstat 打印classload和jvm heap长久层的信息. 包含每个classloader的名字,活泼性,地址,父classloader和加载的class数量. 另外,内部String的数量和占用内存数也会打印出来.

使用示例：jmap -heap 27900 # 展示pid的整体堆信息如下：



![img](https://pic4.zhimg.com/80/v2-455a6cbd760e401ee365f69d24c7ee5b_720w.jpg)



异常排查



CPU 资源占用过高

1、top 查看当前 CPU 情况，找到占用 CPU 过高的进程 PID=123。

2、top -H -p123 找出两个 CPU 占用较高的线程，记录下来 PID=2345, 3456 转换为十六进制。

3、jstack -l 123 > temp.txt 打印出当前进程的线程栈。

4、查找到对应于第二步的两个线程运行栈，分析代码。



OOM 异常排查

1、使用 top 指令查询服务器系统状态。

2、ps -aux|grep java 找出当前 Java 进程的 PID。

3、jstat -gcutil pid interval 查看当前 GC 的状态。

4、jmap -histo:live pid 可用统计存活对象的分布情况，从高到低查看占据内存最多的对象。

5、jmap -dump:format=b,file= 文件名 [pid] 利用 Jmap dump。

6、使用性能分析工具对上一步 dump 出来的文件进行分析，工具有 MAT 等。