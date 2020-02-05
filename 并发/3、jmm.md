#### 1.概念
定义一套内存模型规范的，屏蔽了各种硬件和操作系统的访问差异的，保证了Java程序在各种平台下对内存的访问都能保证效果一致的机制及规范，主要是解决多线程的原子性、可见性、有序性
#### 2.原子性
原子性是指操作是不可分的，要么全部一起执行，要么不执行。在java中，其表现在对于共享变量的某些操作，是不可分的，必须连续的完成  
实现方法：锁机制、无锁CAS机制
#### 3.可见性
可见性是值一个线程对共享变量的修改，对于另一个线程来说是否是可以看到的  
实现方法：volatile、synchronized、锁、final
#### 4.有序性
有序性指的是程序按照代码的先后顺序执行  
- happen-before原则  
单线程happen-before原则：在同一个线程中，书写在前面的操作happen-before后面的操作。  
锁的happen-before原则：同一个锁的unlock操作happen-before此锁的lock操作。  
volatile的happen-before原则：对一个volatile变量的写操作happen-before对此变量的任意操作(当然也包括写操作了)。  
happen-before的传递性原则：如果A操作 happen-before B操作，B操作happen-before C操作，那么A操作happen-before C操作。  
线程启动的happen-before原则：同一个线程的start方法happen-before此线程的其它方法。  
线程中断的happen-before原则：对线程interrupt方法的调用happen-before被中断线程的检测到中断发送的代码。  
线程终结的happen-before原则：线程中的所有操作都happen-before线程的终止检测。  
对象创建的happen-before原则：一个对象的初始化完成先于他的finalize方法调用。  
