<a id = "jump">[首页](/README.md)</a>

<!-- TOC -->

- [Synchronized](#synchronized)
- [synchronized与Lock的区别](#synchronized与lock的区别)
- [volatile](#volatile)
- [可重入锁](#可重入锁)
    - [ReentrantLock](#reentrantlock)
    - [AQS](#aqs)
- [无锁CAS](#无锁cas)
    - [ABA问题](#aba问题)
- [分布式锁](#分布式锁)
    - [基于数据库实现分布式锁](#基于数据库实现分布式锁)
    - [基于缓存（Redis等）实现分布式锁](#基于缓存redis等实现分布式锁)
    - [基于Zookeeper实现分布式锁](#基于zookeeper实现分布式锁)

<!-- /TOC -->


# Synchronized
synchronized是java中的一个关键字。

* 应用
    1) 修饰实例方法，作用于当前实例加锁，进入同步代码前要获得当前实例的锁
    2) 修饰静态方法，作用于当前类对象加锁，进入同步代码前要获得当前类对象的锁
    3) 修饰代码块，指定加锁对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁。

同步方法 并不是由 monitorenter 和 monitorexit 指令来实现同步的，而是由方法调用指令读取运行时常量池中方法的 ACC_SYNCHRONIZED 标志来隐式实现的。

参考1 ：[深入理解Java并发之synchronized实现原理](https://blog.csdn.net/javazejian/article/details/72828483)

[toTop](#jump)

# synchronized与Lock的区别

![](/img/synchronizedAndLock.png)

[toTop](#jump)

# volatile

volatile具备两种特性，第一就是**保证共享变量对所有线程的可见性**(所谓可见性，是指当一条线程修改了共享变量的值，新值对于其他线程来说是可以立即得知的)。将一个共享变量声明为volatile后，会有以下效应：
1) 当写一个volatile变量时，JMM会把该线程对应的本地内存中的变量强制刷新到主内存中去；
2) 这个写会操作会导致其他线程中的缓存无效。

volatile还有一个特性：**禁止指令重排序优化**(重排序是指编译器和处理器为了优化程序性能而对指令序列进行排序的一种手段。)。

1) 重排序操作不会对存在数据依赖关系的操作进行重排序。
比如：a=1;b=a; 这个指令序列，由于第二个操作依赖于第一个操作，所以在编译时和处理器运行时这两个操作不会被重排序。

2) 重排序是为了优化性能，但是不管怎么重排序，单线程下程序的执行结果不能被改变
比如：a=1;b=2;c=a+b这三个操作，第一步（a=1)和第二步(b=2)由于不存在数据依赖关系，所以可能会发生重排序，但是c=a+b这个操作是不会被重排序的，因为需要保证最终的结果一定是c=a+b=3。

**volitale不能保证原子性。**

使用volatile必须具备以下2个条件：
1) 对变量的写操作不依赖于当前值
2) 该变量没有包含在具有其他变量的不变式中

参考1 ：[谈谈Java中的volatile](https://www.cnblogs.com/chengxiao/p/6528109.html)

参考2 : [就是要你懂 Java 中 volatile 关键字实现原理](http://www.importnew.com/27002.html)

参考3 : [死磕Java并发：深入分析volatile的实现原理](http://www.importnew.com/23520.html)

[toTop](#jump)

# 可重入锁

可重入锁，也叫做递归锁，指的是同一线程 外层函数获得锁之后 ，内层递归函数仍然有获取该锁的代码，但不受影响。
ReentrantLock 和synchronized 都是 可重入锁。
可重入锁最大的作用是**避免死锁**。

## ReentrantLock

ReetrantLock是基于AQS并发框架实现的。

## AQS
AQS即是AbstractQueuedSynchronizer,内部通过一个int类型的成员变量state来控制同步状态(对同步状态执行CAS操作),当state=0时，则说明没有任何线程占有共享资源的锁，当state=1时，则说明有线程目前正在使用共享变量，其他线程必须加入同步队列进行等待，AQS内部通过内部类Node构成FIFO的同步队列来完成线程获取锁的排队工作，同时利用内部类ConditionObject构建等待队列，当Condition调用wait()方法后，线程将会加入等待队列中，而当Condition调用signal()方法后，线程将从等待队列转移到同步队列中进行锁竞争。

参考1 ：[深入剖析基于并发AQS的(独占锁)重入锁(ReetrantLock)及其Condition实现原理](https://blog.csdn.net/javazejian/article/details/75043422)

[toTop](#jump)

# 无锁CAS
CAS有3个操作数，内存值V，旧的预期值A，要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。
下面是sun.misc.Unsafe类的compareAndSwapInt()方法的源代码

```java
public final native boolean compareAndSwapInt(Object o, long offset,
                                              int expected,
                                              int x);
```

CAS存在的问题
1) ABA问题。
2) 循环时间长开销大。
3) 只能保证一个共享变量的原子操作。

## ABA问题
线程1准备用CAS将变量的值由A替换为B，在此之前，线程2将变量的值由A替换为C，又由C替换为A，然后线程1执行CAS时发现变量的值仍然为A，所以CAS成功。
例:

![](/img/ABA_1.png)

现有一个用单向链表实现的堆栈，栈顶为A，这时线程T1已经知道A.next为B，然后希望用CAS将栈顶替换为B：

head.compareAndSet(A,B);

在T1执行上面这条指令之前，线程T2介入，将A、B出栈，再pushD、C、A，此时堆栈结构如下图，而对象B此时处于游离状态：

![](/img/ABA_2.png)

此时轮到线程T1执行CAS操作，检测发现栈顶仍为A，所以CAS成功，栈顶变为B，但实际上B.next为null，所以此时的情况变为：

![](/img/ABA_3.png)

其中堆栈中只有B一个元素，C和D组成的链表不再存在于堆栈中，平白无故就把C、D丢掉了。

以上就是由于ABA问题带来的隐患，各种乐观锁的实现中通常都会用版本戳version来对记录或对象标记，避免并发操作带来的问题，在Java中，AtomicStampedReference也实现了这个作用，它通过包装[E,Integer]的元组来对对象标记版本戳stamp，从而避免ABA问题。

如下面的代码分别用AtomicInteger和AtomicStampedReference来对初始值为100的原子整型变量进行更新，AtomicInteger会成功执行CAS操作，而加上版本戳的AtomicStampedReference对于ABA问题会执行CAS失败：

```java
public class ABA {
    private static AtomicInteger atomicInt = new AtomicInteger(100);
    private static AtomicStampedReference atomicStampedRef = new AtomicStampedReference(100, 0);
    public static void main(String[] args) throws InterruptedException {
        Thread intT1 = new Thread(new Runnable() {
            @Override
            public void run() {
                atomicInt.compareAndSet(100, 101);
                atomicInt.compareAndSet(101, 100);
            }
        });
        Thread intT2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                }
                boolean c3 = atomicInt.compareAndSet(100, 101);
                System.out.println(c3); // true
            }
        });
        intT1.start();
        intT2.start();
        intT1.join();
        intT2.join();
        Thread refT1 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                }
                atomicStampedRef.compareAndSet(100, 101, atomicStampedRef.getStamp(), atomicStampedRef.getStamp() + 1);
                atomicStampedRef.compareAndSet(101, 100, atomicStampedRef.getStamp(), atomicStampedRef.getStamp() + 1);
            }
        });
        Thread refT2 = new Thread(new Runnable() {
            @Override
            public void run() {
                int stamp = atomicStampedRef.getStamp();
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                }
                boolean c3 = atomicStampedRef.compareAndSet(100, 101, stamp, stamp + 1);
                System.out.println(c3); // false
            }
        });
        refT1.start();
        refT2.start();
    }
}
```

参考1 ：[Java中CAS详解](https://blog.csdn.net/ls5718/article/details/52563959)

参考2 : [传说中的并发编程ABA问题](https://blog.csdn.net/u012813201/article/details/72841801)

参考3 :[Java并发编程-无锁CAS与Unsafe类及其并发包Atomic](https://blog.csdn.net/javazejian/article/details/72772470)

[toTop](#jump)

# 分布式锁

## 基于数据库实现分布式锁

基于数据库的实现方式的核心思想是：在数据库中创建一个表，表中包含方法名等字段，并在方法名字段上创建唯一索引，想要执行某个方法，就使用这个方法名向表中插入数据，成功插入则获取锁，执行完成后删除对应的行数据释放锁。

1) 创建一个表：

```java
DROP TABLE IF EXISTS `method_lock`;
CREATE TABLE `method_lock` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
  `method_name` varchar(64) NOT NULL COMMENT '锁定的方法名',
  `desc` varchar(255) NOT NULL COMMENT '备注信息',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uidx_method_name` (`method_name`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8 COMMENT='锁定中的方法';
```

2) 想要执行某个方法，就使用这个方法名向表中插入数据：

```java
INSERT INTO method_lock (method_name, desc) VALUES ('methodName', '测试的methodName');
```

因为我们**对method_name做了唯一性约束**，这里如果有多个请求同时提交到数据库的话，数据库会保证只有一个操作可以成功，那么我们就可以认为操作成功的那个线程获得了该方法的锁，可以执行方法体内容。

3) 成功插入则获取锁，执行完成后删除对应的行数据释放锁：

```java
delete from method_lock where method_name ='methodName';
```

## 基于缓存（Redis等）实现分布式锁
* 实现思想
1) 获取锁的时候，使用setnx加锁，并使用expire命令为锁添加一个超时时间，超过该时间则自动释放锁，锁的value值为一个随机生成的UUID，通过此在释放锁的时候进行判断。
2) 获取锁的时候还设置一个获取的超时时间，若超过这个时间则放弃获取锁。
3) 释放锁的时候，通过UUID判断是不是该锁，若是该锁，则执行delete进行锁释放。

* 简单实现代码

```java
public class DistributedLock {

    private final JedisPool jedisPool;

    public DistributedLock(JedisPool jedisPool) {
        this.jedisPool = jedisPool;
    }

    /**
     * 加锁
     * @param lockName       锁的key
     * @param acquireTimeout 获取超时时间
     * @param timeout        锁的超时时间
     * @return 锁标识
     */
    public String lockWithTimeout(String lockName, long acquireTimeout, long timeout) {
        Jedis conn = null;
        String retIdentifier = null;
        try {
            // 获取连接
            conn = jedisPool.getResource();
            // 随机生成一个value
            String identifier = UUID.randomUUID().toString();
            // 锁名，即key值
            String lockKey = "lock:" + lockName;
            // 超时时间，上锁后超过此时间则自动释放锁
            int lockExpire = (int) (timeout / 1000);

            // 获取锁的超时时间，超过这个时间则放弃获取锁
            long end = System.currentTimeMillis() + acquireTimeout;
            while (System.currentTimeMillis() < end) {
                if (conn.setnx(lockKey, identifier) == 1) {
                    conn.expire(lockKey, lockExpire);
                    // 返回value值，用于释放锁时间确认
                    retIdentifier = identifier;
                    return retIdentifier;
                }
                // 返回-1代表key没有设置超时时间，为key设置一个超时时间
                if (conn.ttl(lockKey) == -1) {
                    conn.expire(lockKey, lockExpire);
                }

                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        } catch (JedisException e) {
            e.printStackTrace();
        } finally {
            if (conn != null) {
                conn.close();
            }
        }
        return retIdentifier;
    }

    /**
     * 释放锁
     * @param lockName   锁的key
     * @param identifier 释放锁的标识
     * @return
     */
    public boolean releaseLock(String lockName, String identifier) {
        Jedis conn = null;
        String lockKey = "lock:" + lockName;
        boolean retFlag = false;
        try {
            conn = jedisPool.getResource();
            while (true) {
                // 监视lock，准备开始事务
                conn.watch(lockKey);
                // 通过前面返回的value值判断是不是该锁，若是该锁，则删除，释放锁
                if (identifier.equals(conn.get(lockKey))) {
                    Transaction transaction = conn.multi();
                    transaction.del(lockKey);
                    List<Object> results = transaction.exec();
                    if (results == null) {
                        continue;
                    }
                    retFlag = true;
                }
                conn.unwatch();
                break;
            }
        } catch (JedisException e) {
            e.printStackTrace();
        } finally {
            if (conn != null) {
                conn.close();
            }
        }
        return retFlag;
    }
}
```

## 基于Zookeeper实现分布式锁

基于ZooKeeper实现分布式锁的步骤如下：
1) 创建一个目录mylock； 
2) 线程A想获取锁就在mylock目录下创建临时顺序节点； 
3) 获取mylock目录下所有的子节点，然后获取比自己小的兄弟节点，如果不存在，则说明当前线程顺序号最小，获得锁； 
4) 线程B获取所有节点，判断自己不是最小节点，设置监听比自己次小的节点； 
5) 线程A处理完，删除自己的节点，线程B监听到变更事件，判断自己是不是最小的节点，如果是则获得锁。

可以直接使用zookeeper第三方库Curator客户端，这个客户端中封装了一个可重入的锁服务。

```java
public boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException {
    try {
        return interProcessMutex.acquire(timeout, unit);
    } catch (Exception e) {
        e.printStackTrace();
    }
    return true;
}
public boolean unlock() {
    try {
        interProcessMutex.release();
    } catch (Throwable e) {
        log.error(e.getMessage(), e);
    } finally {
        executorService.schedule(new Cleaner(client, path), delayTimeForClean, TimeUnit.MILLISECONDS);
    }
    return true;
}
```

Curator提供的InterProcessMutex是分布式锁的实现。acquire方法用户获取锁，release方法用于释放锁。

优点：具备高可用、可重入、阻塞锁特性，可解决失效死锁问题。

缺点：因为需要频繁的创建和删除节点，性能上不如Redis方式。zk锁的问题在于zk是通过心跳监控进程存活状态，如果进程进行GC pause或者因为网络原因导致很长时间没与zk联系，则将导致zk认为进程已挂，而后锁自动释放，而此时进程并未挂任然在执行。

参考1 ：[分布式锁简单入门以及三种实现方式介绍](https://blog.csdn.net/xlgen157387/article/details/79036337)

参考2 ：[三种方式实现分布式锁](https://blog.csdn.net/guoyunyuhou/article/details/79108717)

[toTop](#jump)
