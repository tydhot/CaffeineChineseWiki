在默认情况下，当一个缓存元素过期的时候，Caffeine不会自动立即将其清理和驱逐。而它将会在写操作之后进行少量的维护工作，在写操作较少的情况下，也偶尔会在读操作之后进行。如果你的缓存吞吐量较高，那么你不用去担心你的缓存的过期维护问题。但是如果你的缓存读写操作都很少，可以像下文所描述的方式额外通过一个线程去通过`Cache.cleanUp()` 方法在合适的时候触发清理操作。   

```java
LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
    .scheduler(Scheduler.systemScheduler())
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .build(key -> createExpensiveGraph(key));
```

 `Scheduler`可以提前触发过期元素清理移除。在过期事件之间进行调度，以期在短时间内最小化连续的批处理操作的数量。这里的调度是尽可能做到合理，并不能保证在一个元素过期的时候就将其清除。Java 9以上的用户可以通过`Scheduler.systemScheduler()`来利用专用的系统范围内的调度线程。    

```java
Cache<Key, Graph> graphs = Caffeine.newBuilder().weakValues().build();
Cleaner cleaner = Cleaner.create();

cleaner.register(graph, graphs::cleanUp);
graphs.put(key, graph);
```

Java 9以上的用户也可以通过[Cleaner][cleaner]去触发移除关于基于引用的元素(在使用了 `weakKeys`, `weakValues`, 或者 `softValues`的情况下)。只要将key或者缓存的元素value注册到`Cleaner`上，就可以在程序运行中调用`Cache.cleanUp()`方法触发缓存的维护工作。    

[cleaner]: https://docs.oracle.com/javase/9/docs/api/java/lang/ref/Cleaner.html