术语:
* **驱逐** 缓存元素因为策略被移除
* **失效** 缓存元素被手动移除
* **移除** 由于驱逐或者失效而最终导致的结果

### 显式移除
在任何时候，你都可以手动去让某个缓存元素失效而不是只能等待其因为策略而被驱逐。

```java
// 失效key
cache.invalidate(key)
// 批量失效key
cache.invalidateAll(keys)
// 失效所有的key
cache.invalidateAll()
```

### 移除监听器
```java
Cache<Key, Graph> graphs = Caffeine.newBuilder()
    .removalListener((Key key, Graph graph, RemovalCause cause) ->
        System.out.printf("Key %s was removed (%s)%n", key, cause))
    .build();
```

你可以为你的缓存通过`Caffeine.removalListener(RemovalListener)`方法定义一个移除监听器在一个元素被移除的时候进行相应的操作。`RemovalListener` 可以获得key，缓存元素value和被移除原因`RemovalCause`。

移除监听器监听器的操作将会异步执行在一个[Executor][2]上。默认的线程池实现是[ForkJoinPool.commonPool()][3]当然也可以通过覆盖`Caffeine.executor(Executor)`方法自定义线程池的实现。当移除之后的自定义操作必须要同步执行的时候，你需要使用 [CacheWriter](Writer)作为代替。

记住任何在 `RemovalListener`中被抛出的异常将会被打印日志 (通过[Logger][1])并被吞食。

[1]: http://docs.oracle.com/javase/8/docs/api/java/util/logging/package-summary.html
[2]: http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executor.html
[3]: https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinPool.html