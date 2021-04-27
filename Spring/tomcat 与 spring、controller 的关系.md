# [Servlet／Tomcat/ Spring 之间的关系](https://www.cnblogs.com/shawshawwan/p/9002126.html)



**0.基础知识**

在idea中打开servlet的源码：

![img](https://images2018.cnblogs.com/blog/1251492/201805/1251492-20180507131022182-2140190383.png)



可以看见servlet就是一个接口；接口就是规定了一些规范，使得一些具有某些共性的类都能实现这个接口，从而都遵循某些规范。

有的人往往以为就是servlet直接处理客户端的http请求，其实并不是这样，servlet并不会去监听8080端口；直接与客户端打交道是“容器”，比如常用的tomcat。

客户端的请求直接打到tomcat，它监听端口，请求过来后，根据url等信息，确定要将请求交给哪个servlet去处理，然后调用那个servlet的service方法，service方法返回一个response对象，tomcat再把这个response返回给客户端。

![img](https://images2018.cnblogs.com/blog/1251492/201805/1251492-20180507132038095-966635725.jpg)



**1. Servlet的生命周期**

 

从创建到毁灭：

 

1. 调用 `init()` 方法初始化
2. 调用 `service()` 方法来处理客户端的请求
3. 调用 `destroy()` 方法释放资源，标记自身为可回收
4. 被垃圾回收器回收

 

由上面可以看见，servlet的init方法和destroy方法，一般容器调用这两个方法之间的过程，就叫做servlet的生命周期。

调用的整个过程就如上图所示。

当请求来容器第一次调用某个servlet时，需要先初始化init()，

但当某个请求再次打到给servlet时，容器会起多个线程同时访问一个servlet的service（）方法。

![img](https://images2018.cnblogs.com/blog/1251492/201805/1251492-20180507141819258-2036948492.png)

**面试问题：Servlet如何同时处理多个请求访问？** 

**单实例多线程:** 主要是请求来时，会由线程调度者从线程池李取出来一个线程，来作为响应线程。这个线程可能是已经实例化的，也可能是新创建的。


Servlet容器默认是采用**单实例多线程**的方式处理多个请求的： 
1.当web服务器启动的时候（或客户端发送请求到服务器时），Servlet就被加载并实例化(只存在一个Servlet实例)； 
2.容器初始化化Servlet主要就是读取配置文件（例如tomcat,可以通过servlet.xml的设置线程池中线程数目，初始化线程池通过web.xml,初始化每个参数值等等。 
3.当请求到达时，Servlet容器通过调度线程(Dispatchaer Thread) 调度它管理下线程池中等待执行的线程（Worker Thread）给请求者； 
4.线程执行Servlet的service方法； 
5.请求结束，放回线程池，等待被调用； 
（注意：避免使用实例变量（成员变量），因为如果存在成员变量，可能发生多线程同时访问该资源时，都来操作它，照成数据的不一致，因此产生线程安全问题）

 

从上面可以看出： 
第一：Servlet单实例，减少了产生servlet的开销； 
第二：通过线程池来响应多个请求，提高了请求的响应时间； 
第三：Servlet容器并不关心到达的Servlet请求访问的是否是同一个Servlet还是另一个Servlet，直接分配给它一个新的线程；如果是同一个Servlet的多个请求，那么Servlet的service方法将在多线程中并发的执行； 
第四：每一个请求由ServletRequest对象来接受请求，由ServletResponse对象来响应该请求；





![img](https://t11.baidu.com/it/u=1555143694,4189494472&fm=173&app=49&f=JPEG?w=640&h=377&s=8066DA1410BEC5CC1436C44E0300F0FB)





tomcat启动。spring的监听器监听ServletContext。初始化ServletContext时,马上初始化spring（父），并将spring存入ServletContext中(和儿子的约定的地方)。第一次请求DispatcherServlet时，初始化 spring mvc（子）,并且会第一时间去ServletContext中找spring（父）。spirng mvc 需要什么资源(dao,service等) spring 只要有的都会给他，反过却不是，spring 并不会到spring mvc中去要资源(controller等)。

**2. Spring** 

 

 任何Spring Web的entry point，都是servlet。

 

大名顶顶的spring框架已经风靡多时，一个事物的出现和流行都是会有原因的，那么为什么spring 框架会出现呢？原因就是为了**简化java开发**。

spring的核心就是通过**依赖注入、面向切面编程aop、和模版技术**，解耦业务与系统服务，消除重复代码。借助aop，可以将遍布应用的关注点（如事物和安全）从它们的应用对象中解耦出来。



![img](https://images2018.cnblogs.com/blog/1251492/201807/1251492-20180705093420518-608899068.png)

