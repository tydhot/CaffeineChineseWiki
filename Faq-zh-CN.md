### 固定缓存元素

一个固定的缓存元素不会被驱逐策略所移除。当这个元素是一个有状态的资源的时候，这将非常有用，比如锁，只有当客户端主动结束其使用的时候才会将其移除。在这种情况下去对这些元素进行驱逐或重生成将会导致资源泄漏。    

在基于容量的驱逐策略中可以把一个缓存元素的权重设置为0来使得在驱逐过程中排除这个元素。这个元素将不会被计入到当前缓存的总量中，并会在基于容量的驱逐发生时被直接跳过。当自定义 `Weigher`的时候必须声明使用其的元素是否能被固定在缓存中。    

一个元素可以通过将持续时间设置为`Long.MAX_VALUE`，大概三百年左右，而在过期校验中被排除。当自定义`Expiry` 的时候必须声明使用其的元素是否能被固定在缓存中。    

元素的权重和过期时间将在下一次被修改的时候被重置。可以通过`cache.asMap().compute`方法来固定元素或者将元素解除固定。    

### 递归生成

在缓存中一次原子操作中发生的加载，生成或者回调可能不会写入到缓存中。在`ConcurrentHashMap` 中，类似的递归写入是[不被允许的][recursive-update]，在Java 8情况下这可能会导致活锁的产生，而在Java 9中可能会出现`IllegalStateException`异常。    

其中一个解决方案是通过异步生成的方式，比如使用`AsyncLoadingCache`。在这个场景下，映射已经生成，而映射的对象则是一个`CompletableFuture`，生成的过程将会在缓存的原子作用域之外进行。但是在无序的调用链中或者线程池耗尽了线程仍有死锁的风险。为了减少这样的场景，需要通过`Caffeine.executor`指定线程池实现为[Executors.newCachedThreadPool()][cached-executor]。  

[recursive-update]: https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentHashMap.html#compute-K-java.util.function.BiFunction-
[cached-executor]: https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executors.html#newCachedThreadPool--

### 写竞争

可能会存在Caffeine当前正在生成的元素个数接近或者大于当前缓存持有过的最大元素个数。这代表将要生成的元素个数可能已经接近当前缓存底层的 `ConcurrentHashMap`总量，在直到所有元素被生成计算完毕之前，map的扩容操作将会被一直阻塞。    

这种情况在缓存预热（接近但不是全部）的情况下很有可能发生。正在生成计算的数量与缓存的总大小接近的情况在小型缓存中会非常普遍。如果你正遇到这样的问题而在观察竞争（现象是不同线程的不同请求在`ConcurrentHashMap`中竞争同一个锁），也许你需要考虑去增加你的缓存的初始大小或者使用异步缓存。    

在`ConcurrentHashMap's`的文档中描述了一个很好的经验规则，
> 两个不同线程访问不同元素的锁竞争概率的大概为1  / (8 * 元素个数)    