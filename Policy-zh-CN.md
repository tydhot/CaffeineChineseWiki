策略的选择在缓存的构造中是灵活可选的。在程序的运行过程中，这些策略的配置也可以被检查并修改。策略通过[Optional][1]表明当前缓存是否支持其该策略。    

### 基于容量
```java
cache.policy().eviction().ifPresent(eviction -> {
  eviction.setMaximum(2 * eviction.getMaximum());
});
```

如果当前缓存容量是受最大权重所限制的，那么可以通过`weightedSize()`方法获得当前缓存。这与`Cache.estimatedSize()`区别在于，`Cache.estimatedSize()`将会返回当前缓存中存在的元素个数。   

缓存的最大容量或者总权重可以通过`getMaximum()` 得到并且可以通过`setMaximum(long)`方法对其进行调整。缓存将会不断驱逐元素，直到符合最新的阈值。    

如果想要得到缓存中最有可能被保留和最有可能被驱逐的元素子集，可以通过 `hottest(int)` 和`coldest(int)` 方法获得以上两个子集的元素快照。   

### 基于时间
```java
cache.policy().expireAfterAccess().ifPresent(expiration -> ...);
cache.policy().expireAfterWrite().ifPresent(expiration -> ...);
cache.policy().expireVariably().ifPresent(expiration -> ...);
cache.policy().refreshAfterWrite().ifPresent(refresh -> ...);
```

 `ageOf(key, TimeUnit)`方法提供了查看缓存元素在`expireAfterAccess`，`expireAfterWrite`或者 `refreshAfterWrite` 策略下的空闲时间的途径。缓存中的元素最大可持续时间可以通过`getExpiresAfter(TimeUnit)`方法获取，并且可以通过`setExpiresAfter(long, TimeUnit)`方法来进行调整。    

如果需要查看最接近保留或者最接近过期的元素子集，那么需要调用 `youngest(int)` 和`oldest(int)`方法来得到以上两个子集的元素快照。     

[1]: https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html