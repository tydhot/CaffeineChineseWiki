```java
CaffeineSpec spec = CaffeineSpec.parse(
    "maximumWeight=1000, expireAfterWrite=10m, recordStats");
LoadingCache<Key, Graph> graphs = Caffeine.from(spec)
    .weigher((Key key, Graph graph) -> graph.vertices().size())
    .build(key -> createExpensiveGraph(key));
```

`CaffeineSpec`为`Caffeine`提供了一个简单的字符格式配置。这里的字符串语法是一系列由逗号隔开的键值对组成，其中每个键值对对应一个配置方法。但是这里的字符配置不支持需要对象来作为参数的配置方法，比如 `removalListener`，这必须要在代码中进行配置。       

以下是各个配置字符串所对应的配置方法。将`maximumSize`和`maximumWeight`或者将`weakValues`和`weakValues` 在一起使用是不被允许的。      

 - initialCapacity=[integer]: 相当于配置 `Caffeine.initialCapacity`
 - maximumSize=[long]: 相当于配置 `Caffeine.maximumSize`
 - maximumWeight=[long]: 相当于配置 `Caffeine.maximumWeight`
 - expireAfterAccess=[duration]: 相当于配置 `Caffeine.expireAfterAccess`
 - expireAfterWrite=[duration]: 相当于配置 `Caffeine.expireAfterWrite`
 - refreshAfterWrite=[duration]: 相当于配置 `Caffeine.refreshAfterWrite`
 - weakKeys: 相当于配置 `Caffeine.weakKeys`
 - weakValues: 相当于配置 `Caffeine.weakValues`
 - softValues: 相当于配置 `Caffeine.softValues`
 - recordStats: 相当于配置 `Caffeine.recordStats`

时间间隔可以通过在一个integer类型之后跟上一个"d"，"h"，"m"，或者"s"来分别表示天，小时，分钟或者秒。另外，从2.8.7开始，ISO-8601标准的字符串也将被支持，并通过[Duration.parse](https://docs.oracle.com/javase/8/docs/api/java/time/Duration.html#parse-java.lang.CharSequence-)来进行解析。处于缓存时间的目的，不支持负的持续时间配置，并将会抛出异常。两种持续时间的表示格式如下所示。     

| 普通 | ISO-8601 | 描述
|--------|:--------|:------------|
| 50s    |  PT50S   | 50秒                    |
| 11m    |  PT11M   | 11分钟                  |
|  6h    |  PT6H    | 6小时                   |
|  3d    |  P3D     | 3天                     |
|        |  P3DT3H4M| 3天3小时4分钟           |
|        | -PT7H3M  | -7小时，-3分钟（不支持）|