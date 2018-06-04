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

