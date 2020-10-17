Caffeine提供了四种缓存添加策略：手动加载，自动加载，手动异步加载和自动异步加载。

### 手动加载

```java
Cache<Key, Graph> cache = Caffeine.newBuilder()
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .maximumSize(10_000)
    .build();

// 查找一个缓存元素， 没有查找到的时候返回null
Graph graph = cache.getIfPresent(key);
// 查找缓存，如果缓存不存在则生成缓存元素,  如果无法生成则返回null
graph = cache.get(key, k -> createExpensiveGraph(key));
// 添加或者更新一个缓存元素
cache.put(key, graph);
// 移除一个缓存元素
cache.invalidate(key);
```

`Cache` 接口提供了显式搜索查找、更新和移除缓存元素的能力。

缓存元素可以通过调用 `cache.put(key, value)`方法被加入到缓存当中。如果缓存中指定的key已经存在对应的缓存元素的话，那么先前的缓存的元素将会被直接覆盖掉。因此，通过 `cache.get(key, k -> value)` 的方式将要缓存的元素通过原子计算的方式 插入到缓存中，以避免和其他写入进行竞争。值得注意的是，当缓存的元素无法生成或者在生成的过程中抛出异常而导致生成元素失败，`cache.get` 也许会返回 `null` 。    
当然，也可以使用`Cache.asMap()`所暴露出来的[ConcurrentMap][concurrent-map]的方法对缓存进行操作。

### 自动加载

```java
LoadingCache<Key, Graph> cache = Caffeine.newBuilder()
    .maximumSize(10_000)
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .build(key -> createExpensiveGraph(key));

// 查找缓存，如果缓存不存在则生成缓存元素,  如果无法生成则返回null
Graph graph = cache.get(key);
// 批量查找缓存，如果缓存不存在则生成缓存元素
Map<Key, Graph> graphs = cache.getAll(keys);
```

一个`LoadingCache`是一个`Cache` 附加上 `CacheLoader`能力之后的缓存实现。    

通过 `getAll`可以达到批量查找缓存的目的。 默认情况下，在`getAll` 方法中，将会对每个不存在对应缓存的key调用一次 `CacheLoader.load` 来生成缓存元素。 在批量检索比单个查找更有效率的场景下，你可以覆盖并开发`CacheLoader.loadAll` 方法来使你的缓存更有效率。

值得注意的是，你可以通过实现一个 `CacheLoader.loadAll`并在其中为没有在参数中请求的key也生成对应的缓存元素。打个比方，如果对应某个key生成的缓存元素与包含这个key的一组集合剩余的key所对应的元素一致，那么在`loadAll`中也可以同时加载剩下的key对应的元素到缓存当中。

### 手动异步加载
```java
AsyncCache<Key, Graph> cache = Caffeine.newBuilder()
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .maximumSize(10_000)
    .buildAsync();

// 查找缓存元素，如果不存在，则异步生成
CompletableFuture<Graph> graph = cache.get(key, k -> createExpensiveGraph(key));
```

一个`AsyncCache` 是 `Cache` 的一个变体，`AsyncCache`提供了在 [Executor][executor]上生成缓存元素并返回 [CompletableFuture][future]的能力。这给出了在当前流行的响应式编程模型中利用缓存的能力。

`synchronous()`方法给 `Cache`提供了阻塞直到异步缓存生成完毕的能力。

当然，也可以使用 `AsyncCache.asMap()`所暴露出来的[ConcurrentMap][concurrent-map]的方法对缓存进行操作。

默认的线程池实现是 [ForkJoinPool.commonPool()][fork-join] ，当然你也可以通过覆盖并实现 `Caffeine.executor(Executor)`方法来自定义你的线程池选择。

### 自动异步加载

```java
AsyncLoadingCache<Key, Graph> cache = Caffeine.newBuilder()
    .maximumSize(10_000)
    .expireAfterWrite(10, TimeUnit.MINUTES)
    // 你可以选择: 去异步的封装一段同步操作来生成缓存元素
    .buildAsync(key -> createExpensiveGraph(key));
    // 你也可以选择: 构建一个异步缓存元素操作并返回一个future
    .buildAsync((key, executor) -> createExpensiveGraphAsync(key, executor));

// 查找缓存元素，如果其不存在，将会异步进行生成
CompletableFuture<Graph> graph = cache.get(key);
// 批量查找缓存元素，如果其不存在，将会异步进行生成
CompletableFuture<Map<Key, Graph>> graphs = cache.getAll(keys);
```

一个 `AsyncLoadingCache`是一个 `AsyncCache` 加上 `AsyncCacheLoader`能力的实现。

在需要同步的方式去生成缓存元素的时候，`CacheLoader`是合适的选择。而在异步生成缓存的场景下， `AsyncCacheLoader`则是更合适的选择并且它会返回一个 [CompletableFuture][future]。

[concurrent-map]: https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentMap.html
[executor]: https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executor.html
[fork-join]: https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinPool.html
[future]: https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html