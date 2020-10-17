### 当前
当前的重点是帮助用户使用该代码库并将用户的想法移植到自定义的方案中。特别是对以下几点保持强烈的兴趣，   
* 解决缺陷并听取用户反馈      
* 集成到流行的框架中 (比如Spring)      
* 被数据库和搜索产品所采用 (比如 HBase，Solr)       
* 为移植到其他系统提供咨询 (比如 Postgres，Cassandra's off-heap)        

### Version 2.x
* 在dedicated queue中维持0权重元素
* 为批量更新添加CacheLoader的`reloadAll`API ([#7](https://github.com/ben-manes/caffeine/issues/7))
* 为`AsyncLoadingCache`添加`Map<K,CompletableFuture<V>>`视图  ([#156](https://github.com/ben-manes/caffeine/issues/156))

### Version 3.0
* 移除过时的类和方法
* JDK 9: 将`sun.misc.Unsafe` 的用法迁移至 [VarHandles](http://openjdk.java.net/jeps/193)
* `refresh`时future使调用方能够阻碍在重载时阻塞([143](https://github.com/ben-manes/caffeine/issues/143))      
* 当元素不处于缓存当中时主动通过CacheWriter的 `delete`方法使其失效     
* 重写JCache过期部分使得`eager`和`lazy`配置合并到一起   
* 使`CacheStats` value-type ready by removing constructor (a la `Optional`)          
* 使用JDK9'的shared scheduler进行主动过期 ([#195](https://github.com/ben-manes/caffeine/issues/195))        
* 适配TinyLFU策略 ([106](https://github.com/ben-manes/caffeine/issues/106))        

### 未来

* Future JDKs
  * JDK 11: 思考方法如何更好利用[Value Types](http://openjdk.java.net/jeps/169)