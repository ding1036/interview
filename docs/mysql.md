<a id = "jump">[首页](/README.md)</a>

<!-- TOC -->

- [MyISAM和innerDB区别](#myisam和innerdb区别)
- [mysql千万级大表在线加索引（待验证是否可行）](#mysql千万级大表在线加索引待验证是否可行)
- [密集索引和稀疏索引的区别](#密集索引和稀疏索引的区别)
- [定位并优化慢sql](#定位并优化慢sql)
- [联合索引最左匹配原则成因](#联合索引最左匹配原则成因)
- [表级锁，行级锁](#表级锁行级锁)
- [InnoDB在可重复读下如何实现避免幻读](#innodb在可重复读下如何实现避免幻读)
- [为什么MySQL数据库要用B+树存储索引](#为什么mysql数据库要用b树存储索引)
- [MySQL 中 varchar 与 char 的区别？varchar(50) 中的 50 代表的涵义？](#mysql-中-varchar-与-char-的区别varchar50-中的-50-代表的涵义)
- [一张表，里面有 ID 自增主键，当 insert 了 17 条记录之后，删除了第 15,16,17 条记录，再把 MySQL 重启，再 insert 一条记录，这条记录的 ID 是 18 还是 15？](#一张表里面有-id-自增主键当-insert-了-17-条记录之后删除了第-151617-条记录再把-mysql-重启再-insert-一条记录这条记录的-id-是-18-还是-15)
- [为什么 SELECT COUNT(*) FROM table 在 InnoDB 比 MyISAM 慢？](#为什么-select-count-from-table-在-innodb-比-myisam-慢)
- [索引](#索引)
    - [索引的类型？(针对mysql)](#索引的类型针对mysql)
    - [MySQL 索引的“创建”原则？](#mysql-索引的创建原则)
    - [MySQL 索引的“使用”注意事项？](#mysql-索引的使用注意事项)
    - [MySQL 索引的原理？](#mysql-索引的原理)

<!-- /TOC -->

# MyISAM和innerDB区别
* 事务：InnoDB 是事务型的，可以使用 Commit 和 Rollback 语句。

* 并发：MyISAM 只支持表级锁，而 InnoDB 还支持行级锁。

* 外键：InnoDB 支持外键。

* 备份：InnoDB 支持在线热备份。

* 崩溃恢复：MyISAM 崩溃后发生损坏的概率比 InnoDB 高很多，而且恢复的速度也更慢。

* 其它特性：MyISAM 支持压缩表和空间数据索引。

![](/img/innodb_myisam.PNG)

[toTop](#jump)

# mysql千万级大表在线加索引（待验证是否可行）

```sql
create table tmp like paper_author;

ALTER TABLE tmp ADD INDEX ( `PaperID` );

insert into tmp(ooo，...)  select  ooo,... from paper_author;

RENAME TABLE paper_author TO tmp2, tmp to paper_author;

drop table tmp2;
```

[toTop](#jump)

# 密集索引和稀疏索引的区别
* 区别
**秘籍索引**文件中的每个搜索码值都对应一个索引值
**稀疏索引**文件只为索引码的某些值建立索引项

密集索引的定义：叶子节点保存的不只是键值，还保存了位于同一行记录里的其他列的信息，由于密集索引决定了表的物理排列顺序，一个表只有一个物理排列顺序，所以**一个表只能创建一个密集索引**

稀疏索引：叶子节点仅保存了键位信息以及该行数据的地址，有的稀疏索引只保存了键位信息机器主键

MYISAM存储引擎，不管是主键索引，唯一键索引还是普通索引都是稀疏索引

innodb存储引擎：有且只有一个密集索引。密集索引的选取规则如下：
```
    若主键被定义，则主键作为密集索引
    如果没有主键被定义，该表的第一个唯一非空索引则作为密集索引
    若不满足以上条件，innodb内部会生成一个隐藏主键（密集索引）
    非主键索引存储相关键位和其对应的主键值，包含两次查找
```

参考1 ：[密集索引和稀疏索引的区别](https://blog.csdn.net/fansenjun/article/details/85647734)

[toTop](#jump)

# 定位并优化慢sql
1、默认情况下，MySQL认为10秒才是一个慢查询

修改慢查询的时间（1为1秒）
```sql
mysql> set long_query_time=1;
```
显示慢查询的时间值的命令
```sql
mysql> show variables like 'long_query_time';
```
统计慢查询次数（一条sql语句执行所需时间超过慢查询设置的时间，就统计一次）
```sql
mysql> show status like 'slow_queries';   
```



2、定位慢查询的sql语句

查询是否开启了慢查询日志
```sql
mysql> show variables like '%slow%';
```

如果slow_query_log 为 off，则
```
mysql> set global slow_query_log=1;
```

* 慢查询主要参数
long_query_cache
slow_query_log
slow_query_log_file

3.explain分析慢日志

* explain关键字段
```

id:select查询的序列号

select_type:select查询的类型，主要是区别普通查询和联合查询、子查询之类的复杂查询

table: 输出的行所引用的表

type:

联合查询所使用的类型，type显示的是访问类型，是较为重要的一个指标，结果值从好到坏依次是：

system > const > eq_ref > ref >fulltext > ref_or_null > index_merge > unique_subquery >index_subquery > range > index > ALL

一般来说，得保证查询至少达到range级别，最好能达到ref。

         type=const表示通过索引一次就找到了；

         type=all,表示为全表扫描；

         type=ref,因为这时认为是多个匹配行，在联合查询中，一般为REF；

key:

显示MySQL实际决定使用的键。如果没有索引被选择，键是NULL。

         key=primary的话，表示使用了主键；

         key=null表示没用到索引。

possible_keys:

指出MySQL能使用哪个索引在该表中找到行。如果是空的，没有相关的索引。这时要提高性能，可通过检验WHERE子句，看是否引用某些字段，或者检查字段不是适合索引。

key_len:

显示MySQL决定使用的键长度。如果键是NULL，长度就是NULL。文档提示特别注意这个值可以得出一个多重主键里mysql实际使用了哪一部分。

ref:

显示哪个字段或常数与key一起被使用。

rows：

这个数表示mysql要遍历多少数据才能找到，在innodb上是不准确的。

Extra:

如果是Only index，这意味着信息只用索引树中的信息检索出的，这比扫描整个表要快。

如果是where used，就是使用上了where限制。

如果是impossible where 表示用不着where，一般就是没查出来啥。
```

使用``force index(XXX)``可以强制使用索引XXX

参考1 :[MySQL优化 之 定位慢查询的sql语句](https://blog.csdn.net/mlx212/article/details/17845241)

参考2 :[mysql之explain关键字](https://blog.csdn.net/hll814/article/details/50765232)

[toTop](#jump)

# 联合索引最左匹配原则成因

1. 最左前缀匹配原则，非常重要的原则，mysql会一直向右匹配直到遇到范围查询(``>、<、between、like``)就停止匹配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。

2. ``=和in可以乱序``，比如``a = 1 and b = 2 and c = 3 ``建立(a,b,c)索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式


比如当 (张三, 20, F) 这样的数据来检索的时候，B+Tree 会优先比较 name 来确定下一步的所搜方向，如果 name 相同再依次比较 age 和 sex ，最后得到检索的数据。
但当 (20, F) 这样的没有 name 的数据来的时候，B+Tree 就不知道下一步该查哪个节点，因为建立搜索树的时候 name 就是第一个比较因子，必须要先根据 name 来搜索才能知道下一步去哪里查询。
比如当 (张三, F) 这样的数据来检索时，B+Tree 可以用 name 来指定搜索方向，但下一个字段 age 的缺失，所以只能把名字等于张三的数据都找到，然后再匹配性别是 F 的数据了。


参考1：[联合索引的最左前缀匹配原则](https://www.jianshu.com/p/b7911e0394b0)

[toTop](#jump)


# 表级锁，行级锁
不走索引 表级锁
InnoDB 支持行级锁，支持表级锁
MyIsam 支持表级锁，不支持行级锁


[toTop](#jump)

# InnoDB在可重复读下如何实现避免幻读

在快照读读情况下，mysql通过mvcc来避免幻读。
在当前读读情况下，mysql通过next-key来避免幻读

1) 什么是mvcc
mvcc全称是multi version concurrent control（多版本并发控制）。mysql把每个操作都定义成一个事务，每开启一个事务，系统的事务版本号自动递增。每行记录都有两个隐藏列：创建版本号和删除版本号
select：事务每次只能读到创建版本号小于等于此次系统版本号的记录，同时行的删除版本号不存在或者大于当前事务的版本号。
update：插入一条新记录，并把当前系统版本号作为行记录的版本号，同时保存当前系统版本号到原有的行作为删除版本号。
delete：把当前系统版本号作为行记录的删除版本号
insert：把当前系统版本号作为行记录的版本号

2) 什么是next-key锁
可以简单的理解为行锁+GAP锁

3) 什么是快照读和当前读
快照读：简单的select操作，属于快照读，不加锁。(当然，也有例外，下面会分析)
```sql
select * from table where ?;
```
当前读：特殊的读操作，插入/更新/删除操作，属于当前读，需要加锁。
```sql
select * from table where ? lock in share mode;
select * from table where ? for update;
insert into table values (…);
update table set ? where ?;
delete from table where ?;

```

[toTop](#jump)

# 为什么MySQL数据库要用B+树存储索引

文件系统和数据库的索引都是存在硬盘上的，并且如果数据量大的话，不一定能一次性加载到内存中。
数据库中 Select 数据，不一定只选一条，很多时候会选多条，比如按照 ID 排序后选 10 条。B 树需要做局部的中序遍历，可能要跨层访问。
而 B+ 树由于所有数据都在叶子结点，不用跨层，同时由于有链表结构，只需要找到首尾，通过链表就能把所有数据取出来了。

如果只选一个数据，那确实是 Hash 更快。但是数据库中经常会选择多条，这时候由于 B+ 树索引有序，并且又有链表相连，它的查询效率比 Hash 就快很多了。而且数据库中的索引一般是在磁盘上，数据量大的情况可能无法一次装入内存，B+ 树的设计可以允许数据分批加载，同时树的高度较低，提高查找效率。

[toTop](#jump)

# MySQL 中 varchar 与 char 的区别？varchar(50) 中的 50 代表的涵义？

1) ``varchar`` 与 ``char`` 的区别，``char`` 是一种固定长度的类型，``varchar`` 则是一种可变长度的类型。
2) ``varchar(50)`` 中 50 的涵义最多存放 50 个字符。``varchar(50)`` 和 ``(200)`` 存储 hello 所占空间一样，但后者在排序时会消耗更多内存，因为 ``ORDER BY col`` 采用`` fixed_length`` 计算 col 长度(memory引擎也一样)。

[toTop](#jump)

# 一张表，里面有 ID 自增主键，当 insert 了 17 条记录之后，删除了第 15,16,17 条记录，再把 MySQL 重启，再 insert 一条记录，这条记录的 ID 是 18 还是 15？

一般情况下，我们创建的表的类型是`` InnoDB ``，如果新增一条记录（不重启 MySQL 的情况下），这条记录的 ID 是``18`` ；但是如果``重启 MySQL`` 的话，这条记录的 ID 是 ``15`` 。因为 InnoDB 表只把自增主键的最大 ID 记录到内存中，所以重启数据库或者对表 OPTIMIZE 操作，都会使最大 ID 丢失。
但是，如果我们使用表的类型是 ``MyISAM`` ，那么这条记录的 ID 就是 ``18`` 。因为 MyISAM 表会把自增主键的最大 ID 记录到数据文件里面，重启 MYSQL 后，自增主键的最大 ID 也不会丢失。

**生产数据，不建议进行物理删除记录。**

[toTop](#jump)

# 为什么 SELECT COUNT(*) FROM table 在 InnoDB 比 MyISAM 慢？

对于 ``SELECT COUNT(*) FROM table`` 语句，在没有 WHERE 条件的情况下，InnoDB 比 MyISAM ``可能会慢很多，尤其在大表的情况下。因为，InnoDB`` 是去实时统计结果，会``全表扫描``；而 ``MyISAM`` 内部维持了一个``计数器``，预存了结果，所以直接返回即可。

[toTop](#jump)

# 索引
## 索引的类型？(针对mysql)

索引，都是实现在存储引擎层的。主要有六种类型：

1) 普通索引：最基本的索引，没有任何约束。

2) 唯一索引：与普通索引类似，但具有唯一性约束。

3) 主键索引：特殊的唯一索引，不允许有空值。

4) 复合索引：将多个列组合在一起创建索引，可以覆盖多个列。

5) 外键索引：只有InnoDB类型的表才可以使用外键索引，保证数据的一致性、完整性和实现级联操作。

6) 全文索引：MySQL 自带的全文索引只能用于 InnoDB、MyISAM ，并且只能对英文进行全文检索，一般使用全文索引引擎。

[toTop](#jump)

## MySQL 索引的“创建”原则？

1) 最适合索引的列是出现在 WHERE 子句中的列，或连接子句中的列，而不是出现在 SELECT 关键字后的列。

2) 索引列的基数越大，索引效果越好。

3) 根据情况创建复合索引，复合索引可以提高查询效率。

4) 避免创建过多的索引，索引会额外占用磁盘空间，降低写操作效率。

5) 主键尽可能选择较短的数据类型，可以有效减少索引的磁盘占用提高查询效率。

6) 对字符串进行索引，应该定制一个前缀长度，可以节省大量的索引空间。

[toTop](#jump)

## MySQL 索引的“使用”注意事项？

1) 应尽量避免在 ``WHERE`` 子句中使用 ``!= ``或`` <>`` 操作符，否则将引擎放弃使用索引而进行全表扫描。优化器将无法通过索引来确定将要命中的行数,因此需要搜索该表的所有行。

    **注意，``column IS NULL`` 也是不可以使用索引的。**

2) 应尽量避免在 ``WHERE`` 子句中使用 ``OR`` 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描，如：``SELECT id FROM t WHERE num = 10 OR num = 20`` 。

3) 应尽量避免在 ``WHERE`` 子句中对字段进行``表达式操作``，这将导致引擎放弃使用索引而进行全表扫描。

4) 应尽量避免在 ``WHERE`` 子句中对字段进行``函数操作``，这将导致引擎放弃使用索引而进行全表扫描。

5) 不要在 ``WHERE`` 子句中的`` = 左边进行函数``、``算术运算``或``其他表达式运算``，否则系统将可能无法正确使用索引。

6) 复合索引遵循前缀原则。

7) 如果 MySQL 评估使用索引比全表扫描更慢，会放弃使用索引。如果此时想要索引，可以在语句中添加强制索引。(**使用force index**)

例如：
```sql
SELECT  * FROM XXX_log force index(ctime) WHERE (`ctime` BETWEEN '2017-09-11 09:34:13'  AND '2017-10-11 09:34:13')
and     id > 27851
AND column1 = 'xxx'
AND (column2 = 'null' OR LENGTH(column2) > 91)
ORDER BY  id DESC LIMIT 0, 30
```

8) 列类型是``字符串类型``，查询时``一定要给值加引号``，否则索引失效。

9) ``LIKE`` 查询，``% 不能在前``，因为无法使用索引。如果需要模糊匹配，可以使用全文索引。

[toTop](#jump)

## MySQL 索引的原理？

[MySQL索引背后的数据结构及算法原理](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)

[MySQL索引原理](https://blog.csdn.net/u013235478/article/details/50625677)

[深入理解MySQL索引原理和实现——为什么索引可以加速查询？](https://blog.csdn.net/tongdanping/article/details/79878302)

[toTop](#jump)