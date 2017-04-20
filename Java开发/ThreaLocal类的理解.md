###ThreadLocal作用
ThreadLocal的设计并不是解决资源共享的问题，而是用来提供线程内部的局部变量，这样每个线程自己管理自己的局部变量，别的线程操作的数据不会对本线程产生影响，互不影响。

###ThreadLocal的API

1. initialValue，初始化ThreadLocal变量值。
2. get()，获取ThreadLocal变量值。
3. set(Object value)，设置ThreadLocal变量值。
4. remove()，删除当前线程ThreadMap保存的Entry，目的是减少内存的占用。

###注意点
1. 一般使用ThreadLocal，官方建议我们定义为private static。为什么要定义成静态的呢？因为一个Thread维持一个ThreadLocalMap对象，而该Map对象的key又由提供该Value的ThreadLocal对象弱引用提供，就会有这种情况：如果ThreadLocal不设置为static，由于Thread的生命周期不可预知，这就导致了当系统gc时将会回收，而ThreadLocal对象被回收了，此时它对应key必定为null，这就导致了该key对应得value拿不出来了，而value之前被Thread所引用，所以就存在key为null、value存在强引用导致这个Entry回收不了，从而导致内存泄露。
2. ThreadLocal在线程使用完毕之后，我们应该手动调用remove方法，移除它内部的值，这样可以防止内存泄漏，当然还有设为static。

###使用
```
class Task implements Runnable {

    private static ThreadLocal<String> threadLocal = new ThreadLocal<String>() {
        @Override
        protected String initialValue() {
            return Thread.currentThread().getName();
        }
    };

    @Override
    public void run() {
        threadLocal.set("Hello World!");
        System.out.println("value: " + threadLocal.get());
        threadLocal.remove();
    }
}
```

###ThreadLocal在Android中的应用
在Android中，Looper类就是利用了ThreadLocal的特性，保证每个线程只有一个Looper对象。

```
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```

set的实现

1. 首先获取当前线程
2. 利用当前线程作为句柄获取一个ThreadLocalMap的对象
3. 如果上述ThreadLocalMap对象不为空，则设置值，否则创建这个ThreadLocalMap对象并设置值

```
public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
    
ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
    
void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
    
class Thread implements Runnable {
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */

    ThreadLocal.ThreadLocalMap threadLocals = null;
}
```
总结：实际上ThreadLocal的值是放入了当前线程的一个ThreadLocalMap实例中，所以只能在本线程中访问，其他线程无法访问。

###问题1
ThreadLocal只对当前线程可见，是不是说ThreadLocal的值只能被一个线程访问呢？使用InheritableThreadLocal可以实现多个线程访问ThreadLocal的值。使用InheritableThreadLocal可以将某个线程的ThreadLocal值在其子线程创建时传递过去。因为在线程创建过程中，有相关的处理逻辑。

```
//Thread.java
 private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc) {
        //code goes here
        if (parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
        /* Stash the specified stack size in case the VM cares */
        this.stackSize = stackSize;

        /* Set thread ID */
        tid = nextThreadID();
}
```
上面代码就是在线程创建的时候，复制父线程的inheritableThreadLocals的数据。