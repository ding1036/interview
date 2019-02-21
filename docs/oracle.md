<a id = "jump">[首页](/README.md)</a>

<!-- TOC -->

- [oracle存储函数和函数的区别](#oracle存储函数和函数的区别)
- [oracle触发器类型和出发事件](#oracle触发器类型和出发事件)
- [触发器启用或者禁用](#触发器启用或者禁用)
- [视图中使用DML的规定](#视图中使用dml的规定)
- [分页查询语句](#分页查询语句)

<!-- /TOC -->

# oracle存储函数和函数的区别

定义关键字不同，存储函数是``procedure`` , 函数是 ``function``

存储过程不需要声明返回类型，可以返回多个值
函数需要声明返回类型

[toTop](#jump)

# oracle触发器类型和出发事件
类型 ：before , after ,instead of
事件 ：insert ,delete ,update

[toTop](#jump)

# 触发器启用或者禁用

启用：Alter trigger trigger_name enable
禁用：Alter trigger trigger_name disable
禁用所有触发器: Alter table table_name disable all triggers
启用所有触发器: Alter table table_name enable all triggers
重新编译触发器：Alter trigger trigger_name compile

[toTop](#jump)

# 视图中使用DML的规定
视图中使用DML的规定

1) 可以在简单视图中执行 DML操作

2) 当视图定义中包含以下元素之一时不能使用delete：

```
i. 组函数

ii. GROUP BY子句

iii. DISTINCT关键字

iv. ROWNUM 伪列 DUAL伪表

```
3) 当视图定义中包含以下元素之一时不能使用update：

```
i. 组函数

ii. GROUP BY子句

iii. DISTINCT关键字

iv. ROWNUM 伪列

v. 列的定义为表达式
```

4) 当视图定义中包含以下元素之一时不能使用insert:

```
i.组函数

ii.GROUP BY子句

iii.DISTINCT关键字

iv.ROWNUM 伪列

v.列的定义为表达式

vi.中非空的列在视图定义中未包括
```

  WITH CHECK OPTION 子句

1) 使用 WITH CHECKOPTION 子句确保DML只能在特定的范围内执行
2) 任何违反WITH CHECKOPTION 约束的请求都会失败

[toTop](#jump)

# 分页查询语句

```sql
SELECT * FROM  
(  
SELECT A.*, ROWNUM RN  
FROM (SELECT * FROM TABLE_NAME) A  
WHERE ROWNUM <= 40  
)  
WHERE RN >= 21  
```

```sql
SELECT * FROM  
(  
SELECT A.*, ROWNUM RN  
FROM (SELECT * FROM TABLE_NAME) A  
)  
WHERE RN BETWEEN 21 AND 40  
```
绝大多数的情况下，第一个查询的效率比第二个高得多

```sql

select a1.* from 
  (
  select student.*,rownum rn 
  from student 
  where rownum <=5
  ) a1 
where rn >=3;
```
[toTop](#jump)