###主要类的说明

1. Message：消息分为硬件产生的消息（按钮、触摸）和软件生成的消息。
2. MessageQueue：消息队列的主要功能是向消息池投递消息（MessageQueue.enqueueMessage）和取走消息池的消息（MessageQueue.next）
3. Handler：消息辅助类，主要功能是向消息池中发送各种消息事件（Handler.sendMessage）和处理相应消息事件（Handler.handleMessage）
4. 不断的循环执行（Looper.loop），按分发机制将消息分发给目标处理者。

典型事例

```
public class LooperThread extends Thread {

    private Handler mHandler;

    @Override
    public void run() {
        // 创建Handler对象之前必须新建一个Looper对象
        Looper.prepare();
        mHandler = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
            }
        };
        // 开启消息循环
        Looper.loop();
    }
}
```
###总结
1. 线程中可以创建多个Handler对象，但是一个线程只能对应一个Looper和一个MessageQueue。创建Handler对象的时候就与之产生关联，假如所在线程没有Looper对象，必须先调用Looper.prepare()。
2. Message是按照时间顺序放入消息队列的。
3. Handler.enqueueMessage()方法中的msg.target = this保证了谁发送消谁处理
4. 消息从消息队列取出的时候会被移出消息队列。
5. 主线程的Looper.loop()死循环是不是特别消耗CPU资源？其实不然，这里就涉及到Linux pipe/epoll机制，简单说就是在主线程的MessageQueue没有消息时，便阻塞在loop的queue.next()中的nativePollOnce()方法里，此时主线程会释放CPU资源进入休眠状态，直到下个消息到达或者有事务发生，通过往pipe管道写端写入数据来唤醒主线程工作（或者enqueueMessage中调用nativeWake()唤醒主线程工作）。这里采用的epoll机制，是一种IO多路复用机制，可以同时监控多个描述符，当某个描述符就绪(读或写就绪)，则立刻通知相应程序进行读或写操作，本质同步I/O，即读写是阻塞的。 所以说，主线程大多数时候都是处于休眠状态，并不会消耗大量CPU资源。
6. 所以在子线程中创建的Looper对象，当所有的事情处理完成了，需要手动调用quit方法终止循环，不然子线程就会一直处理等待状态，资源得不到释放。