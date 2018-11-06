<a id = "jump">[首页](/README.md)</a>




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