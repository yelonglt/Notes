###ThreadPoolExecutor
构造函数

```
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             threadFactory, defaultHandler);
    }
```

1. corePoolSize 表示核心线程数，在不设置allowCoreThreadTimeOut为true的情况下，核心线程就算没事做也不会被销毁。
2. maximumPoolSize 最大线程数
3. keepAliveTime 超时时长，非核心线程（核心线程）处于闲置超过这个时长就会被销毁。
4. TimeUnit unit 时间单位
5. BlockingQueue<Runnable> workQueue 缓存任务队列

###可缓存线程池newCachedThreadPool

```
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

###定长线程池FixedThreadPool

```
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

###单线程的线程池SingleThreadExecutor

```
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```

###延时或者周期执行的线程池

```
public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE,
              DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
              new DelayedWorkQueue());
    }
```

###ExecutorService执行线程的方法

1. execute方法，执行一个任务不需要得到返回结果直接用execute()会提升很多性能。这个方法会抛出异常
2. submit方法，执行一个任务需要得到返回结果。这个方法不会抛出异常，除非你调用Future.get()。
3. ScheduledExecutorService.schedule()，执行延时或者周期性任务。
