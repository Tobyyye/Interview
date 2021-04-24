# Mybatis常见面试题（一）

作者：小谢backup
链接：https://www.nowcoder.com/discuss/611747?type=all&order=time&pos=&page=1&channel=-1&source_id=search_all_nctrack
来源：牛客网

https://www.nowcoder.com/discuss/384244?type=all&order=time&pos=&page=0&channel=-1&source_id=search_all_nctrack

## **1**、**MyBatis** 中 **#{}**和 **${}**的区别是什么？ 

 \#{}是预编译处理，${}是字符替换。 

 在使用 #{}时，MyBatis 会将 SQL 中的 #{}替换成“?”，配合 PreparedStatement 的 set 方法赋值，这样可以有效的防止 SQL 注入，保证程序的运行安全。 

 
 

##  **2**、**MyBatis** 有哪两种分页方式？ 

 逻辑分页： 使用 MyBatis 自带的 RowBounds 进行分页，它是一次性查询很多数据，然后在数据中再进行检索。 

 物理分页： 自己手写 SQL 分页或使用分页插件 PageHelper，去数据库查询指定条数的分页数据的形式。 

 
 

##  **3**、**RowBounds** 是一次性查询全部结果吗？为什么？ 

 不是。RowBounds 表面是在“所有”数据中检索数据，其实并非是一次性查询出所有数据。 

 因为 MyBatis 是对 jdbc 的封装，在 jdbc 驱动中有一个 Fetch Size 的配置，它规定了每次最多从数据库查询多少条数据，假如你要查询更多数据，它会在你执行 next()的时候，去查询更多的数据。 

 就好比你去自动取款机取 10000 元，但取款机每次最多能取 2500 元，所以你要取 4 次才能把钱取完。只是对于 jdbc 来说，当你调用 next()的时候会自动帮你完成查询工作。这样做的好处可以有效的防止内存溢出。 

 
 

##  **4**、**MyBatis** 是否支持延迟加载？延迟加载的原理是什么？ 

 MyBatis 支持延迟加载，设置 lazyLoadingEnabled=true 即可。 

 延迟加载的原理的是调用的时候触发加载，而不是在初始化的时候就加载信息。比如调用 a. getB(). getName()，这个时候发现 a. getB() 的值为 null，此时会单独触发事先保存好的关联 B 对象的 SQL，先查询出来 B，然后再调用 a. setB(b)，而这时候再调用 a. getB(). getName() 就有值了，这就是延迟加载的基本原理。 

 
 

 当然了，不光是Mybatis，几乎所有的包括Hibernate，支持延迟加载的原理都是一样的。 

 
 

##  **5**、说一下 **MyBatis** 的一级缓存和二级缓存？ 

 一级缓存：基于 PerpetualCache 的 HashMap 本地缓存，它的生命周期是和 SQLSession 一致的，有多个 SQLSession 或者分布式的环境中数据库操作，可能会出现脏数据。当 Session flush 或 close 之后，该 Session 中的所有 Cache 就将清空，默认一级缓存是开启的。 

 
 

 二级缓存：也是基于 PerpetualCache 的 HashMap 本地缓存，不同在于其存储作用域为 Mapper 级别的。如果多个SQLSession之间需要共享缓存，则需要使用到二级缓存，并且二级缓存可自定义存储源，如 Ehcache。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现 Serializable 序列化接口(可用来保存对象的状态)。 

 
 

 开启二级缓存数据查询流程：二级缓存 -> 一级缓存 -> 数据库。 

 缓存更新机制：当某一个作用域（一级缓存 Session/二级缓存 Mapper）进行了C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear。 

 
 

##  **6**、**MyBatis** 有哪些执行器（**Executor**）？ 

 MyBatis 有三种基本的Executor执行器： 

 SimpleExecutor：每执行一次 update 或 select 就开启一个 Statement 对象，用完立刻关闭 Statement 对象； 

 ReuseExecutor：执行 update 或 select，以 SQL 作为 key 查找 Statement 对象，存在就使用，不存在就创建，用完后不关闭 Statement 对象，而是放置于 Map 内供下一次使用。简言之，就是重复使用 Statement 对象； 

 BatchExecutor：执行 update（没有 select，jdbc 批处理不支持 select），将所有 SQL 都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个 Statement 对象，每个 Statement 对象都是 addBatch()完毕后，等待逐一执行 executeBatch()批处理，与 jdbc 批处理相同。 

 
 

 作用范围：Executor的这些特点，都严格限制在SqlSession生命周期范围内。 

 
 

##  **7**、**Mybatis**有哪四大对象？ 

 Executor、ParameterHandler、StatementHandler、ResultSetHandler 

 
 

##  **8**、**MyBatis** 分页插件的实现原理是什么？ 

 分页插件的基本原理是使用 MyBatis 提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的 SQL，然后重写 SQL，根据 dialect 方言，添加对应的物理分页语句和物理分页参数。 

 
 

##  **9**、什么是**MyBatis**的接口绑定？有哪些实现方式？ 

 接口绑定，就是在MyBatis中任意定义接口，然后把接口里面的方法和SQL语句绑定，我们直接调用接口方法就可以，这样比起原来了SqlSession提供的方法我们可以有更加灵活的选择和设置。 

 
 

 接口绑定有两种实现方式： 

 通过注解绑定，就是在接口的方法上面加上 @Select、@Update等注解，里面包含Sql语句来绑定； 

  通过xml里面写SQL来绑定， 在这种情况下，要指定xml映射文件里面的namespace必须为接口的全路径名。当Sql语句比较简单时候，用注解绑定， 当SQL语句比较复杂时候，用xml绑定，一般用xml绑定的比较多。 

  
 

##  **10**、**Mybatis**动态**sql**是做什么的？都有哪些动态**sql**？能简述一下动态**sql**的执行原理不？ 

 Mybatis动态sql可以让我们在Xml映射文件内，以标签的形式编写动态sql，完成逻辑判断和动态拼接sql的功能，Mybatis提供了9种动态sql标签： 

 trim、where、set、foreach|、if、choose、when、otherwise、bind。 

 
 

  其执行原理为，使用 OGNL从 sql参数对象中计算表达式的值，根据表达式的值动态拼接 sql，以此来完成动态 sql的功能。