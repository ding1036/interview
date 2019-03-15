<a id = "jump">[首页](/README.md)</a>

<!-- TOC -->

- [mybatis缓存机制](#mybatis缓存机制)
    - [一级缓存](#一级缓存)
    - [二级缓存](#二级缓存)
- [如何获取自动生成的(主)键值](#如何获取自动生成的主键值)

<!-- /TOC -->


# mybatis缓存机制

mybatis的缓存分为两级：一级缓存、二级缓存

* 一级缓存是**SqlSession级别**的缓存，缓存的数据只在SqlSession内有效

* 二级缓存是**mapper级别**的缓存，同一个namespace公用这一个缓存，所以对SqlSession是共享的

## 一级缓存
MyBatis **默认开启了一级缓存**。同一个SqlSession ，多次调用同一个Mapper和同一个方法的同一个参数，只会进行一次数据库查询，然后把数据缓存到缓冲中，以后直接先从缓存中取出数据，不会直接去查数据库。

## 二级缓存
当我们的配置文件配置了cacheEnabled=true时，就会开启二级缓存。不同的sqlsession使用同一个mapper查询是，查询到的数据可能是另一个sqlsession做相同操作留下的缓存。


如果你配置了二级缓存，那么查询数据的顺序应该为：二级缓存→一级缓存→数据库

[toTop](#jump)

# 如何获取自动生成的(主)键值

MySQL 有两种方式，但是自增主键
方式一较为常用

```java
// 方式一，使用 useGeneratedKeys + keyProperty 属性
<insert id="insert" parameterType="Person" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO person(name, pswd)
    VALUE (#{name}, #{pswd})
</insert>
    
// 方式二，使用 `<selectKey />` 标签
<insert id="insert" parameterType="Person" useGeneratedKeys="true" keyProperty="id">
    <selectKey keyProperty="id" resultType="long" order="AFTER">
        SELECT LAST_INSERT_ID()
    </selectKey>
        
    INSERT INTO person(name, pswd)
    VALUE (#{name}, #{pswd})
</insert>
```
Oracle 有两种方式
序列和触发器。基于序列，根据 ``<selectKey /> ``执行的时机，也有两种方式

```java
// 这个是创建表的自增序列
CREATE SEQUENCE student_sequence
INCREMENT BY 1
NOMAXVALUE
NOCYCLE
CACHE 10;

// 方式一，使用 `<selectKey />` 标签 + BEFORE
<insert id="add" parameterType="Student">
　　<selectKey keyProperty="student_id" resultType="int" order="BEFORE">
      select student_sequence.nextval FROM dual
    </selectKey>
    
     INSERT INTO student(student_id, student_name, student_age)
     VALUES (#{student_id},#{student_name},#{student_age})
</insert>

// 方式二，使用 `<selectKey />` 标签 + AFTER
<insert id="save" parameterType="com.threeti.to.ZoneTO" >
    <selectKey resultType="java.lang.Long" keyProperty="id" order="AFTER" >
      SELECT SEQ_ZONE.CURRVAL AS id FROM dual
    </selectKey>
    
    INSERT INTO TBL_ZONE (ID, NAME ) 
    VALUES (SEQ_ZONE.NEXTVAL, #{name,jdbcType=VARCHAR})
</insert>
```