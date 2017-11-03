
#Linux进程间通讯(IPC)方式

1. 管道：在创建时分配一个page大小的内存，缓存区大小比较有限。
2. 消息队列：信息复制两次，额外的CPU消耗；不合适频繁或信息量大的通讯。
3. 共享内存：无需复制，共享缓冲区直接附加到进程虚拟地址空间，速度快；但进程间的同步问题操作系统无法实现，必须各进程利用同步工具解决。
4. 套接字：作为更通用的接口，传输效率低。主要用于不同机器或跨网络的通信。
5. 信号量：作为常用的一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。
6. 信号：不适用于信息交换，更适用于进程中断控制，比如非法内存访问，杀死某个进程等。

#为什么基于Linux系统的Android，使用Binder进行进程间通讯呢？
从五个角度来展开对Binder的分析

1. 从性能的角度。
   * 数据拷贝次数：Binder数据拷贝只需要一次，而管道、消息队列、Socket都需要2次，但共享内存方式一次内存拷贝都不需要。
   * 总结，从性能角度看，Binder性能仅次于共享内存。
2. 从稳定性的角度。
   * Binder是基于C/S架构的，简单解释下C/S架构，是指客户端和服务端组成的架构，客户端有什么需求直接发送给服务端去完成，架构清晰明朗，客户端和服务端相对独立，稳定性好。
   * 共享内存实现方式复杂，需要充分考虑到访问临界资源的并发同步问题，否者可能会出现死锁等问题。
   * 总结，从稳定角度看，Binder架构优于共享内存。
3. 从安全的角度。
   * 传统的Linux进程间通信的接收方无法获得对方进程可靠的UID/PID，从而无法鉴别对方身份。
   * Android作为开放的体系，手机安全显得额外重要。Android为每个安装的应用程序分配了自己的UID，故进程的UID是鉴别进程身份的重要标志。
   * 总结，从安全的角度看，Binder架构更加安全。
4. 从语言层面的角度。
   * Linux是基于C语言（面向过程的语言）；Android是基于Java语言（面向对象的语言），而对于Binder恰恰也符合面向对象的思想，将进程间通信转化为通过对某个Binder对象的引用调用该对象的方法，而其独特之处是在于Binder对象是一个可以跨进程引用的对象，它的实体位于一个进程中，而它的引用却遍布于系统的各个进程中。Binder模糊了进程边界，淡化了进程间通讯过程。
   * Binder是为Android这类系统而生，在Android系统中依然采用了大量的Linux现有的IPC机制，根据每类IPC的原理特性，因时制宜。例如，在Android系统中的Zygote进程的IPC采用的是Socket（套接字）机制；Android中的Kill Process采用的signal（信号）机制。
   * Binder更多则用在system_server进程与上层App层的IPC交互。
   * 总结，从语言层面的角度看，Binder架构更加合适
5. 从公司战略的角度。
   * Binder是基于开源项目OpenBinder实现的。OpenBinder是一个开源的系统IPC机制。

#简述Android Binder架构
> Binder在Android系统中江湖地位非常高。在Zygote孵化出system\_server进程后，在system\_server进程中出现初始化支持整个Android Framework的各种各样的Service，而这些Service从大的方向来划分，分为Java层Framework和Native Framework层（C++）的Service，几乎都是基于Binder IPC机制。

1. Java Framework层：作为Server端继承（或间接继承）于Binder类，Client端继承（或间接继承）于BinderProxy类。例如ActivityManagerService这个服务作为Server端，间接继承Binder类，而相应的ActivityManager作为Client端，间接继承于BinderProxy类。当然还有PackageManagerService、WindowManagerService等很多系统服务都是采用C/S架构。
2. Native Framework层：这是C++层，作为Server端继承（或间接继承）于BBinder类，Client端继承（或间接继承）于BpBinder。例如MediaPlayService（Server端）和MediaPlay（Client端）


