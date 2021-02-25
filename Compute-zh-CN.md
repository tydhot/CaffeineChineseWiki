```java
Cache<Key, Graph> graphs = Caffeine.newBuilder()
    .evictionListener((Key key, Graph graph, RemovalCause cause) -> {
      // atomically intercept the entry's eviction
    }).build();

graphs.asMap().compute(key, (k, v) -> {
  Graph graph = createExpensiveGraph(key);
  ... // update a secondary store
  return graph;
});
```

在复杂的工作流中，当外部资源对key的操作变更顺序有要求的时候，Caffeine 提供了实现的扩展点。对于手动操作，[Map][concurrent Map] 的 compute 方法提供了执行原子创建、原子更新或原子删除条目操作的能力。当一个元素被自动移除的时候，驱逐监听器可以根据映射关系计算扩展自定义操作。这意味着在缓存中，当一个key的写入操作在完成之前，后续其他写操作都是阻塞的，同时在这段时间内，尝试获取这个key对应的缓存元素的时候获取到的也将都是旧值。

### 可能的使用场景

#### 写模式

计算可以作为实现 write-through 和 write-back 两种模式的缓存的方式。

在一个 *write-through* 模式下的缓存里，操作都将是同步的并且缓存的变更只有在Writer中更新成功才会生效。这避免了缓存更新与外部资源更新都是独立的原子操作的时候的资源竞争。

在一个 *write-back* 模式下的缓存里，在缓存更新之后将会异步执行外部数据源的更新。这会增加数据不一致性的风险，比如在外部数据源更新失败的情况下缓存里的数据将会变得非法。这种方式在一定时间后延迟写，控制写的速率和批量写的场景下将会十分有效。 

write-back模式下可以实现以下几种特性：  
- 批量和合并操作    
- 延迟操作指定的时间窗口   
- 如果批处理的数据量超过阈值大小，那么将在定期刷新之前提前执行批处理操作   
- 如果外部资源操作还没有刷新，则从write-behind缓存当中加载数据    
- 根据外部资源的特性控制重试，速率和并发度    

可以通过查看这个 [write-behind-rxjava][write-behind-rxjava] 使用了[RxJava][rxjava]的简单demo来进行参考。

#### 分层

*layered cache* 多级缓存支持从一个数据系统所支持的外部缓存中加载和写入。这允许构建一个数据量更小速度也更快的缓存将数据回落到一个数据量更大但是速度较慢的大缓存中。典型的多级缓存包含堆外缓存，基于文件的缓存和远程的缓存。 

*victim cache* 是多级缓存的一种变种，它将把被驱逐的数据写入到二级缓存当中去。 `delete(K, V, RemovalCause)`方法支持查看被移除缓存的具体信息和移除原因。  

#### 同步监听器

一个 *synchronous listener* 同步监听器将会按照指定key的操作顺序接收到相应的事件。这个监听器可以用来阻塞该操作也可以把这次操作事件投入到队列中被异步处理。这种类型的监听器可以用来作为备份或者构造一个分布式缓存。    

[concurrent-map]: https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentMap.html
[rxjava]: https://github.com/ReactiveX/RxJava
[write-behind-rxjava]: https://github.com/ben-manes/caffeine/tree/master/examples/write-behind-rxjava