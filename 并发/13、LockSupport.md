#### 1.常用线程唤醒
- 使用Object中的wait()方法让线程等待，使用Object中的notify()方法唤醒线程
- 使用juc包中Condition的await()方法让线程等待，使用signal()方法唤醒线程
- LockSupport

#### 2.使用Object类中的方法实现线程等待和唤醒
- wait()/notify()/notifyAll()方法都必须放在同步代码（必须在synchronized内部执行）中执行，需要先获取锁
- 线程唤醒的方法（notify、notifyAll）需要在等待的方法（wait）之后执行，等待中的线程才可能会被唤醒，否则无法唤醒  

缺陷：
- 由上面的例子可知,wait和notify都是Object中的方法,在调用这两个方法前必须先获得锁对象，这限制了其使用场合:只能在同步代码块中。
- 另一个缺点可能上面的例子不太明显，当对象的等待队列中有多个线程时，notify只能随机选择一个线程唤醒，无法唤醒指定的线程

#### 3.使用Condition实现线程的等待和唤醒
```java
public class ConditionDemo {
    public static void main(String[] args) {
        ReentrantLock reentrantLock = new ReentrantLock();
        Condition condition = reentrantLock.newCondition();

        new Thread(() -> {
            reentrantLock.lock();
            System.out.println(Thread.currentThread().getName()+"get lock successfully");
            System.out.println(Thread.currentThread().getName()+"waiting signal");
            try {
                condition.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+" get signal successfully");
            reentrantLock.unlock();

        },"first thread ").start();

        new Thread(() -> {
            reentrantLock.lock();
            System.out.println(Thread.currentThread().getName()+"get lock successfully");

            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println(Thread.currentThread().getName()+"send signal");
            condition.signalAll();
            System.out.println(Thread.currentThread().getName()+" get signal successfully");
            reentrantLock.unlock();

        },"second thread ").start();
    }
}
```
- 使用Condtion中的线程等待和唤醒方法之前，需要先获取锁。否者会报 IllegalMonitorStateException异常
- signal()方法先于await()方法之前调用，线程无法被唤醒

#### 4.LockSupport
- LockSupport是一个线程阻塞工具类，所有的方法都是静态方法，可以让线程在任意位置阻塞，当然阻塞之后肯定得有唤醒的方法,park和unpark可以实现类似wait和notify的功能，但是并不和wait和notify交叉，也就是说unpark不会对wait起作用，notify也不会对park起作用。
- park和unpark的使用不会出现死锁的情况
- blocker的作用是在dump线程的时候看到阻塞对象的信息

##### 4.1 常用方法
```java
public static void park(Object blocker); // 暂停当前线程
public static void parkNanos(Object blocker, long nanos); // 暂停当前线程，不过有超时时间的限制
public static void parkUntil(Object blocker, long deadline); // 暂停当前线程，直到某个时间
public static void park(); // 无期限暂停当前线程
public static void parkNanos(long nanos); // 暂停当前线程，不过有超时时间的限制
public static void parkUntil(long deadline); // 暂停当前线程，直到某个时间
public static void unpark(Thread thread); // 恢复当前线程
public static Object getBlocker(Thread t);
```
主要有两类方法：park和unpark
##### 4.2 示例
```java
public class LockSupportTest {

    public static void main(String[] args) {
        Thread parkThread = new Thread(new ParkThread());
        parkThread.start();
        System.out.println("开始线程唤醒");
        LockSupport.unpark(parkThread);
        System.out.println("结束线程唤醒");

    }

    static class ParkThread implements Runnable{

        @Override
        public void run() {
            System.out.println("开始线程阻塞");
            LockSupport.park();
            System.out.println("结束线程阻塞");
        }
    }
}
```
LockSupport.park();可以用来阻塞当前线程,park是停车的意思，把运行的线程比作行驶的车辆，线程阻塞则相当于汽车停车，相当直观。该方法还有个变体LockSupport.park(Object blocker),指定线程阻塞的对象blocker，该对象主要用来排查问题。方法LockSupport.unpark(Thread thread)用来唤醒线程，因为需要线程作参数，所以可以指定线程进行唤醒

##### 4.3 原理
- 当调用park()方法时，会将_counter置为0，同时判断前值，小于1说明前面被unpark过,则直接退出，否则将使该线程阻塞。
- 当调用unpark()方法时，会将_counter置为1，同时判断前值，小于1会进行线程唤醒，否则直接退出。形象的理解，线程阻塞需要消耗凭证(permit)，这个凭证最多只有1个。当调用park方法时，如果有凭证，则会直接消耗掉这个凭证然后正常退出；但是如果没有凭证，就必须阻塞等待凭证可用；而unpark则相反，它会增加一个凭证，但凭证最多只能有1个


-|Object|Condtion|LockSupport
-|-|-|-
前置条件|	需要在synchronized中运行|	需要先获取Lock的锁|	无
无限等待|	支持|	支持|	支持
超时等待|	支持|	支持|	支持
等待到将来某个时间返回|	不支持|	支持|	支持
等待状态中释放锁|	会释放|	会释放|	不会释放
唤醒方法先于等待方法执行，能否唤醒线程|	否|	否|	可以
是否能响应线程中断|	是	|是|	是
线程中断是否会清除中断标志|	是|	是|	否
是否支持等待状态中不响应中断|	不支持	|支持|	不支持
