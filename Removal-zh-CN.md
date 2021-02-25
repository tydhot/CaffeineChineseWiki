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
    .evictionListener((Key key, Graph graph, RemovalCause cause) ->
        System.out.printf("Key %s was evicted (%s)%n", key, cause))
    .removalListener((Key key, Graph graph, RemovalCause cause) ->
        System.out.printf("Key %s was removed (%s)%n", key, cause))
    .build();
```

你可以为你的缓存通过`Caffeine.removalListener(RemovalListener)`方法定义一个移除监听器在一个元素被移除的时候进行相应的操作。这些操作是使用 [Executor][2] 异步执行的，其中默认的 Executor 实现是 [ForkJoinPool.commonPool()][3] 并且可以通过覆盖`Caffeine.executor(Executor)`方法自定义线程池的实现。

当移除之后的自定义操作必须要同步执行的时候，你需要使用 `Caffeine.evictionListener(RemovalListener)` 。这个监听器将在 `RemovalCause.wasEvicted()` 为 true 的时候被触发。为了移除操作能够明确生效， `Cache.asMap()` 提供了方法来执行原子操作。

记住任何在 `RemovalListener`中被抛出的异常将会被打印日志 (通过[Logger][1])并被吞食。

[1]: http://docs.oracle.com/javase/8/docs/api/java/util/logging/package-summary.html
[2]: http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executor.html
[3]: https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinPool.html