```java
CaffeineSpec spec = CaffeineSpec.parse(
    "maximumWeight=1000, expireAfterWrite=10m, recordStats");
LoadingCache<Key, Graph> graphs = Caffeine.from(spec)
    .weigher((Key key, Graph graph) -> graph.vertices().size())
    .build(key -> createExpensiveGraph(key));
```

`CaffeineSpec`为`Caffeine`提供了一个简单的字符格式配置。这里的字符串语法是一系列由逗号隔开的键值对组成，其中每个键值对对应一个配置方法。但是这里的字符配置不支持需要对象来作为参数的配置方法，比如 `removalListener`，这样的配置必须要在代码中进行配置。       

以下是各个配置键值对字符串所对应的配置方法。将`maximumSize`和`maximumWeight`或者将`weakValues`和`weakValues` 在一起使用是不被允许的。      

 - initialCapacity=[integer]: 相当于配置 `Caffeine.initialCapacity`
 - maximumSize=[long]: 相当于配置 `Caffeine.maximumSize`
 - maximumWeight=[long]: 相当于配置 `Caffeine.maximumWeight`
 - expireAfterAccess=[持续时间]: 相当于配置 `Caffeine.expireAfterAccess`
 - expireAfterWrite=[持续时间]: 相当于配置 `Caffeine.expireAfterWrite`
 - refreshAfterWrite=[持续时间]: 相当于配置 `Caffeine.refreshAfterWrite`
 - weakKeys: 相当于配置 `Caffeine.weakKeys`
 - weakValues: 相当于配置 `Caffeine.weakValues`
 - softValues: 相当于配置 `Caffeine.softValues`
 - recordStats: 相当于配置 `Caffeine.recordStats`

持续时间可以通过在一个integer类型之后跟上一个"d"，"h"，"m"，或者"s"来分别表示天，小时，分钟或者秒。另外，ISO-8601标准的字符串也被支持来配置持续时间，并通过[Duration.parse](https://docs.oracle.com/javase/8/docs/api/java/time/Duration.html#parse-java.lang.CharSequence-)来进行解析。出于表示缓存持续时间的目的，这里不支持配置负的持续时间，并将会抛出异常。两种持续时间表示格式的示例如下所示。     

| 普通 | ISO-8601 | 描述
|--------|:--------|:------------|
| 50s    |  PT50S   | 50秒                    |
| 11m    |  PT11M   | 11分钟                  |
|  6h    |  PT6H    | 6小时                   |
|  3d    |  P3D     | 3天                     |
|        |  P3DT3H4M| 3天3小时4分钟           |
|        | -PT7H3M  | -7小时，-3分钟（不支持）|