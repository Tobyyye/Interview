![在这里插入图片描述](https://img-blog.csdnimg.cn/20200527214023981.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NsZWFyTEI=,size_16,color_FFFFFF,t_70)

### 1.什么是Spring

Spring是一个轻量级Java开发框架，最早由Rod Johnson创建，目的是为了解决企业级应用程序开发的业务逻辑层和其他各层的耦合问题，Spring的两个核心特性就是依赖注入（DI）和面向切面编程（AOP）

为了降低Java开发的复杂性，Spring采用了以下4中关键策略：

    基于POJO的轻量级和最小入侵编程（最小入侵编程简单来说就是，Spring Framework没有强迫开发者实现框架自带的接口）
    通过依赖注入和面向接口实现松耦合
    基于切面和惯例进行声明式编程
    通过切面和模板减少样板式代码
2.Spring框架的设计目标，设计理念和核心是什么

    Spring设计目标：Spring为开发者提供一个一站式轻量级（区分轻量级和重量级，主要看它使用了多少服务，而不是看框架使用了多少内存空间）应用开发平台
    Spring设计理念：Spring通过IoC容器实现对象耦合关系的管理，并实现依赖反转（IoC），将对象之间的依赖关系交给IoC容器，实现解耦
    Spring框架的核心：IoC容器和AOP模块，通过IoC容器管理POJO对象以及它们之间的耦合关系；通过AOP以动态非入侵的方式增强服务
    spring core：提供了框架的基本组成部分，包括控制反转（IOC）和依赖注入（DI）功能
    spring beans：提供了BeanFactory，是工厂模式的一个经典实现，Spring将管理归降称为Bean
    spring context：提供了一种框架式的对象访问方法，提供框架的上下文环境
    spring aop：提供了面向切面的编程实现，可以自定义拦截器、切点等
    spring Web：提供了针对Web开发的集成特性
    spring test：主要为测试提供支持



4.Spring框架中都用到了哪些设计模式

    工厂模式：BeanFactory就是简单的工厂模式的体现，用来创建对象的实例
    单例模式：Bean默认为单例模式
    代理模式：Spring的AOP功能用到了JDK的动态代理
    观察者模式：定义对象键一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知被制动更新
