https://blog.csdn.net/javazejian/article/details/54561302

详细的解析

Spring 依赖注入

所谓的依赖注入，其实是当一个bean实例引用到了另外一个bean实例时spring容器帮助我们创建依赖bean实例并注入（传递）到另一个bean中，如上述案例中的AccountService依赖于AccountDao，Spring容器会在创建AccountService的实现类和AccountDao的实现类后，把AccountDao的实现类注入AccountService实例中



## 自动装配与注解注入

### 基于xml的自动装配





#### 基于@Autowired注解的自动装配

通过上述的分析，我们已对自动装配有所熟悉，但是在bean实例过多的情景下，手动设置自动注入属性还是不太完美，好在Spring 2.5 中引入了 @Autowired 注释，它可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作。 通过 @Autowired的使用标注到成员变量时不需要有set方法，请注意@Autowired 默认按类型匹配的，先看示例用注解演示前面注入userDao实例的三种方式。当然使用注解前必须先注册注解驱动，这样注解才能被正确识别







# IOC 与依赖注入的区别

IOC:控制反转:将对象的创建权,由Spring管理. 
 DI(依赖注入):在Spring创建对象的过程中,把对象依赖的属性注入到类中。 
 它们两就这样，其他什么毛线都没有了。



