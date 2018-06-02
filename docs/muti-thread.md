<a id = "jump">[首页](/README.md)</a>

<!-- TOC -->

- [线程池](#线程池)
    - [四种线程池](#四种线程池)

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