```java
LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
    .maximumSize(10_000)
    .refreshAfterWrite(1, TimeUnit.MINUTES)
    .build(key -> createExpensiveGraph(key));
```

刷新和驱逐并不相同。可以通过`LoadingCache.refresh(K)`方法，异步为key对应的缓存元素刷新一个新的值。与驱逐不同的是，在刷新的时候如果查询缓存元素，其旧值将仍被返回，直到该元素的刷新完毕后结束后才会返回刷新后的新值。 

与 `expireAfterWrite`相反，`refreshAfterWrite` 将会使在写操作之后的一段时间后允许key对应的缓存元素进行刷新，但是只有在这个key被真正查询到的时候才会正式进行刷新操作。所以打个比方，你可以在同一个缓存中同时用到  `refreshAfterWrite`和`expireAfterWrite` ，这样缓存元素的在被允许刷新的时候不会直接刷新使得过期时间被盲目重置。当一个元素在其被允许刷新但是没有被主动查询的时候，这个元素也会被视为过期。

一个`CacheLoader`可以通过覆盖重写 `CacheLoader.reload(K, V)` 方法使得在刷新中可以将旧值也参与到更新的过程中去，这也使得刷新操作显得更加智能。 

更新操作将会异步执行在一个[Executor][2]上。默认的线程池实现是[ForkJoinPool.commonPool()][3]当然也可以通过覆盖`Caffeine.executor(Executor)`方法自定义线程池的实现。

在刷新的过程中，如果抛出任何异常，都会使旧值被保留，并且异常将会被打印日志 (通过 [System.Logger][1] )并被吞食。

[1]: http://docs.oracle.com/javase/8/docs/api/java/util/logging/package-summary.html
[2]: http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executor.html
[3]: https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinPool.html