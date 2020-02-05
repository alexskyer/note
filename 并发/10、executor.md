#### 1.Executors框架
***Excecutor框架主要包含3部分的内容：***
- 任务相关的：包含被执行的任务要实现的接口：Runnable接口或Callable接口
- 任务的执行相关的：包含任务执行机制的核心接口Executor，以及继承自Executor的ExecutorService接口。Executor框架中有两个关键的类实现了ExecutorService接口（ThreadPoolExecutor和ScheduleThreadPoolExecutor）
- 异步计算结果相关的：包含接口Future和实现Future接口的FutureTask类

##### 1.1 Executor接口
Executor接口中定义了方法execute(Runable able)接口，该方法接受一个Runable实例，他来执行一个任务，任务即实现一个Runable接口的类
##### 1.2 ExecutorService接口
ExecutorService继承于Executor接口，他提供了更为丰富的线程实现方法，比如ExecutorService提供关闭自己的方法，以及为跟踪一个或多个异步任务执行状况而生成Future的方法。
ExecutorService有三种状态：运行、关闭、终止。创建后便进入运行状态，当调用了shutdown()方法时，便进入了关闭状态，此时意味着ExecutorService不再接受新的任务，但是他还是会执行已经提交的任务，当所有已经提交了的任务执行完后，便达到终止状态。如果不调用shutdown方法，ExecutorService方法会一直运行下去，系统一般不会主动关闭。
##### 1.3 ThreadPoolExecutor类
线程池类，实现了ExecutorService接口中所有方法
##### 1.4 ScheduleThreadPoolExecutor定时器
ScheduleThreadPoolExecutor继承自ScheduleThreadPoolExecutor，他主要用来延迟执行任务，或者定时执行任务。功能和Timer类似，但是ScheduleThreadPoolExecutor更强大、更灵活一些。Timer后台是单个线程，而ScheduleThreadPoolExecutor可以在创建的时候指定多个线程
##### 1.5 Executors类
Executors类，提供了一系列工厂方法用于创建线程池，返回的线程池都实现了ExecutorService接口。常用的方法有：
- newSingleThreadExecutor  
创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。内部使用了无限容量的LinkedBlockingQueue阻塞队列来缓存任务，任务如果比较多，单线程如果处理不过来，会导致队列堆满，引发OOM。
```java
public static ExecutorService newSingleThreadExecutor()
public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory)
```
- newFixedThreadPool  
创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，在提交新任务，任务将会进入等待队列中等待。如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。内部使用了无限容量的LinkedBlockingQueue阻塞队列来缓存任务，任务如果比较多，如果处理不过来，会导致队列堆满，引发OOM。
```java
public static ExecutorService newFixedThreadPool(int nThreads)
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory)
```  
- newCachedThreadPool  
创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，
那么就会回收部分空闲（60秒处于等待任务到来）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池的最大值是Integer的最大值(2^31-1)。内部使用了SynchronousQueue同步队列来缓存任务，此队列的特性是放入任务时必须要有对应的线程获取任务，任务才可以放入成功。如果处理的任务比较耗时，任务来的速度也比较快，会创建太多的线程引发OOM。
```java
public static ExecutorService newCachedThreadPool()
public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory)
```
- newScheduledThreadPool  
创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。
```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize, ThreadFactory threadFactory)
```

##### 1.6 Future、Callable接口
- Future接口定义了操作异步异步任务执行一些方法，如获取异步任务的执行结果、取消任务的执行、判断任务是否被取消、判断任务执行是否完毕等。
- Callable接口中定义了需要有返回的任务需要实现的方法  

比如主线程让一个子线程去执行任务，子线程可能比较耗时，启动子线程开始执行任务后，主线程就去做其他事情了，过了一会才去获取子任务的执行结果
- 获取异步任务执行结果
```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(1);
        Future<Integer> result = executorService.submit(() -> {
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName()+",start!");
            TimeUnit.SECONDS.sleep(5);
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName()+",end!");
            return 10;
        });
        System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName());
        System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + ",结果：" + result.get());
    }
```
代码中创建了一个线程池，调用线程池的submit方法执行任务，submit参数为Callable接口：表示需要执行的任务有返回值，submit方法返回一个Future对象，Future相当于一个凭证，可以在任意时间拿着这个凭证去获取对应任务的执行结果（调用其get方法），***代码中调用了result.get()方法之后，此方法会阻塞当前线程直到任务执行结束***
- 超时获取异步任务执行结果
可能任务执行比较耗时，比如耗时1分钟，我最多只能等待10秒，如果10秒还没返回，我就去做其他事情了。
```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(1);
        Future<Integer> result = executorService.submit(() -> {
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName()+",start!");
            TimeUnit.SECONDS.sleep(5);
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName()+",end!");
            return 10;
        });
        System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName());
        try {
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + ",结果：" + result.get(3,TimeUnit.SECONDS));
        } catch (TimeoutException e) {
            e.printStackTrace();
        }
        executorService.shutdown();
    }
```
任务执行中休眠了5秒，get方法获取执行结果，超时时间是3秒，3秒还未获取到结果，get触发了TimeoutException异常，当前线程从阻塞状态苏醒了
##### 1.7 FutureTask类
FutureTask除了实现Future接口，还实现了Runnable接口，因此FutureTask可以交给Executor执行，也可以交给线程执行执行（Thread有个Runnable的构造方法），FutureTask表示带返回值结果的任务
```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<Integer> futureTask = new FutureTask<Integer>(()->{
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName()+",start!");
            TimeUnit.SECONDS.sleep(5);
            System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName()+",end!");
            return 10;
        });
        System.out.println(System.currentTimeMillis()+","+Thread.currentThread().getName());
        new Thread(futureTask).start();
        System.out.println(System.currentTimeMillis()+","+Thread.currentThread().getName());
        System.out.println(System.currentTimeMillis()+","+Thread.currentThread().getName()+",结果:"+futureTask.get());
    }
```

#### 2.CompletionService接口
CompletionService相当于一个执行任务的服务，通过submit丢任务给这个服务，服务内部去执行任务，可以通过服务提供的一些方法获取服务中已经完成的任务,通过submit向内部提交任意多个任务，通过take方法可以获取已经执行完成的任务，如果获取不到将等待
常用方法：
```java
Future<V> submit(Callable<V> task);
```
用于向服务中提交有返回结果的任务，并返回Future对象
```java
Future<V> submit(Runnable task, V result);
```
用户向服务中提交有返回值的任务去执行，并返回Future对象
```java
Future<V> take() throws InterruptedException;
```
从服务中返回并移除一个已经完成的任务，如果获取不到，会一致阻塞到有返回值为止。此方法会响应线程中断。
```java
Future<V> poll();
```
从服务中返回并移除一个已经完成的任务，如果内部没有已经完成的任务，则返回空，此方法会立即响应。
```java
Future<V> poll(long timeout, TimeUnit unit) throws InterruptedException;
```
尝试在指定的时间内从服务中返回并移除一个已经完成的任务，等待的时间超时还是没有获取到已完成的任务，则返回空。此方法会响应线程中断
#### 3.ExecutorCompletionService类
ExecutorCompletionService类是CompletionService接口的具体实现，ExecutorCompletionService创建的时候会传入一个线程池，调用submit方法传入需要执行的任务，任务由内部的线程池来处理；ExecutorCompletionService内部有个阻塞队列，任意一个任务完成之后，会将任务的执行结果（Future类型）放入阻塞队列中，然后其他线程可以调用它take、poll方法从这个阻塞队列中获取一个已经完成的任务，获取任务返回结果的顺序和任务执行完成的先后顺序一致，所以最先完成的任务会先返回  
使用方法：
```java
public ExecutorCompletionService(Executor executor) {
        if (executor == null)
            throw new NullPointerException();
        this.executor = executor;
        this.aes = (executor instanceof AbstractExecutorService) ?
            (AbstractExecutorService) executor : null;
        this.completionQueue = new LinkedBlockingQueue<Future<V>>();
    }
```
构造方法需要传入一个Executor对象，这个对象表示任务执行器，所有传入的任务会被这个执行器执行。

completionQueue是用来存储任务结果的阻塞队列，默认用采用的是LinkedBlockingQueue，也支持开发自己设置。通过submit传入需要执行的任务，任务执行完成之后，会放入completionQueue中，有兴趣的可以看一下原码，还是很好理解的
