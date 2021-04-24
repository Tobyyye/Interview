![img](https://img-blog.csdn.net/20170817122606046?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYXdxNTIwdHQxMzE0/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)





### Collection的实现



 我们可以看到collection接口是由两个子接口Set和List以及Map(不是collection的接口)组成的。下面我们分别看一下各个子接口的组成和实现。 



 List(interface): List可以通过index知道元素的位置，它允许元素的重复。ArrayList, LinkedList, Vector可以实现List接口。



 Set(interface):是不允许元素的重复。HashSet, LinkedHashSet,TreeSet 可以实现Set接口。



 Map(interface): 使用键值对(key-value), 值(value)可以重复，键(key)不可以重复。HashMap, LinkedHashMap, Hashtale, TreeMap可以实现Map接口。