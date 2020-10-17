```java
LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
  .writer(new CacheWriter<Key, Graph>() {
    @Override public void write(Key key, Graph graph) {
      // 持久化或者次级缓存
    }
    @Override public void delete(Key key, Graph graph, RemovalCause cause) {
      // 从持久化或者次级缓存中删除
    }
  })
  .build(key -> createExpensiveGraph(key));
```

`CacheWriter`给缓存提供了充当底层资源的门面的能力，当其与`CacheLoader`一起使用的时候，所有的读和写操作都可以通过缓存向下传播。Writers提供了原子性的操作，包括从外部资源同步的场景。这意味着在缓存中，当一个key的写入操作在完成之前，后续其他写操作都是阻塞的，同时在这段时间内，尝试获取这个key对应的缓存元素的时候获取到的也将都是旧值。如果写入失败那么之前的旧值将会被保留同时异常将会被传播给调用者。 

`CacheWriter`将会在缓存元素被创建，更新或者移除的时候被触发。但是当一个映射被加载(比如`LoadingCache.get`)，重载 (比如`LoadingCache.refresh`)，或者生成 (比如 `Map.computeIfPresent`) 的时候将不会被触发。

`CacheWriter` 将不能与 `weak keys` 或者`AsyncLoadingCache`在一起使用。

### 可能的使用场景
在复杂的工作流中，当外部资源对key的操作变更顺序有要求的时候，`CacheWriter`可以作为其的一个扩展点。Caffeine可以支持以下几种场景，但是并不是原生支持。

#### 写模式

一个`CacheWriter`可以用来实现write-through和write-back两种模式的缓存。

在一个*write-through*模式下的缓存里，操作都将是同步的并且缓存的变更只有在Writer中更新成功才会生效。这避免了缓存更新与外部资源更新都是独立的原子操作的时候的资源竞争。

在一个*write-back*模式下的缓存里，在缓存更新之后将会异步执行外部数据源的更新。这会增加数据不一致性的风险，比如在外部数据源更新失败的情况下缓存里的数据将会变得非法。这种方式在一定时间后延迟写，控制写的速率和批量写的场景下将会十分有效。 

write-back模式下可以实现以下几种特性：  
- 批量和合并操作    
- 延迟操作指定的时间窗口   
- 如果批处理的数据量超过阈值大小，那么将在定期刷新之前提前执行批处理操作   
- 如果外部资源操作还没有刷新，则从write-behind缓存当中加载数据    
- 根据外部资源的特性控制重试，速率和并发度    

可以通过查看这个 [write-behind-rxjava][write-behind-rxjava] 使用了[RxJava][rxjava]的简单demo来进行参考。

#### 分层

一个`CacheWriter` 可以用来整合多级缓存。

*layered cache* 多级缓存支持从一个数据系统所支持的外部缓存中加载和写入。这允许构建一个数据量更小速度也更快的缓存将数据回落到一个数据量更大但是速度较慢的大缓存中。典型的多级缓存包含堆外缓存，基于文件的缓存和远程的缓存。 

*victim cache* 是多级缓存的一种变种，它将把被驱逐的数据写入到二级缓存当中去。 `delete(K, V, RemovalCause)`方法支持查看被移除缓存的具体信息和移除原因。  

#### 同步监听器

`CacheWriter` 可以用来作为发布同步监听器使用。

一个*synchronous listener* 同步监听器将会按照指定key的操作顺序接收到相应的事件。这个监听器可以用来阻塞该操作也可以把这次操作事件投入到队列中被异步处理。这种类型的监听器可以用来作为备份或者构造一个分布式缓存。    

[rxjava]: https://github.com/ReactiveX/RxJava
[write-behind-rxjava]: https://github.com/ben-manes/caffeine/tree/master/examples/write-behind-rxjava