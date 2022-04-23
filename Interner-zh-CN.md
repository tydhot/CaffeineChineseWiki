_在3.1.0版本中引入_

```java
Interner<String> interner = Interner.newStrongInterner();
String s1 = interner.intern(new String("value"));
String s2 = interner.intern(new String("value"));
assert s1 == s2
```

一个 `Interner` 将会返回给定元素的标准规范形式。当调用 `intern(e)` 方法时，如果该 interner 已经包含一个由 [Object.equals][equals]
方法确认相等的对象的时候，将会返回其中已经包含的对象。否则将会新添加该对象返回。这种重复数据删除通常用于共享规范实例来减少内存使用。

```java
LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
    .weakKeys()
    .build(key -> createExpensiveGraph(key));
Interner<Key> interner = Interner.newWeakInterner();

Key canonical = interner.intern(key);
Graph graph = cache.get(canonical);
```

一个弱引用的 interner 允许其中的内部对象在没有别的强引用的前提下在被 gc 回收。与 caffeine 的 `Caffeine.weakKeys()` 不同的是，
这里通过 (==) 来进行比较而不是 `equals()`。这样可以确保应用程序所引用的 interner 中的值正是缓存中的实际实例。否则在缓存当中的所
持有的弱引用 key 很有可能是与应用程序所持有的实例不同的实例，这会导致这个 key 会在 gc 的时候被提早从缓存当中被淘汰。

[equals]: https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html#equals(java.lang.Object)