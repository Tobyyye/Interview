### Class对象

https://blog.csdn.net/javazejian/article/details/70768369

理解class对象 的作用 反射的概念



深入理解Class对象
RRTI的概念以及Class对象作用

认识Class对象之前，先来了解一个概念，RTTI（Run-Time Type Identification）运行时类型识别，对于这个词一直是 C++ 中的概念，至于Java中出现RRTI的说法则是源于《Thinking in Java》一书，其作用是在运行时识别一个对象的类型和类的信息，这里分两种：传统的”RRTI”,它假定我们在编译期已知道了所有类型(在没有反射机制创建和使用类对象时，一般都是编译期已确定其类型，如new对象时该类必须已定义好)，另外一种是反射机制，它允许我们在运行时发现和使用类型的信息。在Java中用来表示运行时类型信息的对应类就是Class类，Class类也是一个实实在在的类，存在于JDK的java.lang包中，其部分源码如下：

![img](https://img-blog.csdn.net/20170430093618437?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF2YXplamlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)









# 理解反射技术