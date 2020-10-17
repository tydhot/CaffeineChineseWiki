Caffeine 提供了三种驱逐策略，分别是基于容量，基于时间和基于引用三种类型。

### 基于容量
```java
// 基于缓存内的元素个数进行驱逐
LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
    .maximumSize(10_000)
    .build(key -> createExpensiveGraph(key));

// 基于缓存内元素权重进行驱逐
LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
    .maximumWeight(10_000)
    .weigher((Key key, Graph graph) -> graph.vertices().size())
    .build(key -> createExpensiveGraph(key));
```

如果你的缓存容量不希望超过某个特定的大小，那么记得使用`Caffeine.maximumSize(long)`。缓存将会尝试通过基于[就近度和频率的算法](Efficiency)来驱逐掉不会再被使用到的元素。

另一种情况，你的缓存可能中的元素可能存在不同的“权重”--打个比方，你的缓存中的元素可能有不同的内存占用--你也许需要借助`Caffeine.weigher(Weigher)` 方法来界定每个元素的权重并通过 `Caffeine.maximumWeight(long)`方法来界定缓存中元素的总权重来实现上述的场景。除了“最大容量”所需要的注意事项，在基于权重驱逐的策略下，一个缓存元素的权重计算是在其创建和更新时，此后其权重值都是静态存在的，在两个元素之间进行权重的比较的时候，并不会根据进行相对权重的比较。

### 基于时间
```java
// 基于固定的过期时间驱逐策略
LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
    .expireAfterAccess(5, TimeUnit.MINUTES)
    .build(key -> createExpensiveGraph(key));
LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .build(key -> createExpensiveGraph(key));

// 基于不同的过期驱逐策略
LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
    .expireAfter(new Expiry<Key, Graph>() {
      public long expireAfterCreate(Key key, Graph graph, long currentTime) {
        // Use wall clock time, rather than nanotime, if from an external resource
        long seconds = graph.creationDate().plusHours(5)
            .minus(System.currentTimeMillis(), MILLIS)
            .toEpochSecond();
        return TimeUnit.SECONDS.toNanos(seconds);
      }
      public long expireAfterUpdate(Key key, Graph graph, 
          long currentTime, long currentDuration) {
        return currentDuration;
      }
      public long expireAfterRead(Key key, Graph graph,
          long currentTime, long currentDuration) {
        return currentDuration;
      }
    })
    .build(key -> createExpensiveGraph(key));
```

Caffeine提供了三种方法进行基于时间的驱逐:
 * `expireAfterAccess(long, TimeUnit):` 一个元素在上一次读写操作后一段时间之后，在指定的时间后没有被再次访问将会被认定为过期项。在当被缓存的元素时被绑定在一个session上时，当session因为不活跃而使元素过期的情况下，这是理想的选择。
 * `expireAfterWrite(long, TimeUnit):` 一个元素将会在其创建或者最近一次被更新之后的一段时间后被认定为过期项。在对被缓存的元素的时效性存在要求的场景下，这是理想的选择。
 * `expireAfter(Expiry):` 一个元素将会在指定的时间后被认定为过期项。当被缓存的元素过期时间收到外部资源影响的时候，这是理想的选择。

在写操作，和偶尔的读操作中将会进行周期性的过期事件的执行。过期事件的调度和触发将会在O(1)的时间复杂度内完成。

为了使过期更有效率，可以通过在你的Cache构造器中通过`Scheduler`接口和`Caffeine.scheduler(Scheduler)` 方法去指定一个调度线程代替在缓存活动中去对过期事件进行调度。使用Java 9以上版本的用户可以选择`Scheduler.systemScheduler()`利用系统范围内的调度线程。

当测试基于时间的驱逐策略的时候，不需要坐在板凳上等待现实时钟的转动。使用`Ticker`接口和 `Caffeine.ticker(Ticker)`方法在你的Cache构造器中去指定一个时间源可以避免苦苦等待时钟转动的麻烦。Guava的测试库也提供了`FakeTicker`去达到同样的目的。

### 基于引用
```java
// 当key和缓存元素都不再存在其他强引用的时候驱逐
LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
    .weakKeys()
    .weakValues()
    .build(key -> createExpensiveGraph(key));

// 当进行GC的时候进行驱逐
LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
    .softValues()
    .build(key -> createExpensiveGraph(key));
```

Caffeine 允许你配置你的缓存去让GC去帮助清理缓存当中的元素，其中key支持弱引用，而value则支持弱引用和软引用。记住 `AsyncCache`不支持软引用和弱引用。

`Caffeine.weakKeys()` 在保存key的时候将会进行弱引用。这允许在GC的过程中，当key没有被任何强引用指向的时候去将缓存元素回收。由于GC只依赖于引用相等性。这导致在这个情况下，缓存将会通过引用相等(==)而不是对象相等 `equals()`去进行key之间的比较。

`Caffeine.weakValues()`在保存value的时候将会使用弱引用。这允许在GC的过程中，当value没有被任何强引用指向的时候去将缓存元素回收。由于GC只依赖于引用相等性。这导致在这个情况下，缓存将会通过引用相等(==)而不是对象相等 `equals()`去进行value之间的比较。

`Caffeine.softValues()`在保存value的时候将会使用软引用。为了相应内存的需要，在GC过程中被软引用的对象将会被通过LRU算法回收。由于使用软引用可能会影响整体性能，我们还是建议通过使用基于缓存容量的驱逐策略代替软引用的使用。同样的，使用 `softValues()` 将会通过引用相等(==)而不是对象相等 `equals()`去进行value之间的比较。