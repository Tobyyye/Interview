# 获取并分析dump文件



本地获取dump文件：

1、JDK8之前

-XX:PermSize=3m  -XX:MaxPermSize=3m   -XX:+HeapDumpOnOutOfMemoryError

2、JDK8以及之后（没有永久代了，改成元空间Metaspace了）

-XX:MetaspaceSize=3M -XX:MaxMetaspaceSize=3M   -XX:+HeapDumpOnOutOfMemoryError



**线上如何获取dump文件：**

使用jmap -dump:live,format=b,file=xxx [pid]，例如：

jmap -dump:live,format=b,file=heap201712.hropf  72947





分析

**使用Jprofile分析dump文件**

需要先将文件后缀修改为hprof，用Jprofile可以一目了然的看出是大量对象

**注意：直接内存溢出、栈内存溢出没有dump文件生成**



## Java 性能诊断工具简介

在 Java 的世界里，有许多诊断工具可供选择，既包括像 jmap、jstat 这样的简单命令行工具，又包括 JVisualvm、JProfiler 等图形化综合诊断工具，同时还有 SkyWalking、ARMS 这样的针对分布式应用的性能监控系统。下面分别对其进行介绍。

## 简单命令行工具

JDK 内置了许多命令行工具，它们可用来获取目标 JVM 不同方面、不同层次的信息。

- jinfo - 用于实时查看和调整目标 JVM 的各项参数。
- jstack - 用于获取目标 Java 进程内的线程堆栈信息，可用来检测死锁、定位死循环等。
- jmap - 用于获取目标 Java 进程的内存相关信息，包括 Java 堆各区域的使用情况、堆中对象的统计信息、类加载信息等。
- jstat - 一款轻量级多功能监控工具，可用于获取目标 Java 进程的类加载、JIT 编译、垃圾收集、内存使用等信息。
- jcmd - 相比 jstat 功能更为全面的工具，可用于获取目标 Java 进程的性能统计、JFR、内存使用、垃圾收集、线程堆栈、JVM 运行时间等信息。

## 图形化综合诊断工具

使用上述命令行工具或组合能帮您获取目标 Java 应用性能相关的基础信息，但它们存在下列局限：

1. 无法获取方法级别的分析数据，如方法间的调用关系、各方法的调用次数和调用时间等（这对定位应用性能瓶颈至关重要）。
2. 要求用户登录到目标 Java 应用所在的宿主机上，使用起来不是很方便。
3. 分析数据通过终端输出，结果展示不够直观。

下面介绍几款图形化的综合性能诊断工具。

**JVisualvm**

[JVisualvm](https://link.zhihu.com/?target=https%3A//docs.oracle.com/javase/8/docs/technotes/tools/unix/jvisualvm.html) 是 JDK 内置的可视化性能诊断工具，它通过 JMX、jstatd、Attach API 等方式获取目标 JVM 的分析数据，包括 CPU 使用率、内存使用量、线程堆栈信息等。此外，它还能直观地展示 Java 堆中各对象的数量和大小、各 Java 方法的调用次数和执行时间等。

**JProfiler**

[JProfiler](https://link.zhihu.com/?target=https%3A//www.ej-technologies.com/products/jprofiler/overview.html) 是由 ej-technologies 公司开发的一款 Java 应用性能诊断工具。它聚焦于四个重要主题上。

1. 方法调用 - 对方法调用的分析可以帮助您了解应用程序正在做什么，并找到提高其性能的方法。
2. 内存分配 - 通过分析堆上对象、引用链和垃圾收集能帮您修复内存泄漏问题，优化内存使用。
3. 线程和锁 - JProfiler 提供多种针对线程和锁的分析视图助您发现多线程问题。
4. 高级子系统 - 许多性能问题都发生在更高的语义级别上。例如，对于JDBC调用，您可能希望找出执行最慢的 SQL 语句。JProfiler 支持对这些子系统进行集成分析。