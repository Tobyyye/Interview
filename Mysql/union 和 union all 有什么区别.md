

一、区别1：取结果的交集

1、union: 对两个结果集进行并集操作, 不包括重复行,相当于distinct, 同时进行默认规则的排序;

2、union all: 对两个结果集进行并集操作, 包括重复行, 即所有的结果全部显示, 不管是不是重复;

二、区别2：获取结果后的操作

1、union: 会对获取的结果进行排序操作

2、union all: 不会对获取的结果进行排序操作

三、区别3：

1、union看到结果中ID=3的只有一条

select * from student2 where id < 4

union

select * from student2 where id > 2 and id < 6

2、union all 结果中ID=3的结果有两个

select * from student2 where id < 4

union all

select * from student2 where id > 2 and id < 6

四、总结

union all只是合并查询结果，并不会进行去重和排序操作，在没有去重的前提下，使用union all的执行效率要比union高

