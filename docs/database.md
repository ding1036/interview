<a id = "jump">[首页](/README.md)</a>

<!-- TOC -->

- [数据库优化](#数据库优化)
    - [索引优化](#索引优化)
        - [独立的列](#独立的列)
        - [多列索引](#多列索引)
        - [索引列的顺序](#索引列的顺序)
    - [优化数据访问](#优化数据访问)
        - [减少请求的数据量](#减少请求的数据量)
        - [减少服务器端扫描的行数](#减少服务器端扫描的行数)
    - [重构查询方式](#重构查询方式)
        - [切分大查询](#切分大查询)
        - [分解大连接查询](#分解大连接查询)
- [索引](#索引)
    - [建立，修改，重建，删除语句](#建立修改重建删除语句)
    - [索引失效](#索引失效)
    - [为什么不在所有字段上加索引](#为什么不在所有字段上加索引)
- [四种隔离级别](#四种隔离级别)
- [隐式字符编码转换](#隐式字符编码转换)

<!-- /TOC -->

[mysql相关](mysql.md)
[oracle相关](oracle.md)



# 数据库优化

## 索引优化

### 独立的列
在进行查询时，索引列不能是表达式的一部分，也不能是函数的参数，否则无法使用索引。

### 多列索引
在需要使用多个列作为条件进行查询时，使用多列索引比使用多个单列索引性能更好。

### 索引列的顺序
让选择性最强的索引列放在前面。

索引的选择性是指：不重复的索引值和记录总数的比值。最大值为 1，此时每个记录都有唯一的索引与其对应。选择性越高，查询效率也越高。

例如下面显示的结果中 customer_id 的选择性比 staff_id 更高，因此最好把 customer_id 列放在多列索引的前面。

```sql
SELECT COUNT(DISTINCT staff_id)/COUNT(*) AS staff_id_selectivity,
COUNT(DISTINCT customer_id)/COUNT(*) AS customer_id_selectivity,
COUNT(*)
FROM payment;
```

```
staff_id_selectivity: 0.0001
customer_id_selectivity: 0.0373
COUNT(*): 16049

```

## 优化数据访问

### 减少请求的数据量
* 只返回必要的列：最好不要使用 SELECT * 语句。
* 只返回必要的行：使用 LIMIT 语句来限制返回的数据。
* 缓存重复查询的数据：使用缓存可以避免在数据库中进行查询，特别在要查询的数*据经常被重复查询时，缓存带来的查询性能提升将会是非常明显的。

### 减少服务器端扫描的行数
最有效的方式是使用索引来覆盖查询。

## 重构查询方式

### 切分大查询
一个大查询如果一次性执行的话，可能一次锁住很多数据、占满整个事务日志、耗尽系统资源、阻塞很多小的但重要的查询。
例子
```sql
DELEFT FROM messages WHERE create < DATE_SUB(NOW(), INTERVAL 3 MONTH);
```
改为
```sql
rows_affected = 0
do {
    rows_affected = do_query(
    "DELETE FROM messages WHERE create  < DATE_SUB(NOW(), INTERVAL 3 MONTH) LIMIT 10000")
} while rows_affected > 0
```

### 分解大连接查询
将一个大连接查询分解成对每一个表进行一次单表查询，然后在应用程序中进行关联，这样做的好处有：

* 让缓存更高效。对于连接查询，如果其中一个表发生变化，那么整个查询缓存就无法使用。而分解后的多个查询，即使其中一个表发生变化，对其它表的查询缓存依然可以使用。
* 分解成多个单表查询，这些单表查询的缓存结果更可能被其它查询使用到，从而减少冗余记录的查询。
减少锁竞争；
* 在应用层进行连接，可以更容易对数据库进行拆分，从而更容易做到高性能和可伸缩。
* 查询本身效率也可能会有所提升。例如下面的例子中，使用 IN() 代替连接查询，可以让 MySQL 按照 ID 顺序进行查询，这可能比随机的连接要更高效。
例子

```sql
SELECT * FROM tab
JOIN tag_post ON tag_post.tag_id=tag.id
JOIN post ON tag_post.post_id=post.id
WHERE tag.tag='mysql';
```
改为
```sql
SELECT * FROM tag WHERE tag='mysql';
SELECT * FROM tag_post WHERE tag_id=1234;
SELECT * FROM post WHERE post.id IN (123,456,567,9098,8904);
```

[toTop](#jump)

# 索引

## 建立，修改，重建，删除语句

* MYSQL

创建
```sql
alter table table_name add index index_name (column_list) ;
alter table table_name add unique (column_list) ;
alter table table_name add primary key (column_list) ;
create index index_name on table_name 列名;
create unique index index_name on table_name 列名;
```
删除
```sql
drop index index_name on table_name ;
alter table table_name drop index index_name ;
alter table table_name drop primary key ;
```

修改
```sql
--先删除

ALTER TABLE table_name DROP INDEX index_name;

--再以修改后的内容创建同名索引

CREATE INDEX index_name ON table_name;
```

## 索引失效
 1) 隐式转换导致索引失效.这一点应当引起重视.也是开发中经常会犯的错误.
 由于表的字段tu_mdn定义为``varchar2(20)``,但在查询时把该字段作为``number``类型以where条件传给Oracle,这样会导致索引失效.
 ```sql
 错误的例子：select * from test where tu_mdn=13333333333;
 正确的例子：select * from test where tu_mdn='13333333333';
 ```
 2) 对索引列进行运算导致索引失效,我所指的对索引列进行运算包括(+，-，*，/，! 等)
 ```sql
 错误的例子：select * from test where id-1=9;
 正确的例子：select * from test where id=10;
 ```
 3) 使用Oracle内部函数导致索引失效.对于这样情况应当创建基于函数的索引.
 错误的例子：
 ```sql
 select * from test where round(id)=10;
 说明，此时id的索引已经不起作用了
 正确的例子：首先建立函数索引，
 create index test_id_fbi_idx on test(round(id));
 然后 
 select * from test where round(id)=10; 
 ```
 这时函数索引起作用了

 4) 以下使用会使索引失效，应避免使用；
 a. 使用 **<> 、not in 、not exist、!=**
 b. **like "%_" 百分号在前（可采用在建立索引时用reverse(columnName)这种方法处理）**
 c. 单独引用复合索引里非第一位置的索引列.应总是使用索引的第一个列，如果索引是建立在多个列上, 只有在它的第一个列被where子句引用时，优化器才会选择使用该索引。
 d. 字符型字段为数字时在where条件里不添加引号.
 e. 当变量采用的是times变量，而表的字段采用的是date变量时.或相反情况。
 5) 不要将空的变量值直接与比较运算符（符号）比较。
 如果变量可能为空，应使用 IS NULL 或 IS NOT NULL 进行比较，或者使用 ISNULL 函数。
 6) 不要在 SQL 代码中使用双引号。
 因为字符常量使用单引号。如果没有必要限定对象名称，可以使用（非 ANSI SQL 标准）括号将名称括起来。
 7) 将索引所在表空间和数据所在表空间分别设于不同的磁盘chunk上，有助于提高索引查询的效率。
 8) Oracle默认使用的基于代价的SQL优化器（CBO）非常依赖于统计信息，一旦统计信息不正常，会导致数   据库查询时不使用索引或使用错误的索引。
 一般来说，Oracle的自动任务里面会包含更新统计信息的语句，但如果表数据发生了比较大的变化（超过20%）,可以考虑立即手动更新统计信息，例如：analyze table abc compute statistics，但注意，更新   统计信息比较耗费系统资源，建议在系统空闲时执行。
 9) Oracle在进行一次查询时，一般对一个表只会使用一个索引.
 因此，有时候过多的索引可能导致Oracle使用错误的索引，降低查询效率。例如某表有索引1（Policyno）和索引2（classcode），如果查询条件为policyno = ‘xx’ and classcode = ‘xx’，则系统有可能会使用索   引2，相较于使用索引1，查询效率明显降低。
 10) 优先且尽可能使用分区索引。
 
 [toTop](#jump)


 
## 为什么不在所有字段上加索引
 1)  索引会占用存储空间，索引越多，使用的存储空间越多

 2)  插入数据，存储索引也会消耗时间，索引越多，插入数据的速度越慢

 [toTop](#jump)

# 四种隔离级别
**读取未提交(Read uncommitted)**：处于此模式下可能会出现脏读、幻象读、不可重复读
**读取已提交(Read committed)**：处于此模式下可能会出现幻象读、不可重复读（oracle默认隔离级别）
**可重复读(Repeatable read)**：处于此模式下可能会出现幻象读(InnoDB默认隔离级别)
**串行(Serialize)**：不会出现幻象读
 

- **脏读**：其它的事务（执行单个 select 语句也算一个事务）可以读取到某个事务更新（包括插入和删除）了但未提交的数据。脏读是应用中应该避免的，因为读取的是不可靠的数据（我觉得把这个叫做幻象行更形象，实际却不是）。一般数据库不会设定为这个模式，但有时候也会用到。脏读的好处是读取时不会对表或记录加锁，可以绕开写队列的排队，避免了等待。如在一个更新特别频繁的表中要选择表中所有的数据，就可以显示指定隔离级别：
``select .... at isolation 0``
 
- **不可重复读**：这是描述在同一个事务中两条一模一样的 select 语句的执行结果的比较。如果**前后执行的结果一样，则是可重复读**；**如果前后的结果可以不一样，则是不可重复读**。
不可重复读的模式下首先不会出现脏读，即读取的都是已提交的数据。在一个事务中，读取操作是不会加排他锁的，当下一条一模一样的 select 语句的执行时，命中的数据集可能已经被其它事务修改了，这时候，还能读到相同的内容吗？
因此，要达到可重复读的效果，数据库需要做更多的事情，比如，对读取的数据行加共享锁，并保持到事务结束，以禁止其它事务修改它。这样会降低数据库的性能。而隔离级别的串行则比可重复读更严格。一般数据库的的隔离级别只设置到读取已提交。这是兼顾了可靠性和性能的结果。
上面还只提到了对命中的数据行加锁，以防止其它事务修改它。但没有提到，
如果其它事务增加了符合条件的数据行怎么办？有些数据库对这种情况新定义了两个级别：读取稳定性和游标稳定性。前者不限制新增符合条件的数据行，而后者则阻止新增这样的数据行。
 
- **幻象读**：是指两次执行同一条 select 语句会出现不同的结果，第二次读会增加一数据行，并没有说这两次执行是在同一个事务中。一般情况下，幻象读应该正是我们所需要的。但有时候却不是，如果打开的游标，在对游标进行操作时，并不希望新增的记录加到游标命中的数据集中来。隔离级别为 游标稳定性 的，可以阻止幻象读。

 [toTop](#jump)


# 隐式字符编码转换

如果两个表的字符集不一样，一个是``utf8mb4``，一个是``utf8``，因为``utf8mb4``是``utf8``的超集，所以一旦两个字符比较，就会转换为``utf8mb4``再比较。转换的过程相当于加了``CONVERT(id USING utf8mb4)``函数，那又回到上面的问题了，用到函数就用不上索引了。还有大家一会可能会遇到mysql突然卡顿的情况，那可能是``MySQLflush``了。

 [toTop](#jump)


 


