<a id = "jump">[首页](/README.md)</a>

<!-- TOC -->

- [线程池](#线程池)
    - [四种线程池](#四种线程池)
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

