```java
// Guava的LoadingCache接口
LoadingCache<Key, Graph> graphs = CaffeinatedGuava.build(
    Caffeine.newBuilder().maximumSize(10_000),
    new CacheLoader<Key, Graph>() { // Guava'的CacheLoader
        @Override public Graph load(Key key) throws Exception {
          return createExpensiveGraph(key);
        }
    });
```

#### API兼容性

Caffeine 提供了适配器使缓存暴露Guava接口。这些适配器提供与Guava 相同的API规范。这些模仿Guava的操作已经经过Guava的测试组组件验证。      

当迁移至Caffeine的接口的时候，记得注意虽然两个缓存之间可能存在相同名字的方法但是操作完全不同。请更仔细彻底的通过JavaDoc比较两者的用法。     

#### 最大容量 (或总权重)

Guava将通过LRU算法在到达最大容量之前就开始进行驱逐。Caffeine将通过Window TinyLFU算法在超过阈值之后进行清除。     

#### 立即过期

Guava 通过将最大容量设置为0的方式达到使缓存中的元素立即过期(`expireAfterAccess(0, timeUnit)` 和 
`expireAfterWrite(0, timeUnit)`)的效果。这在移除通知中的移除原因显示是由于容量而不是过期。Caffeine 则在移除原因中明确了是由于过期的原因。     

#### 替换通知

Guava中的元素被以任何原因替换的时候移除监听器都将收到通知。Caffeine 将不会通知相同引用性下的新旧值替换。      

#### 失效正在生成计算的元素

当元素正在生成计算中的时候，Guava将会忽略将其失效的请求。在Caffeine中的做法则是阻塞发起失效请求的线程直到生成计算结束，再将其移除。但是， 当通过`invalidateAll()`批量失效一批正在计算的元素的时候，可能由于底层hash表的抑制，导致这批元素的生成计算被直接跳过。当使用异步缓存的时候，失效操作将不会阻塞，而是直接移除缓存中的future，并由执行生成计算的线程发出移除通知。      

#### 异步维护

Guava 将在读写操作中分摊缓存中的维护操作。Caffeine 将通过配置的executor(默认: `ForkJoinPool.commonPool()`) 周期性地执行维护操作。也可以在调用线程中通过`cleanUp()`方法进行执行。      

#### 异步通知

Guava从队列中处理驱逐通知，任何一个调用线程都可以从这个队列中获取驱逐通知。Caffeine则交给配置的executor 去执行(默认: `ForkJoinPool.commonPool()`)。     

#### 异步刷新

Guava 在请求刷新的线程中执行一个元素的重新生成计算。Caffeine则交给配置的executor 去执行(默认: `ForkJoinPool.commonPool()`)。      

#### 生成计算结果为null

Guava生成计算元素的如果是null将会抛出异常，同时如果是在刷新的过程中出现这种情况，将会在缓存中保留这个元素。Caffeine 会直接返回null，如果实在刷新的过程中生成了null将会从缓存中移除这个元素。在使用了Guava适配器的情况下，Caffeine如果使用了Guava 的 `CacheLoader`接口的话将会选择和Guava一样的处理措施。       

#### Map的生成计算方法

Guava在21.0版本之前所继承`ConcurrentMap`的默认方法(`compute`，`computeIfAbsent`，`computeIfPresent`和 `merge`)都是非原子性的。Caffeine实现了这些Java8新增功能的原子操作。    

#### 缓存统计

Guava中`CacheStats.loadExceptionCount()`和`CacheStats.loadExceptionRate()`方法在Caffeine中被重命名为`CacheStats.loadFailureCount()`和`CacheStats.loadFailureRate()`。因为在Caffeine生成计算结果null的时候将会被视为加载失败而不是当作异常处理。      

#### Android 和 GWT 的兼容性

由于Caffeine对Java8的要求，将不兼容不支持Java8的平台。   