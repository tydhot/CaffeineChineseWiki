```java
FakeTicker ticker = new FakeTicker(); // Guava的测试库
Cache<Key, Graph> cache = Caffeine.newBuilder()
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .executor(Runnable::run)
    .ticker(ticker::read)
    .maximumSize(10)
    .build();

cache.put(key, graph);
ticker.advance(30, TimeUnit.MINUTES)
assertThat(cache.getIfPresent(key), is(nullValue()));
```

当测试基于时间的驱逐策略的时候，不需要等着现实时间的推进。可以使用`Ticker`接口和`Caffeine.ticker(Ticker)`方法在你的缓存构造器中去定义一个自定义的时间源而不是等待系统时钟的转动。 为了这个目的，[Guava的测试库][testlib]提供了`FakeTicker` 。过期的元素将在周期性的维护中被清除，所以在测试基于驱逐策略的时候也可以直接使用 `Cache.cleanUp()`方法来立即触发一次维护操作。 
  

Caffeine 将通过`Executor`来执行周期维护，移除通知和异步生成。这将对于调用者来说使得响应耗时变得更加可靠，并且线程池的默认实现是 [ForkJoinPool.commonPool()][fork-join]。可以通过在缓存构造器中使用`Caffeine.executor(Executor)` 方法来指定一个直接（同线程）的executor 而不是等待异步任务执行完毕。      

我们推荐使用[Awaitility][await]来进行多线程下的测试。   

[testlib]: http://www.javadoc.io/doc/com.google.guava/guava-testlib
[fork-join]: https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinPool.html
[await]: https://github.com/jayway/awaitility