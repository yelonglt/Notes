###概述
AMS是系统的引导服务，应用进程的启动、切换和调度、四大组件的启动和管理都需要AMS的支持。从这里可以看出AMS的功能会十分繁多，当然它并不是一个类承担这个重责，它有一些关联类。

###AMS的启动流程
AMS的启动是在SystemServer进程中启动的。从SystemServer的main方法开始分析。

1. 查看SystemServer.main()，直接调用SystemServer.run()方法。