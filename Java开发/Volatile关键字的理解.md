###原子性
原子性：即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。

###可见性
可见性：即当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

###有序性
有序性：即程序执行的顺序按照代码的先后顺序执行。

执行重排序：一般来说，处理器为了提高程序运行效率，可能会对输入代码进行优化，它不保证程序中各个语句的执行先后顺序同代码中的顺序一致，但是它会保证程序最终执行结果和代码顺序执行的结果是一致的。

###关键字Volatile
对于可见性，Java提供了volatile关键字来保证可见性。

当一个共享变量被volatile修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值。

而普通的共享变量不能保证可见性，因为普通共享变量被修改之后，什么时候被写入主存是不确定的，当其他线程去读取时，此时内存中可能还是原来的旧值，因此无法保证可见性。

另外，通过synchronized和Lock也能够保证可见性，synchronized和Lock能保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前会将对变量的修改刷新到主存当中。因此可以保证可见性。

1. Volatile保证可见性。一旦一个共享变量(类的成员变量、累的静态成员变量)被Volatile修饰后具备两层语义：1）保证在不同线程对这个变量的操作是可见性。2）禁止进行指令的重排序
2. Volatile不能确保原子性
3. Volatile保证有序性

```
public class Test {

    private volatile int inc = 0;

    private void increase() {
        //自增操作是不具备原子性的，它包括读取变量的原始值、进行加1操作、写入工作内存。
        inc++;
    }

    public static void main(String[] args) {
        final Test test = new Test();
        for (int i = 0; i < 10; i++) {
            new Thread() {
                public void run() {
                    for (int j = 0; j < 1000; j++)
                        test.increase();
                }

                ;
            }.start();
        }

        while (Thread.activeCount() > 1)  //保证前面的线程都执行完
            Thread.yield();
        System.out.println(test.inc);
    }
}
```

###Volatile使用场景
synchronized关键字是防止多个线程同时执行一段代码，那么就会很影响程序执行效率，而volatile关键字在某些情况下性能要优于synchronized，但是要注意volatile关键字是无法替代synchronized关键字的，因为volatile关键字无法保证操作的原子性。通常来说，使用volatile必须具备以下2个条件：
1. 对变量的写操作不依赖于当前值
2. 该变量没有包含在具有其他变量的不变式中

使用场景之状态标记量

```
    volatile boolean flag = false;
    //线程1
    while(!flag){
        //doSomething();
    }
    //线程2
    public void setFlag() {
        flag = true;
    }
```

使用场景之单例模式double check

```
class Singleton {
    private volatile static Singleton instance = null;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    /**
                     * 不是原子操作做了三件事
                     * 1.给instance分配内存空间
                     * 2.调用Singleton的构造函数来初始化成员变量
                     * 3.将instance对象指向分配的内存空间(执行完这步instance就为非null了)
                     */
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```