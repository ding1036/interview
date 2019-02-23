<a id = "jump">[首页](/README.md)</a>

<!-- TOC -->

- [redis分布式锁](#redis分布式锁)
- [redis的主从切换的两种方式](#redis的主从切换的两种方式)
- [Redis关于缓存雪崩和缓存穿透等问题](#redis关于缓存雪崩和缓存穿透等问题)
    - [缓存雪崩](#缓存雪崩)
    - [缓存穿透](#缓存穿透)
    - [缓存预热](#缓存预热)
    - [缓存更新](#缓存更新)
- [使用Spring Session redis进行Session共享](#使用spring-session-redis进行session共享)
- [Redis两种持久化方式](#redis两种持久化方式)
    - [RDB](#rdb)
    - [AOF](#aof)
    - [二者优缺点](#二者优缺点)
- [redis为什么要设计成单线程](#redis为什么要设计成单线程)
- [Redis实现Session共享](#redis实现session共享)
- [Redis 的过期策略和内存淘汰机制](#redis-的过期策略和内存淘汰机制)
- [Redis 和数据库双写一致性问题](#redis-和数据库双写一致性问题)
- [如何解决 Redis 的并发竞争 Key 问题](#如何解决-redis-的并发竞争-key-问题)
- [Redis基本类型及应用](#redis基本类型及应用)
    - [String](#string)
    - [List](#list)
    - [Hash](#hash)
    - [Set](#set)
    - [ZSet](#zset)

<!-- /TOC -->

# redis分布式锁

参考 : [Redis 分布式锁的正确实现方式](http://www.importnew.com/27477.html)

参考2 : [基于Redis的分布式锁到底安全吗](http://zhangtielei.com/posts/blog-redlock-reasoning.html)

[toTop](#jump)

# redis的主从切换的两种方式

参考 : [redis的主从切换的两种方式](https://blog.csdn.net/u013516966/article/details/50633925)

参考2 : [Redis 复制功能详解](https://blog.csdn.net/men_wen/article/details/72590550)

[toTop](#jump)

# Redis关于缓存雪崩和缓存穿透等问题

## 缓存雪崩

缓存雪崩是由于原有缓存失效(过期)，新缓存未到期间。所有请求都去查询数据库，而对数据库CPU和内存造成巨大压力，严重的会造成数据库宕机。从而形成一系列连锁反应，造成整个系统崩溃。
1) 碰到这种情况，一般并发量不是特别多的时候，使用最多的解决方案是加锁排队。

```java
    public object GetProductListNew()  
            {  
                const int cacheTime = 30;  
                const string cacheKey = "product_list";  
                const string lockKey = cacheKey;  
                  
                var cacheValue = CacheHelper.Get(cacheKey);  
                if (cacheValue != null)  
                {  
                    return cacheValue;  
                }  
                else  
                {  
                    lock (lockKey)  
                    {  
                        cacheValue = CacheHelper.Get(cacheKey);  
                        if (cacheValue != null)  
                        {  
                            return cacheValue;  
                        }  
                        else  
                        {  
                            cacheValue = GetProductListFromDB(); //这里一般是 sql查询数据。                
                            CacheHelper.Add(cacheKey, cacheValue, cacheTime);  
                        }                      
                    }  
                    return cacheValue;  
                }  
            }  
```

2) 加锁排队只是为了减轻数据库的压力，并没有提高系统吞吐量。假设在高并发下，缓存重建期间key是锁着的，这是过来1000个请求999个都在阻塞的。同样会导致用户等待超时，这是个治标不治本的方法。
　还有一个解决办法解决方案是：给每一个缓存数据增加相应的缓存标记，记录缓存的是否失效，如果缓存标记失效，则更新数据缓存。

```java
    public object GetProductListNew()  
            {  
                const int cacheTime = 30;  
                const string cacheKey = "product_list";  
                //缓存标记。  
                const string cacheSign = cacheKey + "_sign";  
                  
                var sign = CacheHelper.Get(cacheSign);  
                //获取缓存值  
                var cacheValue = CacheHelper.Get(cacheKey);  
                if (sign != null)  
                {  
                    return cacheValue; //未过期，直接返回。  
                }  
                else  
                {  
                    CacheHelper.Add(cacheSign, "1", cacheTime);  
                    ThreadPool.QueueUserWorkItem((arg) =>  
                    {  
                        cacheValue = GetProductListFromDB(); //这里一般是 sql查询数据。  
                        CacheHelper.Add(cacheKey, cacheValue, cacheTime*2); //日期设缓存时间的2倍，用于脏读。                  
                    });  
                      
                    return cacheValue;  
                }  
            }  
```

3) 缓存数据
它的过期时间比缓存标记的时间延长1倍，例：标记缓存时间30分钟，数据缓存设置为60分钟。 这样，当缓存标记key过期后，实际缓存还能把旧数据返回给调用端，直到另外的线程在后台更新完成后，才会返回新缓存。这样做后，就可以一定程度上提高系统吞吐量。

**给缓存的失效时间**，加上一个随机值，避免集体失效。

**使用互斥锁**，但是该方案吞吐量明显下降了。

**双缓存**。我们有两个缓存，缓存 A 和缓存 B。缓存 A 的失效时间为 20 分钟，缓存 B 不设失效时间。自己做缓存预热操作。

然后细分以下几个小点：从缓存 A 读数据库，有则直接返回；A 没有数据，直接从 B 读数据，直接返回，并且异步启动一个更新线程，更新线程同时更新缓存 A 和缓存 B。

## 缓存穿透
缓存穿透是指用户查询数据，在数据库没有，自然在缓存中也不会有。这样就导致用户查询的时候，在缓存中找不到，每次都要去数据库再查询一遍，然后返回空。这样请求就绕过缓存直接查数据库，这也是经常提的缓存命中率问题。
解决的办法就是：如果查询数据库也为空，直接设置一个默认值存放到缓存，这样第二次到缓冲中获取就有值了，而不会继续访问数据库，这种办法最简单粗暴。

```java

    public object GetProductListNew()  
            {  
                const int cacheTime = 30;  
                const string cacheKey = "product_list";  
      
                var cacheValue = CacheHelper.Get(cacheKey);  
                if (cacheValue != null)  
                    return cacheValue;  
                      
                cacheValue = CacheHelper.Get(cacheKey);  
                if (cacheValue != null)  
                {  
                    return cacheValue;  
                }  
                else  
                {  
                    cacheValue = GetProductListFromDB(); //数据库查询不到，为空。  
                      
                    if (cacheValue == null)  
                    {  
                        cacheValue = string.Empty; //如果发现为空，设置个默认值，也缓存起来。                  
                    }  
                    CacheHelper.Add(cacheKey, cacheValue, cacheTime);  
                      
                    return cacheValue;  
                }  
            }  
```

把空结果，也给缓存起来，这样下次同样的请求就可以直接返回空了，即可以避免当查询的值为空时引起的缓存穿透。同时也可以单独设置个缓存区域存储空值，对要查询的key进行预先校验，然后再放行给后面的正常缓存处理逻辑。 

**利用互斥锁**，缓存失效的时候，先去获得锁，得到锁了，再去请求数据库。没得到锁，则休眠一段时间重试。

**采用异步更新策略**，无论 Key 是否取到值，都直接返回。Value 值中维护一个缓存失效时间，缓存如果过期，异步起一个线程去读数据库，更新缓存。需要做缓存预热(项目启动前，先加载缓存)操作。

**提供一个能迅速判断请求是否有效的拦截机制**，比如，利用布隆过滤器，内部维护一系列合法有效的 Key。迅速判断出，请求所携带的 Key 是否合法有效。如果不合法，则直接返回。

## 缓存预热
　　缓存预热就是系统上线后，将相关的缓存数据直接加载到缓存系统。这样避免，用户请求的时候，再去加载相关的数据。
解决思路：
1) 直接写个缓存刷新页面，上线时手工操作下。
2) 数据量不大，可以在WEB系统启动的时候加载。
3) 定时刷新缓存

## 缓存更新
存淘汰的策略有两种：
　　(1)定时去清理过期的缓存。
　　(2)当有用户请求过来时，再判断这个请求所用到的缓存是否过期，过期的话就去底层系统得到新数据并更新缓存。 
　　两者各有优劣，第一种的缺点是维护大量缓存的key是比较麻烦的，第二种的缺点就是每次用户请求过来都要判断缓存失效，逻辑相对比较复杂，具体用哪种方案，大家可以根据自己的应用场景来权衡。1. 预估失效时间 2. 版本号（必须单调递增，时间戳是最好的选择）3. 提供手动清理缓存的接口。

参考1 :[缓存穿透、缓存并发、缓存失效之思路变迁](https://mp.weixin.qq.com/s/KlOl8Q_sH1WRvbQqhgvy3Q)

[toTop](#jump)

# 使用Spring Session redis进行Session共享
redis集群做主从复制，利用redis数据库的最终一致性，将session信息存入redis中。当应用服务器发现session不在本机内存的时候，就去redis数据库中查找，因为redis数据库是独立于应用服务器的数据库，所以可以做到session的共享和高可用。

不足：
1.redis需要内存较大，否则会出现用户session从Cache中被清除。

2.需要定期的刷新缓存

初步结构如下：
![](/img/redis_session_share.png)

但是这个结构仍然存在问题，redis master是一个重要瓶颈，如果master崩溃的时候，但是redis不会主动的进行master切换，这时session服务中断。

Spring Boot提供了Spring Session来完成session共享。

[官方文档](http://docs.spring.io/spring-session/docs/current/reference/html5/guides/boot.html#boot-sample)

例子

首先创建简单的Controller

```java
    @Controller  
    public class UserController {  
          
        @RequestMapping(value="/main", method=RequestMethod.GET)  
        public String main(HttpServletRequest request) {  
            HttpSession session = request.getSession();  
            String sessionId = (String) session.getAttribute("sessionId");  
            if (null != sessionId) { // sessionId不为空  
                System.out.println("main sessionId:" + sessionId);  
                return "main";  
            } else { // sessionId为空  
                return "redirect:/login";  
            }  
        }  
          
          
        @RequestMapping(value="/login", method=RequestMethod.GET)  
        public String login() {  
            return "login";  
        }  
          
        @RequestMapping(value="/doLogin", method=RequestMethod.POST)  
        public String doLogin(HttpServletRequest request) {  
            System.out.println("I do real login here");  
            HttpSession session = request.getSession();  
            String sessionId = UUID.randomUUID().toString();  
            session.setAttribute("sessionId", sessionId);  
            System.out.println("login sessionId:" + sessionId);  
            return "redirect:/main";  
        }  
    }  
```

加入以下依赖

```java
    <!-- spring session -->  
    <dependency>  
        <groupId>org.springframework.session</groupId>  
        <artifactId>spring-session</artifactId>  
        <version>1.3.0.RELEASE</version>  
    </dependency>  
    <!-- redis -->  
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-redis</artifactId>  
    </dependency>  
```

增加配置类

```java
    @EnableRedisHttpSession  
    public class HttpSessionConfig {  
        @Bean  
        public JedisConnectionFactory connectionFactory() {  
                return new JedisConnectionFactory();  
        }  
    }  
```

这个配置类可以创建一个过滤器，这个过滤器支持Spring Session代替HttpSession发挥作用。
@EnableRedisHttpSession注解会创建一个springSessionRepositoryFilter的bean对象去实现这个过滤器。过滤器负责代替HttpSession。
也就是说，HttpSession不再发挥作用，而是通过过滤器使用redis直接操作Session。

在application.properties中添加redis的配置

```java
    spring.redis.host=localhost  
    spring.redis.password=  
    spring.redis.port=6379  
```

这样，就完成了Session共享了。是不是很简单？业务代码甚至不需要一点点的修改。 

[toTop](#jump)

# Redis两种持久化方式

## RDB

RDB持久化是指在指定的时间间隔内将内存中的数据集快照写入磁盘，实际操作过程是fork一个子进程，先将数据集写入临时文件，写入成功后，再替换之前的文件，用二进制压缩存储。
![](/img/redis_RDB.png)

## AOF

AOF持久化以日志的形式记录服务器所处理的每一个写、删除操作，查询操作不会记录，以文本的方式记录，可以打开文件看到详细的操作记录。

![](/img/redis_AOF.png)

## 二者优缺点

RDB 的优点:
RDB 是一个非常紧凑（compact）的文件，它保存了 Redis 在某个时间点上的数据集。
RDB 的缺点:
RDB 文件需要保存整个数据集的状态， 所以它并不是一个轻松的操作.
如果数据集非常巨大，并且 CPU 时间非常紧张的话，那么这种停止时间甚至可能会长达整整一秒。 虽然 AOF 重写也需要进行 fork() ，但无论 AOF 重写的执行间隔有多长，数据的耐久性都不会有任何损失。

AOF 的优点:
AOF 的默认策略为每秒钟 fsync 一次，在这种配置下，Redis 仍然可以保持良好的性能，并且就算发生故障停机，也最多只会丢失一秒钟的数据（ fsync 会在后台线程执行，所以主线程可以继续努力地处理命令请求）。
AOF 的缺点:
相同的数据集来说，AOF 文件的体积通常要大于 RDB 文件的体积。
关闭 fsync 可以让 AOF 的速度和 RDB 一样快， 即使在高负荷之下也是如此。

参考资料1: [redis的 rdb 和 aof 持久化的区别](https://blog.csdn.net/jackpk/article/details/30073097)

参考资料2：[Redis教程(十)：持久化详解](http://www.jb51.net/article/65264.htm)

[toTop](#jump)

# redis为什么要设计成单线程

CPU不是Redis的瓶颈。Redis的瓶颈最有可能是机器内存或者网络带宽。
redis用单线程也不是没有问题。有一个很明显的问题就是。当进行一些复杂的集合操作的时候会使redis并发性下降。 
解决办法是：你可以在一个多核的机器上部署多个redis实例。组成master-master，master-slave的形式，实现读写分离。耗时的读命令完全可以放到slave中。

参考 : [redis为什么要设计成单线程](https://www.zybuluo.com/Toby-wei/note/466127)

参考 : [Redis为什么是单线程](https://blog.csdn.net/qqqqq1993qqqqq/article/details/77538202)

[toTop](#jump)

# Redis实现Session共享

参考 : [spring boot + redis 实现session共享](https://www.cnblogs.com/mengmeng89012/p/5519698.html)

参考 : [springboot+springsession+redis+feign实现session共享](https://blog.csdn.net/niemingming/article/details/80862949)

[toTop](#jump)

# Redis 的过期策略和内存淘汰机制

* Redis 采用的是定期删除+惰性删除策略
1) 定时删除，用一个定时器来负责监视 Key，过期则自动删除。虽然内存及时释放，但是十分消耗 CPU 资源。在大并发请求下，CPU 要将时间应用在处理请求，而不是删除 Key，因此没有采用这一策略。

2) 定期删除：Redis 默认每个 100ms 检查，有过期 Key 则删除。需要说明的是，Redis 不是每个 100ms 将所有的 Key 检查一次，而是随机抽取进行检查。
惰性删除：放任键过期不管，但是每次从键空间中获取键时，都检查取得的键是否过期，如果过期的话，就删除该键；如果没有过期，那就返回该键；

* 采用定期删除+惰性删除就没其他问题了么
不是的，如果定期删除没删除掉 Key。并且你也没及时去请求 Key，也就是说惰性删除也没生效。这样，Redis 的内存会越来越高。那么就应该采用内存淘汰机制。
在 redis.conf 中有一行配置

```
maxmemory-policy volatile-lru
```
配置就是配内存淘汰策略的
* **noeviction**：当内存不足以容纳新写入数据时，新写入操作会报错。

* **allkeys-lru**：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的 Key。（推荐使用，目前项目在用这种）(最近最久使用算法)

* **allkeys-random**：当内存不足以容纳新写入数据时，在键空间中，随机移除某个 Key。（应该也没人用吧，你不删最少使用 Key，去随机删）

* **volatile-lru**：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的 Key。这种情况一般是把 Redis 既当缓存，又做持久化存储的时候才用。（不推荐）

* **volatile-random**：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个 Key。（依然不推荐）

* **volatile-ttl**：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的 Key 优先移除。（不推荐）

参考 1 ：[Redis过期键的删除策略](https://www.cnblogs.com/lukexwang/p/4694094.html)

参考 2 ：[为什么我们做分布式要用 Redis](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247485453&idx=1&sn=03079a5c528d6440ca31c72a09f8a202&chksm=fa4977bccd3efeaa7139c0da5bb7febccaf529dae2f9877c835af72b3d00ab4d082f74d86880&mpshare=1&scene=1&srcid=1104HxQYSc9yblXdOfECYZfi#rd)

[toTop](#jump)

# Redis 和数据库双写一致性问题
有强一致性要求的数据，不能放缓存。首先，采取正确更新策略，先更新数据库，再删缓存。其次，因为可能存在删除缓存失败的问题，提供一个补偿措施即可，例如利用消息队列。

参考 1 ：[为什么我们做分布式要用 Redis](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247485453&idx=1&sn=03079a5c528d6440ca31c72a09f8a202&chksm=fa4977bccd3efeaa7139c0da5bb7febccaf529dae2f9877c835af72b3d00ab4d082f74d86880&mpshare=1&scene=1&srcid=1104HxQYSc9yblXdOfECYZfi#rd)

[toTop](#jump)

# 如何解决 Redis 的并发竞争 Key 问题

并不推荐使用 Redis 的事务机制。**因为我们的生产环境，基本都是 Redis 集群环境，做了数据分片操作**。你一个事务中有涉及到多个 Key 操作的时候，这多个 Key 不一定都存储在同一个 redis-server 上。因此，Redis 的事务机制，十分鸡肋。

* 如果对这个 Key 操作，不要求顺序
这种情况下，准备一个分布式锁，大家去抢锁，抢到锁就做 set 操作即可，比较简单。

* 如果对这个 Key 操作，要求顺序

假设有一个 key1，系统 A 需要将 key1 设置为 valueA，系统 B 需要将 key1 设置为 valueB，系统 C 需要将 key1 设置为 valueC。

期望按照 key1 的 value 值按照 valueA > valueB > valueC 的顺序变化。这种时候我们在数据写入数据库的时候，需要保存一个时间戳。

假设时间戳如下：

```
系统 A key 1 {valueA 3:00}

系统 B key 1 {valueB 3:05}

系统 C key 1 {valueC 3:10}
```

那么，假设系统 B 先抢到锁，将 key1 设置为{valueB 3:05}。接下来系统 A 抢到锁，发现自己的 valueA 的时间戳早于缓存中的时间戳，那就不做 set 操作了，以此类推。其他方法，比如利用队列，将 set 方法变成串行访问也可以。加君羊：874811168即可免费领取架构资料一份。

参考 1 ：[为什么我们做分布式要用 Redis](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247485453&idx=1&sn=03079a5c528d6440ca31c72a09f8a202&chksm=fa4977bccd3efeaa7139c0da5bb7febccaf529dae2f9877c835af72b3d00ab4d082f74d86880&mpshare=1&scene=1&srcid=1104HxQYSc9yblXdOfECYZfi#rd)

[toTop](#jump)


# Redis基本类型及应用

在互联网公司一般有以下应用：
String：缓存、限流、计数器、分布式锁、分布式Session
Hash：存储用户信息、用户主页访问量、组合查询
List：微博关注人时间轴列表、简单队列
Set：赞、踩、标签、好友关系
Zset：排行榜

## String
字符串对象的底层实现可以是``int``、``raw``、``embstr``（上面的表对应有名称介绍）。**embstr编码是通过调用一次内存分配函数来分配一块连续的空间，而raw需要调用两次**。
int编码字符串对象和embstr编码字符串对象在一定条件下会转化为raw编码字符串对象。embstr：<=39字节的字符串。int：8个字节的长整型。raw：大于39个字节的字符串。

```
get：sdsrange---O(n)
set：sdscpy—O(n)
create：sdsnew---O(1)
len：sdslen---O(1)
```

## List
List对象的底层实现是``quicklist``（快速列表，是``ziplist`` 压缩列表 和``linkedlist`` 双端链表 的组合）。Redis中的列表支持两端插入和弹出，并可以获得指定位置（或范围）的元素，可以充当数组、队列、栈等。

```
rpush: listAddNodeHead ---O(1)
lpush: listAddNodeTail ---O(1)
push:listInsertNode ---O(1)
index : listIndex ---O(N)
pop:ListFirst/listLast ---O(1)
llen:listLength ---O(N)
```

## Hash
Hash对象的底层实现可以是ziplist（压缩列表）或者hashtable（字典或者也叫哈希表）
Hash对象只有同时满足下面两个条件时，才会使用ziplist（压缩列表）：1.哈希中元素数量小于512个；2.哈希中所有键值对的键和值字符串长度都小于64字节。
使用链地址法来解决键冲突

## Set
Set集合对象的底层实现可以是intset（整数集合）或者hashtable（字典或者也叫哈希表）
intset（整数集合）当一个集合只含有整数，并且元素不多时会使用intset（整数集合）作为Set集合对象的底层实现
intset底层实现为有序，无重复数组保存集合元素

```
sadd:intsetAdd---O(1)
smembers:intsetGetO(1)---O(N)
srem:intsetRemove---O(N)
slen:intsetlen ---O(1)
```

## ZSet
ZSet有序集合对象底层实现可以是ziplist（压缩列表）或者skiplist（跳跃表）
当一个有序集合的元素数量比较多或者成员是比较长的字符串时，Redis就使用skiplist（跳跃表）作为ZSet对象的底层实现。

```
zadd---zslinsert---平均O(logN), 最坏O(N)
zrem---zsldelete---平均O(logN), 最坏O(N)
zrank--zslGetRank---平均O(logN), 最坏O(N)
```


参考：[Redis 为何这么快？聊聊它的数据结构](https://mp.weixin.qq.com/s/69xl2yU4B97aQIn1k_Lwqw)
[toTop](#jump)
