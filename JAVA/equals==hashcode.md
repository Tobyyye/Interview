**hash code、equals和“==”三者的关系**



1.如果是基本变量，没有hashcode和equals方法，基本变量的比较方式就只有==,；

2.如果是变量，由于在java中所有变量定义都是一个指向实际存储的一个句柄（你可以理解为c++中的指针），在这里==是比较句柄的地址（你可以理解为指针的存储地址），而不是句柄指向的实际内存中的内容，如果要比较实际内存中的内容，那就要用equals方法，但是！！！

如果是你自己定义的一个类，比较自定义类用equals和==是一样的，都是比较句柄地址，因为自定义的类是继承于object，而object中的equals就是用==来实现的，你可以看源码。

那为什么我们用的String等等类型equals是比较实际内容呢，是因为String等常用类已经重写了object中的equals方法，让equals来比较实际内容，你也可以看源码。

\3. hashcode
在一般的应用中你不需要了解hashcode的用法，但当你用到hashmap，hashset等集合类时要注意下hashcode。

你想通过一个object的key来拿hashmap的value，hashmap的工作方法是，通过你传入的object的hashcode在内存中找地址，当找到这个地址后再通过equals方法来比较这个地址中的内容是否和你原来放进去的一样，一样就取出value。

所以这里要匹配2部分，hashcode和equals
但假如说你new一个object作为key去拿value是永远得不到结果的，因为每次new一个object，这个object的hashcode是永远不同的，所以我们要重写hashcode，你可以令你的hashcode是object中的一个恒量，这样永远可以通过你的object的hashcode来找到key的地址，然后你要重写你的equals方法，使内存中的内容也相等。。。



**先，**从语法角度，也就是从强制性的角度来说，hashCode和equals是两个独立的，互不隶属，互不依赖的方法，equals成立与hashCode相等这两个命题之间，谁也不是谁的充分条件或者必要条件。 

但是，从为了让我们的程序正常运行的角度，我们应当向Effective Java中所言 

重载equals的时候，一定要（正确）重载hashCode 

使得equals成立的时候，hashCode相等，也就是a.equals(b)->a.hashCode() == b.hashCode()，或者说此时，equals是hashCode相等的充分条件，hashCode相等是equals的必要条件（从数学课上我们知道它的逆否命题：hashCode不相等也不会equals），但是它的逆命题，hashCode相等一定equals以及否命题不equals时hashCode不等都不成立。 

所以，如果面试的时候，最好把hashCode与equals之间没有强制关系，以及根据（没有语法约束力的）规范的角度，应当做到...这两层意思都说出来:P



对于equals，一般我们认为两个对象同类型并且所有属性相等的时候才是相等的，在类中必须改写equals，因为Object类中的equals只是判断两个引用变量是否引用同一对象，如果不是引用同一对象，即使两个对象的内容完全相同，也会返回false。当然，在类中改写这个equals时，你也可以只对部分属性进行比较，只要这些属性相同就认为对象是相等的。 

对于hashCode，只要是用在和哈希运算有关的地方，前面提到了，和equals一样，在你的类中也应该改写。当然如果两个对象是完全相同的，那么他们的hashCode当然也是一样的，但是象前面所述，规则可以由你自己来定义，因此两者之间并没有什么必然的联系。 

当然，大多数情况下我们还是根据所有的属性来计算hashCode和进行相等性比较。



**面试题：hashCode知道是干什么的吗？如果要你重写，需要注意哪些点？(腾讯面试题)
面试题：问我使用hashmap时重写哪两个方法，为什么要重写(百度面试题)**

一、hashCode简介
public int hashCode()：hashCode是根类Obeject中的方法。默认情况下，Object中的hashCode() 返回对象的32位jvm内存地址。也就是说如果对象不重写该方法，则返回相应对象的32为JVM内存地址。
二、hashCode注意点
关于hashCode方法，一致的约定是：
1、重写了euqls方法的对象必须同时重写hashCode()方法。
2、如果两个对象equals相等，那么这两个对象的HashCode一定也相同
3、如果两个对象的HashCode相同，不代表两个对象就相同，只能说明这两个对象在散列存储结构中，存放于同一个位置
三、hashCode作用
从Object角度看，JVM每new一个Object，它都会将这个Object丢到一个Hash表中去，这样的话，下次做Object的比较或者取这个对象的时候（读取过程），它会根据对象的HashCode再从Hash表中取这个对象。这样做的目的是提高取对象的效率。若HashCode相同再去调用equal。
HashCode是用于查找使用的，而equals是用于比较两个对象的是否相等的。
四、为什么重写
实际开发的过程中在hashmap或者hashset里如果不重写的hashcode和equals方法的话会导致我们存对象的时候，把对象存进去了，取的时候却取不到想要的对象。
重写了hashcode和equals方法可以迅速的在hashmap中找到键的位置；
1、重写hashcode是为了保证相同的对象会有相同的hashcode；
2、重写equals是为了保证在发生冲突的情况下取得到Entry对象（也可以理解是key或是元素）；
————————————————



