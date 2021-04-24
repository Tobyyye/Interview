简单介绍下 ArrayList 怎么实现，加操作、取值操作，什么时候扩容？ 



**ArrayList初始化时Object数组的长度是多少呢？**

**无参构造函数初始化的ArrayList执行一次add方法后数组长度又是多少呢？**

**ArrayList是如何扩容的？**

**在 foreach 循环里****可否****进行元素的 remove/add 操作****？为什么？**

**ArrayList源码解读**



![img](https://upload-images.jianshu.io/upload_images/21615605-70c5e0ced6be2e5d.png?imageMogr2/auto-orient/strip|imageView2/2/w/939/format/webp)



根据属性和主要方法的源码我们再来回答上面的问题。

首先是初始化数据的长度是多少，ArrayList一共有三个构造函数，**无**参构造函数会把上图中的第三个变量赋给第四个**，也就是说无参构造函数初始化的数组长度为0**，带初始化长度的的则会初始化指定长度的数组，如果参数为0,则会把第三个变量赋给第四个，还有一个构造函数可以传递一个Collection对象，那么对应数组的长度就是集合的长度，同样如果集合对象长度为0的话也会把第三个变量赋给第四个。

**add（Object）源码流程分析**

接着来分析add方法，add有两个方法一个是指定插入的位置，一个是不指定也就是添加到末尾，先来看不指定索引的方法，主要流程如下：

add方法的第一步是拿到size+1计算出最小容量minCapacity去验证是否需要扩容；

**扩容的第一步是验证属性elementData是否等于默认的初始化数组**（也就是是否等于DEFAULTCAPACITY_EMPTY_ELEMENTDATA），是则表明集合刚被初始化，那么minCapacity就去minCapacity与10的最大值；

最后在验证minCapacity是否大于elementData的length，大于才扩容；

扩容时首先根据老长度oldCapacity = elementData.length来计算新的数组长度newCapacity=oldCapacity + (oldCapacity >> 1);**可以看到每次扩容都是在原有的长度上加二分之一的长度；**

最后再取传递进来的minCapacity与newCapacity的最大值作为真正的下一个elementData的长度。

最后利用Arrays.copyOf()把elementData复制到新数组中并赋值给elementData；

最后把新加数据放到数组中elementData[size+1]=object;



add方法分析完了，就可以弄清楚几个重点了，**首先是无参构造初始化的集合在调用一次add过后就会扩展数组长度为10**，**除此之外每次扩容都是在原有长度的基础上扩展二分之一**。



**其他方法**

add(int index, E element)方法步骤是首先验证index是否小于数组长度，然后扩容，把index及以后的元素后移一位，最后在数组的index处设置element；

addAll(Collection<? extends E> c)与addAll(int index, Collection<? extends E> c)方法与两个add方法差不多，是一次加一个还是加多个的区别，然后就是扩容的时候minCapacity可能会大于newCapacity；

移除方法remove(Object o)与remove(int index)则与add(int index, E element)方法相反，先是找到要删除的数组索引，然后把后面的元素前移一位，最后把--size的地方置为null。

**ArrayList还有一个int属性modCount，不管是添加方法还是移除方法，每次成功都会对modCount加1，也就是相当于记录ArrayList的修改次数。**





最后就是迭代方法，我们知道集合的foreach 最终都是依赖内部的一个Iterator的实现类Itr，**在new一个Itr的实例的时候会保存当前的modCount，next方法首先就会校验记录modCount与集合的modCount是否一致**，所以是不允许在 foreach 循环里进行元素的 remove/add 操作。

那么为什么要这么做呢？为什么这种遍历的时候不让删除呢？前面分析我们知道当删除元素后，后面的元素会往前移，比如现在遍历到索引2然后删除了它，原来3的位置就到了2，下次遍历就是3了，那么实际上原来在3位置的元素就没有遍历到。

实在需要遍历删除，不要使用ArrayList的remove，改为用Iterator的remove即可。

**总结**

ArrayList是最简单常用的类，源码也不复杂，不过还是不要忘了他的几个基本原理，比如是怎么扩容的，foreach 不能进行remove/add 操作。不然不小心一个面试面到相关问题回答不上来就很尴尬了。









