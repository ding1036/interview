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
- [数据库分区分表](#数据库分区分表)
    - [mysql的分区和分表](#mysql的分区和分表)
        - [分区](#分区)
            - [range 分区](#range-分区)
            - [list分区](#list分区)
            - [hash分区](#hash分区)
            - [key分区](#key分区)
            - [分区的优点](#分区的优点)
            - [分区的限制：](#分区的限制)
        - [分表](#分表)
- [索引](#索引)
    - [建立，修改，重建，删除语句](#建立修改重建删除语句)
    - [索引失效](#索引失效)

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

# 数据库分区分表

## mysql的分区和分表

### 分区
1) 横向分区
假如有100W条数据，分成十份，前10W条数据放到第一个分区，第二个10W条数据放到第二个分区，依此类推。
横向分区，并没有改变表的结构。

2) 纵向分区
举例来说明，在设计用户表的时候，开始的时候没有考虑好，而把个人的所有信息都放到了一张表里面去，这样这个表里面就会有比较大的字段，如个人简介,把这样的大字段，分开来

#### range 分区

```sql
create table t_range( 
　　   id int(11), 
　　   money int(11) unsigned not null, 
　　   date datetime 
　　)partition by range(year(date))( 
　　partition p2007 values less than (2008), 
　　partition p2008 values less than (2009), 
　　partition p2009 values less than (2010) 
　　partition p2010 values less than maxvalue  #MAXVALUE 表示最大的可能的整数值
　　)；
```

RANGE分区在如下场合特别有用：
1) 当需要删除一个分区上的“旧的”数据时,只删除分区即可。如果你使用上面最近的那个例子给出的分区方案，你只需简单地使用”ALTER TABLE employees DROP PARTITION p0;”
　　来删除所有在1991年前就已经停止工作的雇员相对应的所有行。对于有大量行的表，这比运行一个如”DELETE FROM employees WHERE YEAR (separated) <= 1990;”
　　这样的一个DELETE查询要有效得多。 
2) 想要使用一个包含有日期或时间值，或包含有从一些其他级数开始增长的值的列。
3) 经常运行直接依赖于用于分割表的列的查询。
　　例如，当执行一个如”SELECT COUNT(*) FROM employees WHERE YEAR(separated) = 2000 GROUP BY store_id;”这样的查询时，
　　MySQL可以很迅速地确定只有分区p2需要扫描，这是因为余下的分区不可能包含有符合该WHERE子句的任何记录

#### list分区
这种模式允许系统通过预定义的列表的值来对数据进行分割。

```sql
create table t_list( 
　　a int(11), 
　　b int(11) 
　　)(partition by list (b) 
　　partition p0 values in (1,3,5,7,9), 
　　partition p1 values in (2,4,6,8,0) 
);
```

LIST分区没有类似如“VALUES LESS THAN MAXVALUE”这样的包含其他值在内的定义。将要匹配的任何值都必须在值列表中找到。

#### hash分区

这中模式允许通过对表的一个或多个列的Hash Key进行计算，最后通过这个Hash码不同数值对应的数据区域进行分区。例如可以建立一个对表主键进行分区的表。

```sql

CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT,
    store_id INT
)
PARTITION BY HASH(store_id)
PARTITIONS 4;
```

#### key分区
 上面Hash模式的一种延伸，这里的Hash Key是MySQL系统产生的。

 ```sql
CREATE TABLE tk (
    col1 INT NOT NULL,
    col2 CHAR(5),
    col3 DATE
)
PARTITION BY LINEAR KEY (col1)
PARTITIONS 3;
 ```

#### 分区的优点

1) 分区可以分在多个磁盘，存储更大一点

2) 根据查找条件，也就是where后面的条件，查找只查找相应的分区不用全部查找了

3) 进行大数据搜索时可以进行并行处理。

4) 跨多个磁盘来分散数据查询，来获得更大的查询吞吐量


#### 分区的限制：

1) 主键或者唯一索引必须包含分区字段，如primary key (id,username),不过innoDB的大组建性能不好。

2) 很多时候，使用分区就不要在使用主键了，否则可能影响性能。

3) 只能通过int类型的字段或者返回int类型的表达式来分区，通常使用year或者to_days等函数（mysql 5.6 对限制开始放开了）。

4) 每个表最多1024个分区，而且多分区会大量消耗内存。

5) 分区的表不支持外键，相关的逻辑约束需要使用程序来实现。

6) 分区后，可能会造成索引失效，需要验证分区可行性。

### 分表

利用mysql cluster ，mysql proxy，mysql replication，drdb等等


参考 1 :[MySQL的分区、分表、集群](https://www.cnblogs.com/myvic/p/7711498.html)

参考 2 :[mysql的分区和分表](https://www.cnblogs.com/phpshen/p/6198375.html)

参考 2 :[数据库的分区分库分表，水平切分与垂直切分](https://blog.csdn.net/weixin_39684625/article/details/79527739)


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

 1．隐式转换导致索引失效.这一点应当引起重视.也是开发中经常会犯的错误.
 由于表的字段tu_mdn定义为``varchar2(20)``,但在查询时把该字段作为``number``类型以where条件传给Oracle,这样会导致索引失效.
 ```sql
 错误的例子：select * from test where tu_mdn=13333333333;
 正确的例子：select * from test where tu_mdn='13333333333';
 ```
 2. 对索引列进行运算导致索引失效,我所指的对索引列进行运算包括(+，-，*，/，! 等)
 ```sql
 错误的例子：select * from test where id-1=9;
 正确的例子：select * from test where id=10;
 ```
 3. 使用Oracle内部函数导致索引失效.对于这样情况应当创建基于函数的索引.
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
 4. 以下使用会使索引失效，应避免使用；
 a. 使用 <> 、not in 、not exist、!=
 b. like "%_" 百分号在前（可采用在建立索引时用reverse(columnName)这种方法处理）
 c. 单独引用复合索引里非第一位置的索引列.应总是使用索引的第一个列，如果索引是建立在多个列上, 只有在它的第一个列被where子句引用时，优化器才会选择使用该索引。
 d. 字符型字段为数字时在where条件里不添加引号.
 e. 当变量采用的是times变量，而表的字段采用的是date变量时.或相反情况。
 5. 不要将空的变量值直接与比较运算符（符号）比较。
 如果变量可能为空，应使用 IS NULL 或 IS NOT NULL 进行比较，或者使用 ISNULL 函数。
 6. 不要在 SQL 代码中使用双引号。
 因为字符常量使用单引号。如果没有必要限定对象名称，可以使用（非 ANSI SQL 标准）括号将名称括起来。
 7. 将索引所在表空间和数据所在表空间分别设于不同的磁盘chunk上，有助于提高索引查询的效率。
 8. Oracle默认使用的基于代价的SQL优化器（CBO）非常依赖于统计信息，一旦统计信息不正常，会导致数据库查询时不使用索引或使用错误的索引。
 一般来说，Oracle的自动任务里面会包含更新统计信息的语句，但如果表数据发生了比较大的变化（超过20%）,可以考虑立即手动更新统计信息，例如：analyze table abc compute statistics，但注意，更新   统计信息比较耗费系统资源，建议在系统空闲时执行。
 9. Oracle在进行一次查询时，一般对一个表只会使用一个索引.
 因此，有时候过多的索引可能导致Oracle使用错误的索引，降低查询效率。例如某表有索引1（Policyno）和索引2（classcode），如果查询条件为policyno = ‘xx’ and classcode = ‘xx’，则系统有可能会使用索   引2，相较于使用索引1，查询效率明显降低。
 10. 优先且尽可能使用分区索引。


