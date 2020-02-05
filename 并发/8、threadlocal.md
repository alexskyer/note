#### 1、ThreadLocal
- 它能让线程拥有了自己内部独享的变量
- 每一个线程可以通过get、set方法去进行操作
- 可以覆盖initialValue方法指定线程独享的值
- 通常会用来修饰类里private static final的属性，为线程设置一些状态信息，例如user ID或者Transaction ID
- 每一个线程都有一个指向threadLocal实例的弱引用，只要线程一直存活或者该threadLocal实例能被访问到，都不会被垃圾回收清理掉
#### 2、ThreadLocalMap
- ThreadLocalMap是一个自定义的hash map，专门用来保存线程的thread local变量
- 16它的操作仅限于ThreadLocal类中，不对外暴露
- 这个类被用在Thread类的私有变量threadLocals和inheritableThreadLocals上
- 为了能够保存大量且存活时间较长的threadLocal实例，hash table entries采用了WeakReferences作为key的类型
- 一旦hash table运行空间不足时，key为null的entry就会被清理掉
```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```
