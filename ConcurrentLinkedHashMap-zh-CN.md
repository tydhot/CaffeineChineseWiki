#### 生成计算

[ConcurrentLinkedHashMap][1]继承了`ConcurrentMap`中非原子操作的默认方法(`compute`, `computeIfAbsent`， `computeIfPresent`，和`merge`)。Caffeine实现了这些Java8新增功能的原子操作。 
  

#### 权重

ConcurrentLinkedHashMap要求权重的最小值为1。与Guava类似，Caffeine 要求权重的最小值为0，以表明在基于容量的驱逐策略中这个元素永远不会被驱逐。    

#### 异步通知

ConcurrentLinkedHashMap从队列中处理驱逐通知，任何一个调用线程都可以从这个队列中获取驱逐通知。Caffeine则交给配置的executor去执行(默认: [ForkJoinPool.commonPool()][2])。    

#### 快照视图

ConcurrentLinkedHashMap支持查看保留顺序的快照视图。Caffeine 则提供通过`Cache.policy()`获取的`Policy.Eviction`提供此功能，`ascendingMapWithLimit`表示`coldest`， 而 `descendingMapWithLimit`表示`hottest`。      

#### 序列化

ConcurrentLinkedHashMap序列化会保留元素并在序列化中拒绝对元素的驱逐。Caffeine和Guava一样，序列化后只保留配置并不会保留数据。     

[1]: https://code.google.com/p/concurrentlinkedhashmap
[2]: https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinPool.html