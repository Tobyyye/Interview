

https://www.jianshu.com/p/4bcfffb27ed5



![img](https://upload-images.jianshu.io/upload_images/5148507-ca8930bca4e10d05.png?imageMogr2/auto-orient/strip|imageView2/2/w/830/format/webp)







mysql不是每次数据更改都立刻写到磁盘，而是会先将修改后的结果暂存在内存中,当一段时间后，再一次性将多个修改写到磁盘上，减少磁盘io成本，同时提高操作速度。



**mysql通过WAL(write-ahead logging)技术保证事务**
 在同一个事务中，每当数据库进行修改数据操作时，将修改结果更新到内存后，会在redo log添加一行记录记录“需要在哪个数据页上做什么修改”，并将该记录状态置为prepare，等到commit提交事务后，会将此次事务中在redo log添加的记录的状态都置为commit状态，之后将修改落盘时，会将redo log中状态为commit的记录的修改都写入磁盘。过程如下图





![img](https://upload-images.jianshu.io/upload_images/5148507-ce39f679f490b64f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)







write pos表示日志当前记录的位置，当ib_logfile_4写满后，会从ib_logfile_1从头开始记录；check point表示将日志记录的修改写进磁盘，完成数据落盘，数据落盘后checkpoint会将日志上的相关记录擦除掉，即write pos->checkpoint之间的部分是redo log空着的部分，用于记录新的记录，checkpoint->write pos之间是redo log待落盘的数据修改记录。当writepos追上checkpoint时，得先停下记录，先推动checkpoint向前移动，空出位置记录新的日志。
 **有了redo log，当数据库发生宕机重启后，可通过redo log将未落盘的数据恢复，即保证已经提交的事务记录不会丢失。**
 **有了redo log，为啥还需要binlog呢？**



1、redo log的大小是固定的，日志上的记录修改落盘后，日志会被覆盖掉，无法用于数据回滚/数据恢复等操作。
2、redo log是innodb引擎层实现的，并不是所有引擎都有。



https://www.cnblogs.com/f-ck-need-u/archive/2018/05/08/9010872.html



innodb事务日志包括redo log和undo log。redo log是重做日志，提供前滚操作，undo log是回滚日志，提供回滚操作。

undo log不是redo log的逆向过程，其实它们都算是用来恢复的日志：
**1.redo log通常是物理日志，记录的是数据页的物理修改，而不是某一行或某几行修改成怎样怎样，它用来恢复提交后的物理数据页(恢复数据页，且只能恢复到最后一次提交的位置)。**
**2.undo用来回滚行记录到某个版本。undo log一般是逻辑日志，根据每行记录进行记录。**







https://hiddenpps.blog.csdn.net/article/details/108505371





日志是 `mysql` 数据库的重要组成部分，记录着数据库运行期间各种状态信息。`mysql`日志主要包括错误日志、查询日志、慢查询日志、事务日志、二进制日志几大类。

作为开发，我们重点需要关注的是二进制日志( `binlog` )和事务日志(包括`redo log` 和 `undo log` )，本文接下来会详细介绍这三种日志。

## binlog

`binlog` 用于记录数据库执行的写入性操作(不包括查询)信息，以二进制的形式保存在磁盘中。`binlog` 是 `mysql`的逻辑日志，并且由 `Server` 层进行记录，使用任何存储引擎的 `mysql` 数据库都会记录 `binlog` 日志。

- **逻辑日志**：可以简单理解为记录的就是sql语句 。
- **物理日志**：`mysql` 数据最终是保存在数据页中的，物理日志记录的就是数据页变更 。

`binlog` 是通过追加的方式进行写入的，可以通过`max_binlog_size` 参数设置每个 `binlog`文件的大小，当文件大小达到给定值之后，会生成新的文件来保存日志。

### binlog使用场景

在实际应用中， `binlog` 的主要使用场景有两个，分别是 **主从复制** 和 **数据恢复** 。

1. **主从复制** ：在 `Master` 端开启 `binlog` ，然后将 `binlog`发送到各个 `Slave` 端， `Slave` 端重放 `binlog` 从而达到主从数据一致。
2. **数据恢复** ：通过使用 `mysqlbinlog` 工具来恢复数据。

### binlog刷盘时机

对于 `InnoDB` 存储引擎而言，只有在事务提交时才会记录`biglog` ，此时记录还在内存中，那么 `biglog`是什么时候刷到磁盘中的呢？

`mysql` 通过 `sync_binlog` 参数控制 `biglog` 的刷盘时机，取值范围是 `0-N`：

- 0：不去强制要求，由系统自行判断何时写入磁盘；
- 1：每次 `commit` 的时候都要将 `binlog` 写入磁盘；
- N：每N个事务，才会将 `binlog` 写入磁盘。

从上面可以看出， `sync_binlog` 最安全的是设置是 `1` ，这也是`MySQL 5.7.7`之后版本的默认值。但是设置一个大一些的值可以提升数据库性能，因此实际情况下也可以将值适当调大，牺牲一定的一致性来获取更好的性能。

### binlog日志格式

`binlog` 日志有三种格式，分别为 `STATMENT` 、 `ROW` 和 `MIXED`。

> 在 `MySQL 5.7.7` 之前，默认的格式是 `STATEMENT` ， `MySQL 5.7.7` 之后，默认值是 `ROW`。日志格式通过 `binlog-format` 指定。

- `STATMENT`：基于`SQL` 语句的复制( `statement-based replication, SBR` )，每一条会修改数据的sql语句会记录到`binlog` 中  。
- - 优点：不需要记录每一行的变化，减少了 binlog 日志量，节约了 IO  , 从而提高了性能；
  - 缺点：在某些情况下会导致主从数据不一致，比如执行sysdate() 、  slepp()  等 。
- `ROW`：基于行的复制(`row-based replication, RBR` )，不记录每条sql语句的上下文信息，仅需记录哪条数据被修改了 。
- - 优点：不会出现某些特定情况下的存储过程、或function、或trigger的调用和触发无法被正确复制的问题 ；
  - 缺点：会产生大量的日志，尤其是` alter table ` 的时候会让日志暴涨
- `MIXED`：基于`STATMENT` 和 `ROW` 两种模式的混合复制(`mixed-based replication, MBR` )，一般的复制使用`STATEMENT` 模式保存 `binlog` ，对于 `STATEMENT` 模式无法复制的操作使用 `ROW` 模式保存 `binlog`

## redo log

### 为什么需要redo log

我们都知道，事务的四大特性里面有一个是 **持久性** ，具体来说就是**只要事务提交成功，那么对数据库做的修改就被永久保存下来了，不可能因为任何原因再回到原来的状态** 。

那么 `mysql`是如何保证一致性的呢？

最简单的做法是在每次事务提交的时候，将该事务涉及修改的数据页全部刷新到磁盘中。但是这么做会有严重的性能问题，主要体现在两个方面：

1. 因为 `Innodb` 是以 `页` 为单位进行磁盘交互的，而一个事务很可能只修改一个数据页里面的几个字节，这个时候将完整的数据页刷到磁盘的话，太浪费资源了！
2. 一个事务可能涉及修改多个数据页，并且这些数据页在物理上并不连续，使用随机IO写入性能太差！

因此 `mysql` 设计了 `redo log` ， **具体来说就是只记录事务对数据页做了哪些修改**，这样就能完美地解决性能问题了(相对而言文件更小并且是顺序IO)。



### redo log基本概念

`redo log` 包括两部分：一个是内存中的日志缓冲( `redo log buffer` )，另一个是磁盘上的日志文件( `redo logfile`)。

`mysql` 每执行一条 `DML` 语句，先将记录写入 `redo log buffer`，后续某个时间点再一次性将多个操作记录写到 `redo log file`。这种 **先写日志，再写磁盘** 的技术就是 `MySQL`
里经常说到的 `WAL(Write-Ahead Logging)` 技术。

在计算机操作系统中，用户空间( `user space` )下的缓冲区数据一般情况下是无法直接写入磁盘的，中间必须经过操作系统内核空间( `kernel space` )缓冲区( `OS Buffer` )。

因此， `redo log buffer` 写入 `redo logfile` 实际上是先写入 `OS Buffer` ，然后再通过系统调用 `fsync()` 将其刷到 `redo log file`
中，过程如下：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9CYXE1bFlwSXc3WE9ZN0V1d24xS0hjOFBXeGhrUjdaZTk3MXpJTlhWd3FwMHNrZW9Za21PUXBHQXRjSmpjcmZYYTFkaDl2eEJERUlvbVl3OVkxdGlidGcvNjQw?x-oss-process=image/format,png)

`mysql` 支持三种将 `redo log buffer` 写入 `redo log file` 的时机，可以通过 `innodb_flush_log_at_trx_commit` 参数配置，各参数值含义如下：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9UZVlrNDc4VzM2Qkg2cjlZaWFIbWljb0ZVZWljUm1DQzZjaGdZcFRzUlU2d1FJcDQyUjBKQ1ZLY0RwVkNsUlFFVHN6TGxpYk5CTk82blpJV1dYQTQ3NWNpYmhnLzY0MA?x-oss-process=image/format,png)

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9CYXE1bFlwSXc3WE9ZN0V1d24xS0hjOFBXeGhrUjdaZVRpYWxCc0VtbVpaVlNpYUx5QVJBSnZYaWF6eG54UjRpY2hhbVBzVkxSNFBYNEJ5RTEwZ0RPYTVVM1EvNjQw?x-oss-process=image/format,png)





### redo log记录形式

前面说过， `redo log` 实际上记录数据页的变更，而这种变更记录是没必要全部保存，因此 `redo log`实现上采用了大小固定，循环写入的方式，当写到结尾时，会回到开头循环写日志。如下图：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9CYXE1bFlwSXc3WE9ZN0V1d24xS0hjOFBXeGhrUjdaZU5URG1zN09DUEY4VUc5ODVHaldlVnhwajFXekNYMHpleE5zanh3aWFsU0lwTDN5bVpRVUVCQWcvNjQw?x-oss-process=image/format,png)

同时我们很容易得知， 在innodb中，既有`redo log` 需要刷盘，还有 `数据页` 也需要刷盘， `redo log`存在的意义主要就是降低对 `数据页` 刷盘的要求 ** 。

在上图中， `write pos` 表示 `redo log` 当前记录的 `LSN` (逻辑序列号)位置， `check point` 表示 **数据页更改记录** 刷盘后对应 `redo log` 所处的 `LSN`(逻辑序列号)位置。

`write pos` 到 `check point` 之间的部分是 `redo log` 空着的部分，用于记录新的记录；`check point` 到 `write pos` 之间是 `redo log` 待落盘的数据页更改记录。当 `write pos`追上`check point` 时，会先推动 `check point` 向前移动，空出位置再记录新的日志。

启动 `innodb` 的时候，不管上次是正常关闭还是异常关闭，总是会进行恢复操作。因为 `redo log`记录的是数据页的物理变化，因此恢复的时候速度比逻辑日志(如 `binlog` )要快很多。

重启`innodb` 时，首先会检查磁盘中数据页的 `LSN` ，如果数据页的`LSN` 小于日志中的 `LSN` ，则会从 `checkpoint` 开始恢复。

还有一种情况，在宕机前正处于`checkpoint` 的刷盘过程，且数据页的刷盘进度超过了日志页的刷盘进度，此时会出现数据页中记录的 `LSN` 大于日志中的 `LSN`，这时超出日志进度的部分将不会重做，因为这本身就表示已经做过的事情，无需再重做。

### redo log与binlog区别

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9UZVlrNDc4VzM2Qkg2cjlZaWFIbWljb0ZVZWljUm1DQzZjaHhuc2pLbnBjTGgzUUw1YmVHb3ZzRzM0ZFhtOHZETmdPVkVMN0VodVcxUXk3dHBpYVFZTzVlbGcvNjQw?x-oss-process=image/format,png)

由 `binlog` 和 `redo log` 的区别可知：`binlog` 日志只用于归档，只依靠 `binlog` 是没有 `crash-safe` 能力的。

但只有 `redo log` 也不行，因为 `redo log` 是 `InnoDB`特有的，且日志上的记录落盘后会被覆盖掉。因此需要 `binlog`和 `redo log`二者同时记录，才能保证当数据库发生宕机重启时，数据不会丢失。





## undo log

数据库事务四大特性中有一个是 **原子性** ，具体来说就是 **原子性是指对数据库的一系列操作，要么全部成功，要么全部失败，不可能出现部分成功的情况**。

实际上， **原子性** 底层就是通过 `undo log` 实现的。`undo log`主要记录了数据的逻辑变化，比如一条 `INSERT` 语句，对应一条`DELETE` 的 `undo log` ，对于每个 `UPDATE` 语句，对应一条相反的 `UPDATE` 的 `undo log` ，这样在发生错误时，就能回滚到事务之前的数据状态。

同时， `undo log` 也是 `MVCC`(多版本并发控制)实现的关键。



***\*一、\** 比如一个有个事务插入persion表插入了一条新记录，记录如下，`name`为Jerry, `age`为24岁，`隐式主键`是1，`事务ID`和`回滚指针`，我们假设为NULL**

![img](https://img-blog.csdnimg.cn/img_convert/9a81a884081afc06afdff8c0260b4698.png)

***\*二、\** 现在来了一个`事务1`对该记录的`name`做出了修改，改为Tom**

- 在`事务1`修改该行(记录)数据时，数据库会先对该行加`排他锁`
- 然后把该行数据拷贝到`undo log`中，作为旧记录，既在`undo log`中有当前行的拷贝副本
- 拷贝完毕后，修改该行`name`为Tom，并且修改隐藏字段的事务ID为当前`事务1`的ID, 我们默认从`1`开始，之后递增，回滚指针指向拷贝到`undo log`的副本记录，既表示我的上一个版本就是它
- 事务提交后，释放锁

![img](https://img-blog.csdnimg.cn/img_convert/f8137f5cacfb0894debeea5ba61cc6ab.png)

***\*三、\** 又来了个`事务2`修改`person表`的同一个记录，将`age`修改为30岁**

- 在`事务2`修改该行数据时，数据库也先为该行加锁
- 然后把该行数据拷贝到`undo log`中，作为旧记录，发现该行记录已经有`undo log`了，那么最新的旧数据作为链表的表头，插在该行记录的`undo log`最前面
- 修改该行`age`为30岁，并且修改隐藏字段的事务ID为当前`事务2`的ID, 那就是`2`，回滚指针指向刚刚拷贝到`undo log`的副本记录
- 事务提交，释放锁

![img](https://img-blog.csdnimg.cn/img_convert/a2650a111ef02211980d7368abefddce.png)

从上面，我们就可以看出，不同事务或者相同事务的对同一记录的修改，会导致该记录的`undo log`成为一条记录版本线性表，既链表，`undo log`的链首就是最新的旧记录，链尾就是最早的旧记录（**当然就像之前说的该undo log的节点可能是会purge线程清除掉，向图中的第一条insert undo log，其实在事务提交之后可能就被删除丢失了，不过这里为了演示，所以还放在这里**）









