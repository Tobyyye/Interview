对 uuid 的理解？

知道哪些 GUID、Random [算法]()？ 

https://zhuanlan.zhihu.com/p/259802265





# UUID 及其在 MySQL 中的使用

# 



# [MySQL生成UUID](https://www.cnblogs.com/codecat/p/10833523.html)

## UUID函数

在MySQL中，可以用uuid()函数来生成一个UUID，如下图：

![img](https://img2018.cnblogs.com/blog/24973/201905/24973-20190508175809858-413830655.png)

## replace函数

默认生成的uuid含有'-'，我们可以使用replace函数替换掉'-'，SQL如下：

```
select replace(uuid(),"-","") as uuid;
```

结果如下：

![img](https://img2018.cnblogs.com/blog/24973/201905/24973-20190508180114754-501666014.png)

## Insert语句中使用UUID

如果一个表中id字段使用uuid来作为主键，那我们可以使用下面的语句来插入数据：

```
INSERT INTO t_inventive_principle (`id`,`code_num`,`name`) VALUES (REPLACE(UUID(),"-",""),1,'分割原理');
```

结果如下：

![img](https://img2018.cnblogs.com/blog/24973/201905/24973-20190508180439721-405015639.png)

在update语句中使用也跟insert语句中一样，就不写例子了。





在复杂的分布式系统中，需要对大量的数据和消息进行唯一的标识。比如美团的金融、支付、餐饮、酒店、猫眼电影等产品的系统中，数据日益增长，将数据库分表后需要一个唯一的ID来标示一条数据或消息，数据库的自增ID显然不能满足需求,特别是订单、骑手、优惠券也需要有唯一的ID才能识别。

此时，一个能够生成全局唯一ID的系统是非常必要的。





https://baike.baidu.com/item/UUID/5921266?fr=aladdin

# 一、为什么要用分布式ID？







### UUID：

>   UUID全称：Universally Unique  Identifier，即通用唯一识别码。是一个由4个连字号(-)将32个字节长的字符串分隔后生成的字符串，总共36个字节长。比如：550e8400-e29b-41d4-a716-446655440000
>  

### UUID的作用 ：

>    UUID是让分布式系统中的所有元素都能有唯一的辨识信息，而不要要通过中央控制端来做辨识信息的指定。如此一来，每个人都可以创建不与其他人冲突的UUID。在这样的情况下，就不需考虑数据库创建时的名称重复问题。目前最广泛应用的UUID，是微软公司的全局唯一标识符（GUID），而其他重要的应用，则有Linux ext2/ext3文件系统、LULS加密分区、GNOME、KDE、Mac OS X等等。
>  

### UUID的组成 :

>   UUID是指在一台机器上生成的数字，它保证对在同一时空中的所有机器都是唯一的。通常平台会提供生成的API。按照开放软件基金会（OSF）制定的标准计算，用到了以太网卡地址、纳秒级时间、芯片ID码和许多可能的数字。
>  

### UUID由以下几部分的组合 :

>   当前日期和时间，UUID的第一个部分与时间有关，如果你在生成一个UUID之后，过几秒后又生成了一个UUID，则第一个部分不同，其余相同。  时钟序列。  全局唯一的IEEE机器识别号，如果有网卡，从网卡MAC地址获得，没有网卡以其他方式获得。
>  UUID的唯一缺陷在于生成的结果穿会比较长。关于UUID这个标准使用最普遍的是微软的GUID（Globals Ujique Identifiers）。
>  

### **GUID：**

>   是微软对UUID这个标准的实现。UUID是由开放软件基金会（OSF）定义的。UUID还有其它各种实现，不止GUID一种。比如我们这里在Java中用到的。