https://blog.csdn.net/qq_41345281/article/details/89218522





Hash算法

在说HashMap之前先来了解一下Hash算法。在数据结构中学习过线性表，我们知道在线性表中查询一个值最坏的情况可能是从头遍历到尾，其平均时间复杂度为O(n),并不理想，而hash表能将查询的时间复杂度降为O(1),因为Hash算法会通过hash函数计算出地址直接取值，其查询次数只有一次。
通过下面例子简单了解一下hash表的查询方式，下面是一个hash表，首先假设hash函数为n%10,hash函数计算出来的结果就是其hash表中该元素的地址，所以10、13和26的存出结果如下图。
![img](https://img-blog.csdnimg.cn/20190411194600731.png)

可能实际的hash函数能很好地避免地址的冲突，但是还是有地址冲突的可能性，比如10和20。java中hashMap解决方式是"链地址法"，如下图，将hash值冲突的值使用链表连接起来，这样查询到地址0的时候就会依次比较10和20，看到底哪个才是要找的。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190411195154723.png)





HashMap
下面来了解什么是HashMap，“Map集合即Key-Value的集合，前面加个Hash，即散列，无序的。所以HashMap即散着的,无序的Key-Value集合”,这是最初我看到的一个对我个人而言比较好理解的解释。当我们使用的hashMap的键值为对象的时候可能就要重写hashCode和eqals。
先看下面一段代码
public class User {
	private String name;
	private String password;
	public User() {
		super();
	}
	public User(String name, String passed) {
		super();
		this.name = name;
		this.passed = passed;
	}
	
public class Demo {
	public static void main(String[] args) {
		User user1=new User("name", "passed");
		User user2=new User("name", "passed");
		//定义hashMap
		HashMap<User, String> hashmap=new HashMap<User,String>();
		//添加
		hashmap.put(user1, "value1");
		//通过user2获取添加的value
		String str=hashmap.get(user2);
		System.out.println("输出结果："+str);
	}
}



#### 总结

由上面的分析可知，**当键值为对象类型的时候就需要重写hashCode和equals方法。**



