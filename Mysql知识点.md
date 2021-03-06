## mysql知识点

#### mysql复制原理

##### mysql复制的基本流程

1.主：binlog线程记录所有改变了数据库数据的语句，放进master的binlog中

2.从: io线程 在使用 start slave之后，负责从master上拉取binlog内容，读取到relay log中

3.从: sql线程执行线程，执行relay log中的语句



#### mysql如何保证复制过程中的一致性

问题： 

​	mysql5.5之前 slave 的sql线程执行的relay log位置只能保存在 （relay-log.info）里面，并且该文件默认每执行10000次事务做一次同步到磁盘，当slave 意外重启时，sql线程执行的位置有可能和数据库的数据是不一致的，会造成复制报错，数据不一致。

1.mysql5.6 引入参数 relay_log_info_repository = TABLE ，Mysql将sql线程执行到的位置存到mysql.slave_relay_log_info表，这样更新该表的位置和sql线程执行的过程绑定维一个事务，这样slave宕机时，也会恢复到sql线程执行的位置和表中位置移植。

2.mysql5.6引入GTID复制，每个GTID对应的事物在每个实例上最多执行一次，这样也提高了复制的数据一致性

3.无损半同步复制：在切成半同步复制之前，事务是不提交的，而是接受到slave的ack确认之后才提交。



### mysql数据引擎

1.InnoDB

​	提供事物处理，回滚，崩溃修复能力和多版本并发控制的事物安全。

​	InnoDB存储引擎支撑 auto_increment，自增长的列不能为空，并且值必须唯一，mysql中规定自增长列必须为主键。

​	InnoDB支持外键

​	缺点：

​		读写效率差，占用空间相对较大

2.MyISAM

​	占用空间小，处理速度快；缺点是不支持事务的完整性和并发性。

3.Memory

​	所有数据都在内存中，速度快，安全性不高。

#### InnoDB和MyISAM的不同

1.InnoDB支持事务

2.InnoDB支持外键

3.InnoDB支持行锁

4.MyISAM支持全文索引，查询多的表适合使用MyISAM

5.InnoDB是索引组织表，MyISAM是堆表

#### mysql优化

1.设计数据库时，数据库表、字段的设计；存储引擎的选择

2.创建合适的索引

3.横向的扩展：Mysql集群，读写分离 主从备份等

4.sql语句的优化

##### 创建数据库 表字段的规范

1.尽可能的选择小的数据类型和指定短的长度

2.尽可能使用not null，not null 字段的处理要比null字段的处理高效

3.单表字段不宜过多，二三十就是极限了

4.尽可能使用整型表示字符串

5.定长和非定长数据类型的选择（dicimal 定长 double定长）

6.mysql5.6之后默认使用 InnoDB存储引擎

7.可以设置预留字段【】

##### 索引

关键字和数据的映射关系成为索引，关键字是从数据当中提取的用于标识、检索数据的特定内容。

索引检索为什么会快？

​	关键字相对于数据本身，数据量比较小

​	关键字是有序的，二分查找可以快速确定位置

索引的分类：

​	普通索引

​			普通索引在写多 多少的 操作中 使用 change buffer 作为优化

​	唯一索引：关键字不可以重复

​	主键索引：关键字唯一并且不为null （聚集索引）	

​	全文索引（InnoDB不支持）

索引的使用范围：

​	Where  Order By  join 等关键字后面使用的字段是索引的关键字可以提高检索效率

索引覆盖；

```java
如果要查询的字段都建立过索引，那么引擎会直接在索引表中查询不会访问原始数据，否则只要有一个
字段没建立索引就会做全表扫描，这个叫做索引覆盖，因此我们尽可能的在select 后只写必要查询字段
```

索引下推

什么是索引下推（Index Condition Pushdown，ICP）呢？假设有这么个需求，查询表中“名字第一个字是张，性别男，年龄为10岁的所有记录”。那么，查询语句是这么写的：

```
mysq> select * from tuser where name like '张 %' and age=10 and ismale=1;
索引下推 适用于 二级索引
索引下推一般可用于所求查询字段（select列）不是/不全是联合索引的字段，查询条件为多条件查询且查询条件子句（where/order by）字段全是联合索引
```

MySQL 5.6引入了索引下推优化，**可以在索引遍历过程中，对索引中包含的字段先做判断，过滤掉不符合条件的记录，减少回表字数**。



MRR(Multi-Range Read )

随机 IO 转化为顺序 IO 以降低查询过程中 IO 开销的一种手段，这对IO-bound类型的SQL语句性能带来极大的提升。

索引失效：

​	sql语句尽可能的使用到索引

​	1.字段单独使用

​		id +1 =20 用不到索引 	id=20-1使用索引

​	2.like查询，不可以使用通配符 %开头，可以使用 %结尾

​		like '%mysql%' 用不了索引， ‘mysql%’可以用到索引

​	3.or查询 两边都要有索引

​	4.INDEX(a,b,c)，当条件为a或a,b或a,b,c时都可以使用索引，但是当条件为b,c时将不会使用索引 

	5. string 和 id 比较 会走一个 cast 隐式转化， 所以没有使用到索引

索引的存储结构

​	1.BTree

​		普通索引 唯一索引 BTree

​		BTree的一个node可以存储多个关键字，BTree的查询效率为 log(x，N)，少量的磁盘读取就可以做到大量数据的遍历

​	2.B+

​		主键索引 聚簇结构使用 B+，关键字和记录是放在一起的。

​	3.哈希索引

索引选错

```java
原因：
    mysql优化器的回表时间也算进去，但是有可能不准
    采样统计
    
1.force index
2.使用覆盖索引+最左原则
```



##### 分表

1.水平分割：通过建立结构相同的几张表分别存储数据

2.垂直分割：将常用的字段放在一张表，不常使用的放在其他表中，设置一一对应的关系

问题：

分表后的id重复问题

1.创建一个表用来专门创建id，并获取最后插入的id，保证id的唯一性

2.使用redis生成id

#### 主从复制 读写分离

#### 监控sql

profile 分析一条sql从开始到结束的各个环节使用的时间

show profiles

show profile for query Query_id

### Msql基本知识点

```java
1.UNION
    合并两个或多个select的数据集，每个select语句必须有相同数量的列
    对合并的数值去重
2.UNION ALL
    合并的字段不会去重
3.not int(null, xx, xx) 查询出来为null
4.int (null, xx) 可以查询出来    
5.case when 字段条件, when 字段条件 then end   
6.truncate(字段[数值类型]，位数)
7.round(数值，保留小数位) 
8.limit(5,10)检索6-15的记录
9.unsigned 属性只针对整型 无符号 扩大数据长度    
```

### Mysql的性能指标 QPS TPS

```java
QPS（Query per second） 每秒查询量
    show status like 'queries'
    例如采样10秒内的查询次数，那么先查询一次Queries值（Q1），等待10秒，再查询一次Queries值（Q2）
	QPS = (Q2 - Q1) / 10
TPS
mysql中没有直接的事务计数器，需要通过事务提交数和事务回滚数来计算
TPS = (Com_commit + Com_rollback) / Seconds    
    show stauts like 'com_commit'
    show stauts like 'com_rollkack'
```





### 事物隔离

1.ACID

​	atomic原子性  consistency 一致性  isolation  隔离性   durabilityd 持久性

2.多个事务同时执行的时候 可能会出现 

​	数据丢失			    A 事务修改的数据 B事物回滚 导致 A事物修改的丢失

​	脏读   				     A事务读取 到 B事物中没有提交的数据

​	不可重复读             A事务本流程 读取到的数据 前后不一致 因为 B事物对读取数据的修改

​	幻读

#### 对应的隔离级别

​	读未提交 	一个事务读取到 另一个事务未提交的数据——

​	读提交		一个事务读取到 另一个事务已提交的数据—— [RC]

​	可重复度	一个事务读取到的数据 在 另一个事务未提交的数据（A）  和 已提交数据（A+1） 是一致的—— 如果这个事务读取到的是A ，那么不管另一个事务是否修改A，在这个事务本次过程中一直为A	[RR]

​	串行化  	 一个事务执行 修改逻辑后会被锁住， 先开的事务的提交后，这个事务才会继续执行



mysql的默认隔离级别是 RR 可重复读

Spring事物使用 RC 读已提交 

#### 事务隔离的实现

​	在Mysql中，每条记录在更新的时候都会同时记录一条回滚操作，记录上的最新值，通过回滚操作，可以得到前一个状态的值。同一条记录在系统中可与存在多个版本，这就是数据库的多版本并发控制

#### 避免使用长事务

长事务意味着系统里面有很老的事务视图，大量的会晕记录，占用存储空间

 INNODB_TRX   当前innodb内部正在运行的事务信息，包括只读事务 

#### 事物的启动

autocommit =0 关闭事务自动提交

autocommit =1 开启事务自动提交

### 索引

### Mysql大表调优三部曲

```java
1.慢查询
    SET GLOBAL slow_query_log = 1
2.Explain
    select 具体的索引
    type
    	system
    	const
    	eq_ref
    	ref
    	range
    	index
    	all
    key
    rows
    ref
    
3.Profile    
    查询到 SQL 会执行多少时间, 并看出 CPU/Memory 使用量, 执行过程中 Systemlock, Table lock 花多少时间等等
     show profiles query_id
    打开表 初始化 执行表 sendingdata
    从连接 - 服务 - 引擎 - 存储四层结构完整生命周期的耗时
4.大表超过5000W条
    1.进行分表 
    	垂直分表： 主键+常用列 放在原表中，再讲 主键+一些不常用列 放在另外的表中		
    	水平分表
    		按照时间分表 日志数据等
    		按照区间分表 id自增 1~1000一张表table1 1001~2000一张表table2，2001~3000一张表table3，并且新建一张表 维护 表名+起始值
    		
    		hash分表 id通过 hash算法 
    	水平分表后，对于增删改查 先确定 对应的表
    	水平分表后 对于自增主键ID的解决方案
    		1.新建一张表维护id 新增的时候 先insert一条语句，每次从这张表拿出下一个id
    		2.使用队列服务，如redis、memcacheq等等，将一定量的ID预分配在一个队列里，每次插入操作，先从队列中获取一个ID，若插入失败的话，将该ID再次添加到队列中，同时监控队列数量，当小于阀值时，自动向队列中添加元素
    		3.redis生成
    			成分布式ID，其实和利用Mysql自增ID类似，可以利用Redis中的incr命令来实现原子性的自增与返回
    2.读写分离 主从复制
    
```

### 分表后的全局唯一Id

```java
1.UUID
    无序、无法保证趋势递增（要求3）字符存储、传输、查询慢、不可读
2.雪花算法：
    雪花算法是一个64字节的long类型数字，其中各部分含义如下。
	1，第一部分1个 bit：0，这个是无意义的，它保证了我们的二进制中的首位为0，如果为1则为负数。
	2，第二部分41个 bit：表示的是单位为毫秒的时间戳，来保证我们的id是有序的。
	3，第三部分是10个 bit：前5个bit表示的是机房id，最多可以有32个机房；后5个bit表示的是机器id，最多可以有32台机器；
	4，最后部分是12个 bit：表示的序号，就是某个机房某台机器上这一毫秒内同时生成的 id 的序号，可以保证一毫秒内有4096个id；
    
```

### Mysql乐观锁

```java
银行两操作员同时操作同一账户。
比如A、B操作员同时读取一余额为1000元的账户，A操作员为该账户增加100元，B操作员同时为该账户扣除50元，A先提交，B后提交。最后实际账户余额为1000-50=950元，但本该为1000+100-50=1050。这就是典型的并发问题。

乐观锁机制在一定程度上解决了这个问题。乐观锁，大多是基于数据版本(Version)记录机制实现。何谓数据版本？即为数据增加一个版本标识，在基于数据库表的版本解决方案中，一般是通过为数据库表增加一个 “version” 字段来实现。

读取出数据时，将此版本号一同读出，之后更新时，对此版本号加一。此时，将提交数据的版本数据与数据库表对应记录的当前版本信息进行比对，如果提交的数据版本号大于数据库表当前版本号，则予以更新，否则认为是过期数据。

对于上面修改用户帐户信息的例子而言，假设数据库中帐户信息表中有一个version字段，当前值为1；而当前帐户余额字段(balance)为1000元。假设操作员A先更新完，操作员B后更新。
a、操作员A此时将其读出(version=1)，并从其帐户余额中增加100(1000+100=1100)。
b、在操作员A操作的过程中，操作员B也读入此用户信息(version=1)，并从其帐户余额中扣除50(1000-50=950)。
c、操作员A完成了修改工作，将数据版本号加一(version=2)，连同帐户增加后余额(balance=1100)，提交至数据库更新，此时由于提交数据版本大于数据库记录当前版本，数据被更新，数据库记录version更新为2。
d、操作员B完成了操作，也将版本号加一(version=2)试图向数据库提交数据(balance=950)，但此时比对数据库记录版本时发现，操作员B提交的数据版本号为2，数据库记录当前版本也为2，不满足 “提交版本必须大于记录当前版本才能执行更新 “的乐观锁策略，因此，操作员B的提交被驳回。
这样，就避免了操作员B用基于version=1的旧数据修改的结果覆盖操作员A的操作结果的可能。
```

















