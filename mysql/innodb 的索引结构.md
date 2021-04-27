innodb 的索引结构是什么？什么是聚簇索引？ 

**InnoDB默认创建的主键索引是聚簇索引（Clustered Index），其它索引都属于辅助索引（Secondary Index），也被称为二级索引或非聚簇索引。**



https://www.cnblogs.com/jiawen010/p/11805241.html



**InnoDB中，表数据文件本身就是按B+Tree组织的一个索引结构，聚簇索引就是按照每张表的主键构造一颗B+树，同时叶子节点中存放的就是整张表的行记录数据，也将聚集索引的叶子节点称为数据页。这个特性决定了索引组织表中数据也是索引的一部分；**

　　**一般建表会用一个自增主键做\**聚簇索引，没有的话MySQL会默认创建，但是这个主键如果更改代价较高，故建表时要考虑自增ID不能频繁update这点。\****

　　**我们日常工作中，根据实际情况自行添加的索引都是辅助索引，辅助索引就是一个为了需找主键索引的二级索引，现在找到主键索引再通过主键索引找数据；**

聚簇索引并不是一种单独的索引类型，而**是一种数据存储方式**。具体细节依赖于其实现方式。

MySQL数据库中innodb存储引擎，B+树索引可以分为聚簇索引（也称聚集索引，clustered index）和辅助索引（有时也称非聚簇索引或二级索引，secondary index，non-clustered index）。这两种索引内部都是B+树，聚集索引的叶子节点存放着一整行的数据。

Innobd中的主键索引是一种聚簇索引，非聚簇索引都是辅助索引，像复合索引、前缀索引、唯一索引。

Innodb使用的是聚簇索引，MyISam使用的是非聚簇索引

### 辅助索引（非聚簇索引）

　　在**聚簇索引之上创建的索引称之为辅助索引**，辅助索引访问数据总是需要二次查找。辅助索引叶子节点存储的不再是行的物理位置，而是主键值。通过辅助索引首先找到的是主键值，再通过主键值找到数据行的数据页，再通过数据页中的Page Directory找到数据行。

　　Innodb辅助索引的叶子节点并**不包含行记录的全部数据**，叶子节点除了包含键值外，还包含了相应行数据的聚簇索引键。

　　辅助索引的存在不影响数据在聚簇索引中的组织，所以一张表可以有多个辅助索引。在innodb中有时也称辅助索引为二级索引。







https://www.cnblogs.com/chenkeyu/p/12686940.html

## **MySQL InnoDB的聚簇索引和二级索引**

了解了B+树，现在就可以很容易区分MySQL的聚簇索引和二级索引。

聚簇索引就是用**主键生成B+树**，在叶子节点**存放这条记录的完整信息**

二级索引就是用**索引行生成B+树**，在叶子节点**只存放索引行和该行对应的主键信息**

下面是聚簇索引和二级索引的区分图

![img](https://img2020.cnblogs.com/blog/1003414/202004/1003414-20200412185846052-323451290.png)



![img](https://img2020.cnblogs.com/blog/1003414/202004/1003414-20200412185851336-922704489.png)



了解上面的知识，对于一个查询，我们就可以大概想象出他的执行步骤

```
select * from books where name = "name400";
```

例如上面sql的执行步骤如下:

　　1. 在二级索引idx_books_name索引中查找name="name400"的字段所对应的主键id

　　2. 通过主键id在聚簇索引找到此id所对应的记录

　　3. 返回记录中的所有字段

当我们select的字段在二级索引上不存在时，都需要使用聚簇索引**回表查询**剩余字段。所以聚簇索引，也就是我们所说的id列，占用空间越小越好， 这样就可以在一个节点中存放更多的id值，减少树的层级，加速查询效率。一般推荐主键使用int或者bigint而不是字符串。同时最好保证插入的id值为递增的，这样就不会造成在一个已经满的节点中插入一条记录造成页分裂，降低查询效率。





https://blog.csdn.net/thesprit/article/details/112989674



MySQL会分别创建主键id的聚簇索引和age的二级索引:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210122164359174.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RoZXNwcml0,size_16,color_FFFFFF,t_70#pic_center)



在MySQL中主键索引的叶子节点存的是整行数据，而二级索引叶子节点内容是主键的值。

四、二级索引的检索过程
在MySQL的查询过程中，SQL优化器会选择合适的索引进行检索，在使用二级索引的过程中，因为二级索引没有存储全部的数据，假如二级索引满足查询需求(一般来说，比如只查id和当前列属性)，则直接返回，即为覆盖索引，反之则需要回表去主键索引(聚簇索引)查询。

例如执行

SELECT * FROM users WHERE age=35;


![在这里插入图片描述](https://img-blog.csdnimg.cn/2021012216441358.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RoZXNwcml0,size_16,color_FFFFFF,t_70#pic_center)