https://www.cnblogs.com/bigbigbigo/p/7596400.html

被private修饰的私有构造函数无法在其他类中调用，也就是该类无法在其他类中实例化。

这种情况常用的使用场景：1、单例模式；　　2、防止实例化。

　　一、单例模式

　　单例模式是一种常用的设计模式，思想是单例对象的类必须保证只有一个实例存在。

　　如何实现呢？一个简单的单例模式如下：

　　

```
public` `class` `Singleton {``  ``private` `static` `Singleton instance;` `  ``private` `Singleton() {``    ``System.out.println(``"Singleton的私有构造器"``);``  ``}` `  ``public` `static` `Singleton getInstance() {``    ``if` `(instance == ``null``)``      ``instance = ``new` `Singleton();``    ``return` `instance;``  ``}``}
```

　　单例模式类的特点：

　　1. 一个private static的自身类型的属性，保证实例的唯一性；

　　2. 私有构造器，防止随意实例化；

　　3.一个public static的getInstance()得到唯一实例的方法；

　　当需要一个类实例时，用一下语句：

　　

```
Singleton single=Singleton.getInstance();
```

　　

　　二、防止实例化

　　某种情况下，我们只需要把某个类（工具类）当成“函数”使用，即只需要用到里面的static方法完成某些功能。

　　这种情况下不需要获得实例，所以getInstance()方法可有可无。

 

　　三、利用反射机制可以打破私有构造器的限制

　　利用反射机制，修改私有构造器的访问权限，也可以获得实例。