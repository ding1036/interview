<a id = "jump">[首页](/README.md)</a>


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