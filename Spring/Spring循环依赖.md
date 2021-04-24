Spring循环依赖怎么解决

https://blog.csdn.net/itmrchen/article/details/90201279

一级一级向下寻找，找出了前面提到的三级缓存，也就是三个Map集合类：

singletonObjects：第一级缓存，里面放置的是实例化好的单例对象；

earlySingletonObjects：第二级缓存，里面存放的是提前曝光的单例对象；

singletonFactories：第三级缓存，里面存放的是要被实例化的对象的对象工厂。

所以当一个Bean调用构造函数进行实例化后，即使属性还未填充，就可以通过三级缓存向外暴露依赖的引用值（所以循环依赖问题的解决也是基于Java的引用传递），这也说明了另外一点，基于构造函数的注入，如果有循环依赖，Spring是不能够解决的。还要说明一点，Spring默认的Bean Scope是单例的，而三级缓存中都包含singleton，可见是对于单例Bean之间的循环依赖的解决，Spring是通过三级缓存来实现的。源码是让我们知其然并且知其所以然的最好参考
————————————————
版权声明：本文为CSDN博主「itmrchen」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/itmrchen/article/details/90201279