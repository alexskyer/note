#### 1.Condition使用简介
任何一个java对象都天然继承于Object类，在线程间实现通信的往往会应用到Object的几个方法，比如wait()、wait(long timeout)、wait(long timeout, int nanos)与notify()、notifyAll()几个方法实现等待/通知机制，同样的， 在java Lock体系下依然会有同样的方法实现等待/通知机制。

从整体上来看Object的wait和notify/notify是与对象监视器配合完成线程间的等待/通知机制，而Condition与Lock配合完成等待通知机制，前者是java底层级别的，后者是语言级别的，具有更高的可控制性和扩展性。两者除了在使用方式上不同外，在功能特性上还是有很多的不同：
- Condition能够支持不响应中断，而通过使用Object方式不支持
- Condition能够支持多个等待队列（new 多个Condition对象），而Object方式只能支持一个
- Condition能够支持超时时间的设置，而Object不支持

Condition由ReentrantLock对象创建，并且可以同时创建多个，Condition接口在使用前必须先调用ReentrantLock的lock()方法获得锁，之后调用Condition接口的await()将释放锁，并且在该Condition上等待，直到有其他线程调用Condition的signal()方法唤醒线程，使用方式和wait()、notify()类似。

对比项|	Object监视器方法|	Condition
-|-|-
前置条件|	获取对象的锁|	调用Lock.lock获取锁，调用Lock.newCondition()获取Condition对象
调用方式|	直接调用，如：object.wait()|	直接调用，如：condition.await()
等待队列个数|	一个|	多个，使用多个condition实现
当前线程释放锁并进入等待状态|	支持|	支持
当前线程释放锁进入等待状态中不响应中断|	不支持|	支持
当前线程释放锁并进入超时等待状态|	支持|	支持
当前线程释放锁并进入等待状态到将来某个时间	|不支持|	支持
唤醒等待队列中的一个线程|	支持	|支持
唤醒等待队列中的全部线程	|支持	|支持
