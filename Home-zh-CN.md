Caffeine是一个基于Java8开发的提供了[近乎最佳][efficiency]命中率的[高性能](Benchmarks-zh-CN)的缓存库。

缓存和[ConcurrentMap][concurrent-map]有点相似，但还是有所区别。最根本的区别是[ConcurrentMap][concurrent-map]将会持有所有加入到缓存当中的元素，直到它们被从缓存当中手动移除。但是，Caffeine的缓存`Cache` 通常会被配置成自动驱逐缓存中元素，以限制其内存占用。在某些场景下，`LoadingCache`和`AsyncLoadingCache` 因为其自动加载缓存的能力将会变得非常实用。

Caffeine提供了灵活的构造器去创建一个拥有下列特性的缓存：
 * [自动加载][population]元素到缓存当中，异步加载的方式也可供选择
 * 当达到最大容量的时候可以使用基于[就近度和频率][efficiency]的算法进行[基于容量的驱逐][size]
 * 将根据缓存中的元素上一次访问或者被修改的时间进行[基于过期时间的驱逐][time]
 * 当向缓存中一个已经过时的元素进行访问的时候将会进行[异步刷新][refresh]
 * key将自动被[弱引用][reference]所封装
 * value将自动被[弱引用或者软引用][reference]所封装
 * 驱逐(或移除)缓存中的元素时将会进行[通知][listener]
 * [写入传播][writer]到一个外部数据源当中
 * 持续计算缓存的访问[统计指标][statistics]

为了提高集成度，扩展模块提供了[JSR-107 JCache](JCache-zh-CN)和[Guava](Guava-zh-CN)适配器。JSR-107规范了基于Java 6的API，在牺牲了功能和性能的代价下使代码更加规范。Guava的Cache是Caffeine的原型库并且Caffeine提供了适配器以供简单的迁移策略。

我们很欢迎你的贡献。请阅读[设计部分][design-doc],[贡献指南][contribute]和[开发路线图](https://github.com/ben-manes/caffeine/wiki/Roadmap)。

[guava]: https://github.com/google/guava
[clhm]: https://code.google.com/p/concurrentlinkedhashmap
[guava-cache]: https://code.google.com/p/guava-libraries/wiki/CachesExplained
[concurrent-map]: https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentMap.html
[population]: https://github.com/ben-manes/caffeine/wiki/Population-zh-CN
[size]: https://github.com/ben-manes/caffeine/wiki/Eviction-zh-CN#基于容量
[efficiency]: https://github.com/ben-manes/caffeine/wiki/Efficiency-zh-CN
[time]: https://github.com/ben-manes/caffeine/wiki/Eviction-zh-CN#基于时间
[refresh]: https://github.com/ben-manes/caffeine/wiki/Refresh-zh-CN
[reference]: https://github.com/ben-manes/caffeine/wiki/Eviction-zh-CN#基于引用
[listener]: https://github.com/ben-manes/caffeine/wiki/Removal-zh-CN
[writer]: https://github.com/ben-manes/caffeine/wiki/Writer-zh-CN
[statistics]: https://github.com/ben-manes/caffeine/wiki/Statistics-zh-CN
[design-doc]: https://github.com/ben-manes/caffeine/wiki/Design-zh-CN
[contribute]: https://github.com/ben-manes/caffeine/wiki/Contribute-zh-CN