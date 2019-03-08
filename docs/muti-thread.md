<a id = "jump">[首页](/README.md)</a>

<!-- TOC -->

- [线程池](#线程池)
    - [四种线程池](#四种线程池)
    - [ExecutorService中submit和execute的区别](#executorservice中submit和execute的区别)
    - [四种拒绝策略](#四种拒绝策略)
- [CountDownLatch CyclicBarrier Semaphore Exchange](#countdownlatch-cyclicbarrier-semaphore-exchange)
    - [CountDownLatch](#countdownlatch)
    - [CyclicBarrier](#cyclicbarrier)
    - [Semaphore](#semaphore)
    - [Exchange](#exchange)
- [线程状态](#线程状态)
- [实现线程的三种方式](#实现线程的三种方式)
    - [继承thread类](#继承thread类)
    - [实现runnable 接口](#实现runnable-接口)
    - [实现callable 接口](#实现callable-接口)
- [ThreadLocal](#threadlocal)
    - [ThreadLocal类提供的几个方法](#threadlocal类提供的几个方法)
    - [ThreadLocalMap](#threadlocalmap)
    - [解决Hash冲突](#解决hash冲突)
    - [如何避免泄漏](#如何避免泄漏)
    - [总结](#总结)
- [守护线程与普通线程](#守护线程与普通线程)
- [sleep wait 区别](#sleep-wait-区别)

<!-- /TOC -->

# 线程池

## 四种线程池
java.util.concurrent.Executors工厂类可以创建四种类型的线程池，通过Executors.newXXX方法即可创建。
```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```
1) CachedThreadPool是一种可以无限扩容的线程池； 
- CachedThreadPool比较适合执行时间片比较小的任务； 
- keepAliveTime为60，意味着线程空闲时间超过60s就会被杀死； 
- 阻塞队列采用SynchronousQueue，这种阻塞队列没有存储空间，意味着只要有任务到来，就必须得有一个工作线程来处理，如果当前没有空闲线程，那么就再创建一个新的线程。
```java
public static ExecutorService newCachedThreadPool(){
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
}
```

2) ScheduledThreadPool接收SchduledFutureTask类型的任务，提交任务的方式有2种； 
    1. scheduledAtFixedRate； 
    2. scheduledWithFixedDelay； 
    - SchduledFutureTask接收参数： 
        time：任务开始时间 
        sequenceNumber：任务序号 
        period：任务执行的时间间隔 
    - 阻塞队列采用DelayQueue，它是一种无界队列； 
    - DelayQueue内部封装了一个PriorityQueue，它会根据time的先后排序，若time相同，则根据sequenceNumber排序； 
    - 工作线程执行流程： 
        1. 工作线程会从DelayQueue中取出已经到期的任务去执行； 
        2. 执行结束后重新设置任务的到期时间，再次放回DelayQueue。

3) FixedThreadPool是一种容量固定的线程池； 
- 阻塞队列采用LinkedBlockingQueue，它是一种无界队列； 
- 由于阻塞队列是一个无界队列，因此永远不可能拒绝执行任务； 
- 由于采用无界队列，实际线程数将永远维持在nThreads，因此maximumPoolSize和keepAliveTime将无效。

```java
public static ExecutorService newFixedThreadPool(int nThreads){
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());
}
```

4) newSingleThreadExecutor 方法，它创建了一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有的任务按照指定的顺序(FIFO, LIFO, 优先级)来执行的。
```java
public static ExecutorService newSingleThreadExecutor(){
    return new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());
}
```

[toTop](#jump)

## ExecutorService中submit和execute的区别
``execute()``方法的入参为一个``Runnable``，返回值为``void``

``submit()``方法，入参可以为``Callable<T>``，也可以为``Runnable``，而且方法有返回值``Future<T>``

总结：

1. 接收的参数不一样;

2. submit()有返回值，而execute()没有;
例如，有个validation的task，希望该task执行完后告诉我它的执行结果，是成功还是失败，然后继续下面的操作。

3. submit()可以进行Exception处理;
例如，如果task里会抛出``checked``或者``unchecked exception``，而你又希望外面的调用者能够感知这些exception并做出及时的处理，那么就需要用到``submit``，通过对``Future.get()``进行抛出异常的捕获，然后对其进行处理。

参考1 ：[ExecutorService中submit和execute区别](https://blog.csdn.net/kaixuanfeng2012/article/details/74786052)

参考2 ：[ExecutorService中submit和execute区别](https://www.cnblogs.com/wanqieddy/p/3853863.html)

[toTop](#jump)

## 四种拒绝策略

1) CallerRunsPolicy：线程调用运行该任务的 execute 本身。此策略提供简单的反馈控制机制，能够减缓新任务的提交速度。
```java
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
     if (!e.isShutdown()){
          r.run(); 
          }
      }
```
这个策略显然不想放弃执行任务。但是由于池中已经没有任何资源了，那么就直接使用调用该execute的线程本身来执行。

2) AbortPolicy：处理程序遭到拒绝将抛出运行时异常 RejectedExecutionException
```java
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    throw new RejectedExecutionException();
    }
```
这种策略直接抛出异常，丢弃任务。（jdk默认策略，队列满并线程满时直接拒绝添加新任务，并抛出异常，所以说有时候放弃也是一种勇气，为了保证后续任务的正常进行，丢弃一些也是可以接收的，记得做好记录）

3) DiscardPolicy：不能执行的任务将被删除
```java
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {}
```
这种策略和AbortPolicy几乎一样，也是丢弃任务，只不过他不抛出异常。

4) DiscardOldestPolicy：如果执行程序尚未关闭，则位于工作队列头部的任务将被删除，然后重试执行程序（如果再次失败，则重复此过程）
```java
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) { 
    if (!e.isShutdown()) {
        e.getQueue().poll();e.execute(r); 
        }
    }
```
该策略就稍微复杂一些，在pool没有关闭的前提下首先丢掉缓存在队列中的最早的任务，然后重新尝试运行该任务。这个策略需要适当小心。

```java
//默认，队列满了丢任务抛出异常
RejectedExecutionHandler rejected = new ThreadPoolExecutor.AbortPolicy();

//队列满了丢任务不异常
RejectedExecutionHandler rejected = new ThreadPoolExecutor.DiscardPolicy();

//将最早进入队列的任务删，之后再尝试加入队列
RejectedExecutionHandler rejected = new ThreadPoolExecutor.DiscardOldestPolicy();

//如果添加到线程池失败，那么主线程会自己去执行该任务
RejectedExecutionHandler rejected = new ThreadPoolExecutor.CallerRunsPolicy();
```

[toTop](#jump)

# CountDownLatch CyclicBarrier Semaphore Exchange

## CountDownLatch
CountDownLatch有一个正数计数器，countDown方法对计数器做减操作，await方法等待计数器达到0。所有await的线程都会阻塞直到计数器为0或者等待线程中断或者超时。

重要方法

```java
public void await() throws InterruptedException { };   //调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
public boolean await(long timeout, TimeUnit unit) throws InterruptedException { };  //和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
public void countDown() { };  //将count值减1
```

例子：

```java
public class Test {
     public static void main(String[] args) {   
         final CountDownLatch latch = new CountDownLatch(2);
 
         new Thread(){
             public void run() {
                 try {
                     System.out.println("子线程"+Thread.currentThread().getName()+"正在执行");
                    Thread.sleep(3000);
                    System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕");
                    latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
             };
         }.start();
 
         new Thread(){
             public void run() {
                 try {
                     System.out.println("子线程"+Thread.currentThread().getName()+"正在执行");
                     Thread.sleep(3000);
                     System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕");
                     latch.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
             };
         }.start();
 
         try {
             System.out.println("等待2个子线程执行完毕...");
            latch.await();
            System.out.println("2个子线程已经执行完毕");
            System.out.println("继续执行主线程");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
     }
}
```

结果

```java
线程Thread-0正在执行
线程Thread-1正在执行
等待2个子线程执行完毕...
线程Thread-0执行完毕
线程Thread-1执行完毕
2个子线程已经执行完毕
继续执行主线程
```

await()方法。内部直接调用了AQS的acquireSharedInterruptibly(1)。

```java
    public final void acquireSharedInterruptibly(int arg) throws InterruptedException {  
        if (Thread.interrupted())  
            throw new InterruptedException();  
        if (tryAcquireShared(arg) < 0)  
            doAcquireSharedInterruptibly(arg);  
    }  
```

共享锁是说所有共享锁的线程共享同一个资源，一旦任意一个线程拿到共享资源，那么所有线程就都拥有的同一份资源。也就是通常情况下共享锁只是一个标志，所有线程都等待这个标识是否满足，一旦满足所有线程都被激活（相当于所有线程都拿到锁一样）。**这里的闭锁CountDownLatch就是基于共享锁的实现**

## CyclicBarrier

CyclicBarrier可以被重用.它允许一组线程互相等待，直到到达某个公共屏障点.

CyclicBarrier提供2个构造器

```java
public CyclicBarrier(int parties, Runnable barrierAction) {
}
 
public CyclicBarrier(int parties) {
}
```

参数parties指让多少个线程或者任务等待至barrier状态；参数barrierAction为当这些线程都达到barrier状态时会执行的内容。
CyclicBarrier中最重要的方法就是await方法

```java
public int await() throws InterruptedException, BrokenBarrierException { };//挂起当前线程，直至所有线程都到达barrier状态再同时执行后续任务；  
public int await(long timeout, TimeUnit unit)throws InterruptedException,BrokenBarrierException,TimeoutException { };//让这些线程等待至一定的时间，如果还有线程没有到达barrier状态就直接让到达barrier的线程执行后续任务 
```

例子

```java
import java.util.concurrent.CyclicBarrier;  
  
/** 
 * PROJECT_NAME:downLoad 
 * Author:lucaifang 
 * Date:2016/3/18 
 */  
public class cyclicBarrierTest {  
    public static void main(String[] args) throws InterruptedException {  
        CyclicBarrier cyclicBarrier = new CyclicBarrier(5, new Runnable() {  
            @Override  
            public void run() {  
                System.out.println("线程组执行结束");  
            }  
        });  
        for (int i = 0; i < 5; i++) {  
            new Thread(new readNum(i,cyclicBarrier)).start();  
        }  
        //CyclicBarrier 可以重复利用，  
        // 这个是CountDownLatch做不到的  
//        for (int i = 11; i < 16; i++) {  
//            new Thread(new readNum(i,cyclicBarrier)).start();  
//        }  
    }  
    static class readNum  implements Runnable{  
        private int id;  
        private CyclicBarrier cyc;  
        public readNum(int id,CyclicBarrier cyc){  
            this.id = id;  
            this.cyc = cyc;  
        }  
        @Override  
        public void run() {  
            synchronized (this){  
                System.out.println("id:"+id);  
                try {  
                    cyc.await();  
                    System.out.println("线程组任务" + id + "结束，其他任务继续");  
                } catch (Exception e) {  
                    e.printStackTrace();  
                }  
            }  
        }  
    }  
}  
```

结果

```java
id:1
id:2
id:4
id:0
id:3
线程组执行结束
线程组任务3结束，其他任务继续
线程组任务1结束，其他任务继续
线程组任务4结束，其他任务继续
线程组任务0结束，其他任务继续
线程组任务2结束，其他任务继续
```

## Semaphore
Semaphore又称信号量，信号量控制的是线程并发的数量。

```java
public Semaphore(int permits)
```
其中参数permits就是允许同时运行的线程数目;

* 例子

```java
public class Driver {
    // 将信号量设为3
    private Semaphore semaphore = new Semaphore(3);

    public void driveCar() {
        try {
            semaphore.acquire();
            System.out.println(Thread.currentThread().getName() + " start at " + System.currentTimeMillis());
            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() + " stop at " + System.currentTimeMillis());
            semaphore.release();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}




public class Car extends Thread{
    private Driver driver;

    public Car(Driver driver) {
        super();
        this.driver = driver;
    }

    public void run() {
        driver.driveCar();
    }
}

public class Run {
    public static void main(String[] args) {
        Driver driver = new Driver();
        for (int i = 0; i < 5; i++) {
            (new Car(driver)).start();
        }
    }
}
```

* 结果

```java
Thread-0 start at 1482665412515
Thread-3 start at 1482665412517
Thread-1 start at 1482665412517
Thread-3 stop at 1482665413517
Thread-0 stop at 1482665413517
Thread-4 start at 1482665413517
Thread-2 start at 1482665413517
Thread-1 stop at 1482665413518
Thread-4 stop at 1482665414517
Thread-2 stop at 1482665414517
```

* 主要方法

`) 获取许可

```java
public void acquire(int permits) throws InterruptedException {
        if (permits < 0) throw new IllegalArgumentException();
        sync.acquireSharedInterruptibly(permits);
    }
```

调用了Sync的acquireSharedInterruptibly方法，该方法在父类AQS中

```java
public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        //如果线程被中断了，抛出异常
        if (Thread.interrupted())
            throw new InterruptedException();
        //获取许可失败，将线程加入到等待队列中
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }
```

2) 释放许可
释放许可也有几个重载方法，但都会调用下面这个带参数的方法

```java
public void release(int permits) {
        if (permits < 0) throw new IllegalArgumentException();
        sync.releaseShared(permits);
    }
```

参考 : [深入理解Semaphore](https://blog.csdn.net/qq_19431333/article/details/70212663)

## Exchange

用于线程间的数据交换，提供一个同步点，在这个同步点上，两个线程可以交换数据。

例子

```java
public class ExchangeTest {  
    public static void main(String[] args) {  
        ExecutorService service =Executors.newCachedThreadPool();  
        final Exchanger exchanger = new Exchanger();  
        service.execute(new Runnable() {  
              
            @Override  
            public void run() {  
                try{  
                    String data1 = "零食";  
                    System.out.println("线程"+Thread.currentThread().getName()+  
                            "正在把数据 "+data1+" 换出去");  
                    Thread.sleep((long)Math.random()*10000);  
                    String data2 = (String)exchanger.exchange(data1);  
                    System.out.println("线程 "+Thread.currentThread().getName()+  
                            "换回的数据为 "+data2);  
                }catch(Exception e){  
                    e.printStackTrace();  
                }  
                  
            }  
        });  
          
        service.execute(new Runnable() {  
              
            @Override  
            public void run() {  
                try{  
                    String data1 = "钱";  
                    System.out.println("线程"+Thread.currentThread().getName()+  
                            "正在把数据 "+data1+" 交换出去");  
                    Thread.sleep((long)(Math.random()*10000));  
                    String data2 =(String)exchanger.exchange(data1);  
                    System.out.println("线程 "+Thread.currentThread().getName()+  
                            "交换回来的数据是: "+data2);  
                }catch(Exception e){  
                    e.printStackTrace();  
                }  
                  
                  
            }  
        });  
    }  
}  
```

结果

```java
线程pool-1-thread-1正在把数据 零食 换出去
线程pool-1-thread-2正在把数据 钱 交换出去
线程 pool-1-thread-1换回的数据为 钱
线程 pool-1-thread-2交换回来的数据是: 零食
```

[toTop](#jump)


# 线程状态

1) 新建状态：新建线程对象，并没有调用start()方法之前
2) 就绪状态：调用start()方法之后线程就进入就绪状态，但是并不是说只要调用start()方法线程就马上变为当前线程，在变为当前线程之前都是为就绪状态。值得一提的是，线程在睡眠和挂起中恢复的时候也会进入就绪状态哦。
3) 运行状态：线程被设置为当前线程，开始执行run()方法。就是线程进入运行状态
4) 阻塞状态：线程被暂停，比如说调用sleep()方法后线程就进入阻塞状态
5) 死亡状态：线程执行结束

![](/img/thread_status.jpg)

[toTop](#jump)

# 实现线程的三种方式

## 继承thread类

```java
public class MyThread extends Thread {
    private int i;

    @Override
    public void run() {
       for (; i < 10; i++) {
         System.out.println(getName()+"\t"+i);
    }
    }
}

public class ThreadTest {
    public static void main(String[] args) {
        for (int i = 0; i <10; i++) {
             System.out.println(Thread.currentThread().getName()+"\t"+i+"======");
             if(i==5){
                 
                 MyThread mt2 =new MyThread();
                 MyThread mt =new MyThread();
                 mt2.start();
                 mt.start();
             }
        }
    }
```

## 实现runnable 接口

```java
public class SecondThread implements Runnable{
    private int i;
    public void run() {
        for (; i < 20; i++) {
            System.out.println(Thread.currentThread().getName()+" "+i);
            if(i==20)
            {
                System.out.println(Thread.currentThread().getName()+"执行完毕");
            }
        }
    }
}


public class RunnableTest {
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            System.out.println(Thread.currentThread().getName()+" "+i);
            if(i==5)
            {
                SecondThread s1=new SecondThread();
                Thread t1=new Thread(s1,"线程1");
                Thread t2=new Thread(s1,"线程2");
                t1.start();
                t2.start();
               
            }
        }
    }
}
```

## 实现callable 接口

```java
public class Target implements Callable<Integer> {
    int i=0;
    public Integer call() throws Exception {
        for (; i < 20; i++) {
            System.out.println(Thread.currentThread().getName()+""+i);
        }
        return i;
    }

}


public class CallableTest {
     public static void main(String[] args) {
            Target t1=new Target();
            FutureTask<Integer> ft=new FutureTask<Integer>(t1);
            Thread t2=new Thread(ft,"新线程");
            t2.start();
            try {
                System.out.println(ft.get());
            } catch (Exception e) {
                // TODO: handle exception
            }
    }
}
```

[toTop](#jump)

# ThreadLocal

![](/img/threadlocal.jpg)

## ThreadLocal类提供的几个方法

```java
public T get() { }
public void set(T value) { }
public void remove() { }
protected T initialValue() { }

```
## ThreadLocalMap

ThreadLocalMap是ThreadLocal的内部类，没有实现Map接口，用独立的方式实现了Map的功能，其内部的Entry也独立实现。

![](/img/threadlocalmap.png)

在ThreadLocalMap中，也是用Entry来保存K-V结构数据的。但是Entry中key只能是ThreadLocal对象，这点被Entry的构造方法已经限定死了。
Entry继承自WeakReference（弱引用，生命周期只能存活到下次GC前），但只有Key是弱引用类型的，Value并非弱引用。

## 解决Hash冲突
ThreadLocalMap解决Hash冲突的方式就是简单的步长加1或减1，寻找下一个相邻的位置。

## 如何避免泄漏
既然Key是弱引用，那么我们要做的事，就是在调用ThreadLocal的get()、set()方法时完成后再调用remove方法，将Entry节点和Map的引用关系移除，这样整个Entry对象在GC Roots分析后就变成不可达了，下次GC的时候就可以被回收。

## 总结
1) ThreadLocal只是操作Thread中的ThreadLocalMap对象的集合；

2) ThreadLocalMap变量属于线程的内部属性，不同的线程拥有完全不同的ThreadLocalMap变量；

3) 线程中的ThreadLocalMap变量的值是在ThreadLocal对象进行set或者get操作时创建的；

4) 使用当前线程的ThreadLocalMap的关键在于使用当前的ThreadLocal的实例作为key来存储value值；

5) ThreadLocal模式至少从两个方面完成了数据访问隔离，即纵向隔离(线程与线程之间的ThreadLocalMap不同)和横向隔离(不同的ThreadLocal实例之间的互相隔离)；

6) 一个线程中的所有的局部变量其实存储在该线程自己的同一个map属性中；

7) 线程死亡时，线程局部变量会自动回收内存；

8) 线程局部变量时通过一个 Entry 保存在map中，该Entry 的key是一个 WeakReference包装的ThreadLocal, value为线程局部变量，key 到 value 的映射是通过：ThreadLocal.threadLocalHashCode & (INITIAL_CAPACITY - 1) 来完成的；

9) 当线程拥有的局部变量超过了容量的2/3(没有扩大容量时是10个)，会涉及到ThreadLocalMap中Entry的回收；

参考 1 : [ThreadLocal-面试必问深度解析](https://www.jianshu.com/p/98b68c97df9b)

参考 2 : [Java多线程编程-（8）-多图深入分析ThreadLocal原理](https://blog.csdn.net/xlgen157387/article/details/78297568)


[toTop](#jump) 

# 守护线程与普通线程

守护线程与普通线程的唯一区别是：当JVM中所有的线程都是守护线程的时候，JVM就可以退出了；如果还有一个或以上的非守护线程则不会退出。（以上是针对正常退出，调用System.exit则必定会退出）   

setDeamon(true)的唯一意义就是告诉JVM不需要等待它退出
守护线程在没有用户线程可服务时自动离开，在Java中比较特殊的线程是被称为守护（Daemon）线程的低级别线程。这个线程具有最低的优先级，用于为系统中的其它对象和线程提供服务。
我们所熟悉的Java垃圾回收线程就是一个典型的守护线程

[toTop](#jump) 

# sleep wait 区别
``sleep()``方法属于``Thread类``中的。
``wait()``方法属于``Object类``中的。

调用``sleep()``方法的过程中，**线程不会释放对象锁**。

调用``wait()``方法的时候，**线程会放弃对象锁**，进入等待此对象的等待锁定池，只有针对此对象调用notify()方法后本线程才进入对象锁定池准备

[toTop](#jump) 
