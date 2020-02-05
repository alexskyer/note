#### 1、简介
同步方法通过ACC_SYNCHRONIZED关键字隐式的对方法进行加锁。当线程要执行的方法被标注上ACC_SYNCHRONIZED时，需要先获得锁才能执行该方法。
同步代码块通过monitorenter和monitorexit执行来进行加锁。当线程执行到monitorenter的时候要先获得所锁，才能执行后面的方法。当线程执行到monitorexit的时候则要释放锁。
每个对象自身维护这一个被加锁次数的计数器，当计数器数字为0时表示可以被任意线程获得锁。当计数器不为0时，只有获得锁的线程才能再次获得锁。即可重入锁。

#### 2、使用
- 普通方法:同步代码执行前，需要获取当前实例对象的锁(对象锁).
```Java
public synchronized void  methodA(){
    //省略同步代码
}
```
- 静态方法:，同步代码执行前，需要获取当前类的锁(类锁)。
```Java
public static synchronized void  methodB(){
    //省略同步代码
}
```
- 修饰代码块，需要看括号里的锁类型
 - 对象锁

 ```java
 public void methodC(){
	//this可以替换成其它实例对象
    synchronized (this){
    	//省略同步代码
    }
}
 ```
 - 类锁

 ```Java
 public void methodD() {
    synchronized (Xxx.class) {
    	//省略同步代码
    }
}
 ```

 #### 3、Lock与synchronized区别
- 可重入锁：Synchronized和ReentrantLook都是可重入锁，锁的可重入性标明了锁是针对线程分配方式而不是针对方法。例如调用Synchronized方法A中可以调用Synchronized方法B，而不需要重新申请锁。

- 读写锁：按照数据库事务隔离特性的类比读写锁，在访问统一个资源（一个文件）的时候，使用读锁来保证多线程可以同步读取资源。ReadWriteLock是一个读写锁，通过readLock()获取读锁，通过writeLock()获取写锁。

- 可中断锁：可中断是指锁是可以被中断的，Synchronized内置锁是不可中断锁，ReentrantLock可以通过lockInterruptibly方法中断显性锁。例如线程B在等待等待线程A释放锁，但是线程B由于等待时间太久，可以主动中断等待锁。

- 公平锁：公平锁是指尽量以线程的等待时间先后顺序获取锁，等待时间最久的线程优先获取锁。synchronized隐性锁是非公平锁，它无法保证等待的线程获取锁的顺序，ReentrantLook可以自己控制是否公平锁。
