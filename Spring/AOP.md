AOP的作用，什么是AOP

https://www.cnblogs.com/joy99/p/10941543.html 

# 一、AOP——另一种编程思想

## 1.1 什么是 AOP

AOP （Aspect Orient Programming）,直译过来就是 面向切面编程。AOP 是一种编程思想，是面向对象编程（OOP）的一种补充。面向对象编程将程序抽象成各个层次的对象，而面向切面编程是将程序抽象成各个切面。
 从《Spring实战（第4版）》图书中扒了一张图：

![img](https://img2018.cnblogs.com/blog/758949/201905/758949-20190529225344641-152289598.png)



## 1.2 为什么需要 AOP

想象下面的场景，开发中在多个模块间有某段重复的代码，我们通常是怎么处理的？显然，没有人会靠“复制粘贴”吧。在传统的面向过程编程中，我们也会将这段代码，抽象成一个方法，然后在需要的地方分别调用这个方法，这样当这段代码需要修改时，我们只需要改变这个方法就可以了。然而需求总是变化的，有一天，新增了一个需求，需要再多出做修改，我们需要再抽象出一个方法，然后再在需要的地方分别调用这个方法，又或者我们不需要这个方法了，我们还是得删除掉每一处调用该方法的地方。实际上涉及到多个地方具有相同的修改的问题我们都可以通过 AOP 来解决。

## 1.3 AOP 实现分类

AOP 要达到的效果是，保证开发者不修改源代码的前提下，去为系统中的业务组件添加某种通用功能。AOP 的本质是由 AOP 框架修改业务组件的多个方法的源代码，看到这其实应该明白了，AOP 其实就是前面一篇文章讲的代理模式的典型应用。
 按照 AOP 框架修改源代码的时机，可以将其分为两类：

- 静态 AOP 实现， AOP 框架在编译阶段对程序源代码进行修改，生成了静态的 AOP 代理类（生成的 *.class 文件已经被改掉了，需要使用特定的编译器），比如 AspectJ。
- 动态 AOP 实现， AOP 框架在运行阶段对动态生成代理对象（在内存中以 JDK 动态代理，或 CGlib 动态地生成 AOP 代理类），如 SpringAOP。





![img](https://img2018.cnblogs.com/blog/758949/201905/758949-20190529225400887-1590896890.png)

aop 的实现原理、动态代理过程

https://www.cnblogs.com/joy99/p/10865391.html



# 二、动态代理

动态代理的根据实现方式的不同可以分为 JDK 动态代理和 CGlib 动态代理。
 JDK 动态代理：利用反射机制生成一个实现代理接口的类，在调用具体方法前调用InvokeHandler来处理。
 CGlib 动态代理：利用ASM（开源的Java字节码编辑库，操作字节码）开源包，将代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。
 区别：JDK代理只能对实现接口的类生成代理；CGlib是针对类实现代理，对指定的类生成一个子类，并覆盖其中的方法，这种通过继承类的实现方式，不能代理final修饰的类。

