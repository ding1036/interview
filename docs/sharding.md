<a id = "jump">[首页](/README.md)</a>


[sharding jdbc相关](sharding_jdbc.md)

[mycat相关](mycat.md)

<!-- TOC -->

- [mysql的分区和分表](#mysql的分区和分表)
    - [分区](#分区)
        - [range 分区](#range-分区)
        - [list分区](#list分区)
        - [hash分区](#hash分区)
        - [key分区](#key分区)
            - [分区的优点](#分区的优点)
        - [分区的限制：](#分区的限制)
    - [分表](#分表)

<!-- /TOC -->


# mysql的分区和分表

## 分区
1) 横向分区
假如有100W条数据，分成十份，前10W条数据放到第一个分区，第二个10W条数据放到第二个分区，依此类推。
横向分区，并没有改变表的结构。

2) 纵向分区
举例来说明，在设计用户表的时候，开始的时候没有考虑好，而把个人的所有信息都放到了一张表里面去，这样这个表里面就会有比较大的字段，如个人简介,把这样的大字段，分开来

### range 分区

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

### list分区
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

### hash分区

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

### key分区
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


### 分区的限制：

1) 主键或者唯一索引必须包含分区字段，如primary key (id,username),不过innoDB的大组建性能不好。

2) 很多时候，使用分区就不要在使用主键了，否则可能影响性能。

3) 只能通过int类型的字段或者返回int类型的表达式来分区，通常使用year或者to_days等函数（mysql 5.6 对限制开始放开了）。

4) 每个表最多1024个分区，而且多分区会大量消耗内存。

5) 分区的表不支持外键，相关的逻辑约束需要使用程序来实现。

6) 分区后，可能会造成索引失效，需要验证分区可行性。

## 分表

利用mysql cluster ，mysql proxy，mysql replication，drdb等等


参考 1 :[MySQL的分区、分表、集群](https://www.cnblogs.com/myvic/p/7711498.html)

参考 2 :[mysql的分区和分表](https://www.cnblogs.com/phpshen/p/6198375.html)

参考 2 :[数据库的分区分库分表，水平切分与垂直切分](https://blog.csdn.net/weixin_39684625/article/details/79527739)


[toTop](#jump)