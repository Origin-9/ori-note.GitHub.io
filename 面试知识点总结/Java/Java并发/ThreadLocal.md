## ThreadLocal是什么

通常情况下，我们创建的变量是可以被任何一个线程访问并修改的。

使用ThreadLocal创建的变量只能被当前线程访问，其他线程则无法访问和修改。

## 原理总结

![image-20200830162056569](C:\Users\Ori\AppData\Roaming\Typora\typora-user-images\image-20200830162056569.png)

- 每个Thread维护着一个ThreadLocalMap的引用
- ThreadLocalMap是ThreadLocal的内部类，用Entry来进行存储
- 调用ThreadLocal的set()方法时，实际上就是往ThreadLocalMap设置值，key是ThreadLocal对象，值是传递进来的对象
- 调用ThreadLocal的get()方法时，实际上就是往ThreadLocalMap获取值，key是ThreadLocal对象
- **ThreadLocal本身并不存储值**，它只是**作为一个key来让线程从ThreadLocalMap获取value**。

## 内存泄漏

`ThreadLocalMap` 中使用的 key 为 `ThreadLocal` 的弱引用,而 value 是强引用。所以，如果 `ThreadLocal` 没有被外部强引用的情况下，在垃圾回收的时候，key 会被清理掉，而 value 不会被清理掉。这样一来，`ThreadLocalMap` 中就会出现key为null的Entry。假如我们不做任何措施的话，value 永远无法被GC 回收，这个时候就可能会产生内存泄露。ThreadLocalMap实现中已经考虑了这种情况，在调用 `set()`、`get()`、`remove()` 方法的时候，会清理掉 key 为 null 的记录。使用完 `ThreadLocal`方法后 最好手动调用`remove()`方法