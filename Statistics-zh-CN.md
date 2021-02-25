```java
Cache<Key, Graph> graphs = Caffeine.newBuilder()
    .maximumSize(10_000)
    .recordStats()
    .build();
```

通过使用`Caffeine.recordStats()`方法可以打开数据收集功能。`Cache.stats()`方法将会返回一个`CacheStats`对象，其将会含有一些统计指标，比如：   
 * `hitRate():` 查询缓存的命中率   
 * `evictionCount():` 被驱逐的缓存数量    
 * `averageLoadPenalty():` 新值被载入的平均耗时    

这些统计指标在缓存的调优中十分重要，我们强烈的建议你在性能指标严格的程序中去留意这些统计指标。   

这些缓存统计指标可以被基于push/pull模式的报告系统进行集成。基于pull模式的系统可以通过调用`Cache.stats()` 方法获取当前缓存最新的统计快照。一个基于push的系统可以通过自定义一个`StatsCounter`对象达到在缓存操作发生时自动推送更新指标的目的。   

如果使用 [Dropwizard Metrics][metrics] 的话建议查看 [metrics-caffeine][metrics-caffeine] 。

如果使用[Prometheus][prometheus]的话可以尝试 [simpleclient-caffeine][simpleclient-caffeine]。    

如果实在难以选择的话可以尝试通过[Micrometer][micrometer]来进行整合。   

[metrics]: http://metrics.dropwizard.io
[metrics-caffeine]: https://github.com/dropwizard/metrics/tree/release/4.1.x/metrics-caffeine
[simpleclient-caffeine]: https://github.com/prometheus/client_java#caches
[prometheus]: https://prometheus.io
[micrometer]: http://micrometer.io