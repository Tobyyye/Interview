# Mysql 中 MyISAM 和 InnoDB 的区别有哪些



先看下《高性能MySQL》中对于他们的评价：

**Innodb引擎概述**

**InnoDB：**MySQL默认的事务型引擎，也是最重要和使用最广泛的存储引擎。它被设计成为大量的短期事务，短期事务大部分情况下是正常提交的，很少被回滚。InnoDB的性能与自动崩溃恢复的特性，使得它在非事务存储需求中也很流行。除非有非常特别的原因需要使用其他的存储引擎，否则应该优先考虑InnoDB引擎。

Innodb引擎提供了对数据库ACID事务的支持，并且实现了SQL标准的四种隔离级别。该引擎还提供了行级锁和外键约束，它的设计目标是处理大容量数据库系统，它本身其实就是基于MySQL后台的完整数据库系统，MySQL运行时Innodb会在内存中建立缓冲池，用于缓冲数据和索引。但是该引擎不支持FULLTEXT类型的索引，而且它没有保存表的行数，当SELECT COUNT(*) FROM TABLE时需要扫描全表。当需要使用数据库事务时，该引擎当然是首选。由于锁的粒度更小，写操作不会锁定全表，所以在并发较高时，使用Innodb引擎会提升效率。但是使用行级锁也不是绝对的，如果在执行一个SQL语句时MySQL不能确定要扫描的范围，InnoDB表同样会锁全表。



**MyISAM引擎概述**

**MyISAM：**在MySQL 5.1 及之前的版本，MyISAM是默认引擎。MyISAM提供的大量的特性，包括全文索引、压缩、空间函数（GIS）等，但MyISAM并不支持事务以及行级锁，而且一个毫无疑问的缺陷是崩溃后无法安全恢复。正是由于MyISAM引擎的缘故，即使MySQL支持事务已经很长时间了，在很多人的概念中MySQL还是非事务型数据库。尽管这样，它并不是一无是处的。对于只读的数据，或者表比较小，可以忍受修复操作，则依然可以使用MyISAM（但请不要默认使用MyISAM，而是应该默认使用InnoDB）

MyISAM是MySQL默认的引擎，但是它没有提供对数据库事务的支持，也不支持行级锁和外键，因此当INSERT(插入)或UPDATE(更新)数据时即写操作需要锁定整个表，效率便会低一些。不过和Innodb不同，MyISAM中存储了表的行数，于是SELECT COUNT(*) FROM TABLE时只需要直接读取已经保存好的值而不需要进行全表扫描。如果表的读操作远远多于写操作且不需要数据库事务的支持，那么MyISAM也是很好的选择。

1、 **存储结构**

**MyISAM：**每个MyISAM在磁盘上存储成三个文件。分别为：**表定义文件、数据文件、索引文件。**第一个文件的名字以表的名字开始，扩展名指出文件类型。.frm文件存储表定义。数据文件的扩展名为.MYD (MYData)。索引文件的扩展名是.MYI (MYIndex)。

**InnoDB：**所有的表都保存在同一个数据文件中（也可能是多个文件，或者是独立的表空间文件），InnoDB表的大小只受限于操作系统文件的大小，一般为2GB。

2、 **存储空间**

**MyISAM：** MyISAM支持支持三种不同的存储格式：静态表(默认，但是注意数据末尾不能有空格，会被去掉)、动态表、压缩表。当表在创建之后并导入数据之后，不会再进行修改操作，可以使用压缩表，极大的减少磁盘的空间占用。

存储空间 MyISAM：可被压缩，存储空间较小 

**InnoDB：** 需要更多的内存和存储，它会在主内存中建立其专用的缓冲池用于高速缓冲数据和索引。

3、 **可移植性、备份及恢复**

**MyISAM：**数据是以文件的形式存储，所以在跨平台的数据转移中会很方便。在备份和恢复时可单独针对某个表进行操作。

**InnoDB：**免费的方案可以是拷贝数据文件、备份 binlog，或者用 mysqldump，在数据量达到几十G的时候就相对痛苦了。

4、 **事务支持**

**MyISAM：**强调的是性能，每次查询具有原子性,其执行数度比InnoDB类型更快，但是不提供事务支持。

**InnoDB：**提供事务支持事务，外部键等高级数据库功能。 具有事务(commit)、回滚(rollback)和崩溃修复能力(crash recovery capabilities)的事务安全(transaction-safe (ACID compliant))型表。

5、 **AUTO_INCREMENT** **MyISAM：**可以和其他字段一起建立联合索引。引擎的自动增长列必须是索引，如果是组合索引，自动增长可以不是第一列，他可以根据前面几列进行排序后递增。

**InnoDB：**InnoDB中必须包含只有该字段的索引。引擎的自动增长列必须是索引，如果是组合索引也必须是组合索引的第一列。

6、 **表锁差异**

**MyISAM：** 只支持表级锁，用户在操作myisam表时，select，update，delete，insert语句都会给表自动加锁，如果加锁以后的表满足insert并发的情况下，可以在表的尾部插入新的数据。

**InnoDB：** 支持事务和行级锁，是innodb的最大特色。行锁大幅度提高了多用户并发操作的新能。但是InnoDB的行锁，只是在WHERE的主键是有效的，非主键的WHERE都会锁全表的。

7、 **全文索引**

[MySql全文索引](https://link.zhihu.com/?target=https%3A//blog.csdn.net/u013276277/article/details/78139159)

**MyISAM：**支持 FULLTEXT类型的全文索引

**InnoDB：**不支持FULLTEXT类型的全文索引，但是innodb可以使用sphinx插件支持全文索引，并且效果更好。

8、**表主键**

**MyISAM：**允许没有任何索引和主键的表存在，索引都是保存行的地址。

**InnoDB：**如果没有设定主键或者非空唯一索引，就会自动生成一个6字节的主键(用户不可见)，数据是主索引的一部分，附加索引保存的是主索引的值。

9、**表的具体行数**

**MyISAM：** 保存有表的总行数，如果select count(*) from table;会直接取出出该值。*

**InnoDB：** 没有保存表的总行数，如果使用select count(*) from table；就会遍历整个表，消耗相当大，但是在加了wehre条件后，myisam和innodb处理的方式都一样。

10、**CRUD操作**

**MyISAM：**如果执行大量的SELECT，MyISAM是更好的选择。

**InnoDB：**如果你的数据执行大量的INSERT或UPDATE，出于性能方面的考虑，应该使用InnoDB表。

11、 外键

**MyISAM：**不支持

**InnoDB：**支持



**简单介绍区别**

1、MyISAM是非事务安全的，而InnoDB是事务安全的

2、MyISAM锁的粒度是表级的，而InnoDB支持行级锁

3、MyISAM支持全文类型索引，而InnoDB不支持全文索引

4、MyISAM相对简单，效率上要优于InnoDB，小型应用可以考虑使用MyISAM

5、MyISAM表保存成文件形式，跨平台使用更加方便

**应用场景**

1、MyISAM管理非事务表，提供高速存储和检索以及全文搜索能力，如果再应用中执行大量select操作，应该选择MyISAM

2、InnoDB用于事务处理，具有ACID事务支持等特性，如果在应用中执行大量insert和update操作，应该选择InnoDB