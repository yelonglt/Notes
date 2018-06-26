#Exception和Error

Exception和Error都是继承于Throwable，在Java中只有Throwable类型的实例才可以抛出（throw）或捕捉（catch），这是异常处理机制的基本组成类型。

###Exception

1. Exception是程序正常运行中，可以预料的意外情况。
2. Exception可分为可检查异常和不检查异常。可检查异常在源代码里必须显示的处理；不检查异常（运行时异常），类似NPE（NullPointException）、CCE（ClassCastException）等

###Error

1. Error是正常情况下是不可能出现的，出现一就导致程序处理费正常、不可恢复状态。因为也就没有必要捕获处理。
2. 如系统奔溃、虚拟机错误、内存空间不足（OutOfMemoryError）、方法调用栈溢出（StackOverflowError）等。

###Throw和throws有什么不同？

1. throw可以抛出任何异常
2. throws通常出现在函数头中，用标记改成员函数可能会抛出异常。
