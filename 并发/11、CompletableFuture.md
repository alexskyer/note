#### 1.异步计算
- 所谓异步调用其实就是实现一个可无需等待被调用函数的返回值而让操作继续运行的方法。在 Java 语言中，简单的讲就是另启一个线程来完成调用中的部分计算，使调用继续运行或返回，而不需要等待计算结果。但调用者仍需要取线程的计算结果。
- JDK5新增了Future接口，用于描述一个异步计算的结果。虽然 Future 以及相关使用方法提供了异步执行任务的能力，但是对于结果的获取却是很不方便，只能通过阻塞或者轮询的方式得到任务的结果。阻塞的方式显然和我们的异步编程的初衷相违背，轮询的方式又会耗费无谓的 CPU 资源，而且也不能及时地得到计算结果。
```Java
ExecutorService executor = Executors.newFixedThreadPool(3);
 Future future = executor.submit(new Callable<String>() {

 @Override
 public String call() throws Exception {
 //do some thing
 Thread.sleep(100);
 return "i am ok";
 }
 });
 println(future.isDone());
 println(future.get());
```
使用Java中Future是用来实现异步计算，其计算结果需要通过 get方法获取，问题是在计算完成之前调用get是阻塞的，这造成了非常严格的使用限制，使异步计算毫无意义。

CompletableFuture除了实现Future接口外，还实现了CompletionStage接口。 CompletionStage是一种承诺。它承诺最终将完成计算，最棒的是CompletionStage是它提供了大量的方法，可以让你能附加方法，在其完成时对这些方法执行的回调，这样我们就可以以非阻塞的方式构建系统。
