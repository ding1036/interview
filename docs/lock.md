<a id = "jump">[首页](/README.md)</a>

<!-- TOC -->

- [Synchronized](#synchronized)
- [synchronized与Lock的区别](#synchronized与lock的区别)
- [volatile](#volatile)
- [可重入锁](#可重入锁)
    - [ReentrantLock](#reentrantlock)
    - [AQS](#aqs)
- [无锁CAS](#无锁cas)

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

# 无锁CAS
