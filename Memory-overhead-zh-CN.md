内存开销测试通过[Java Agent for Memory Measurements][1]来计算运行时内存。这个大小可能受到引用压缩，对象填充等JVM因素影响。这个基准测试可以通过`gradle memoryOverhead`来执行[内存基准测试][2]。    

通过在Java 5的[ConcurrentHashMap][3]上进行分支开发，Guava在每个缓存元素的所消耗的内存进行了优化。当使用软引用和弱饮用的时候这点优势会更加明显。因为Caffeine 是基于Java 8的[ConcurrentHashMap][3]进行封装的，需要额外的字段去移除一个被回收的字段。      

Caffeine将会基于用法懒加载或是动态调整器内部的数据结构大小。这减少了内存开销并可以在使用量增加的时候(比如吞吐量)方便追加内存占用。     

### 无界

| Cache | Baseline | Entry |
:--: | :--: | :--:
Caffeine | 264 bytes | 54 bytes (56 aligned) |
Guava | 1,032 bytes | 58 bytes (64 aligned) |

### 容量有界

| Cache | Baseline | Entry |
:--: | :--: | :--:
Caffeine | 1,112 bytes | 74 bytes (80 aligned) |
Guava | 1,384 bytes | 74 bytes (80 aligned) |

### 容量有界 & 根据访问时间过期

| Cache | Baseline | Entry |
:--: | :--: | :--:
Caffeine | 1,152 bytes | 82 bytes (88 aligned) |
Guava | 1,384 bytes | 74 bytes (80 aligned) |

### 容量有界 & 根据写入时间过期

| Cache | Baseline | Entry |
:--: | :--: | :--:
Caffeine | 1,176 bytes | 90 bytes (96 aligned) |
Guava | 1,544 bytes | 90 bytes (96 aligned) |

### 容量有界 & 根据写入时间刷新

| Cache | Baseline | Entry |
:--: | :--: | :--:
Caffeine | 1,168 bytes | 90 bytes (96 aligned) |
Guava | 27,200 bytes (?) | 90 bytes (96 aligned) |

### 权重有界

| Cache | Baseline | Entry |
:--: | :--: | :--:
Caffeine | 1,120 bytes | 82 bytes (88 aligned) |
Guava | 1,376 bytes | 74 bytes (80 aligned) |

### 根据访问时间过期

| Cache | Baseline | Entry |
:--: | :--: | :--:
Caffeine | 1,008 bytes | 82 bytes (88 aligned) |
Guava | 1,384 bytes | 74 bytes (80 aligned) |

### 根据写入时间过期

| Cache | Baseline | Entry |
:--: | :--: | :--:
Caffeine | 1,008 bytes | 82 bytes (88 aligned) |
Guava | 1,192 bytes | 74 bytes (80 aligned) |

### 根据访问时间 & 写入时间过期

| Cache | Baseline | Entry |
:--: | :--: | :--:
Caffeine | 1,048 bytes | 98 bytes (104 aligned) |
Guava | 1,544 bytes | 90 bytes (96 aligned) |

### 弱引用Keys

| Cache | Baseline | Entry |
:--: | :--: | :--:
Caffeine | 560 bytes | 98 bytes (104 aligned) |
Guava | 1,240 bytes | 66 bytes (72 aligned) |

### 弱引用Values

| Cache | Baseline | Entry |
:--: | :--: | :--:
Caffeine | 560 bytes | 98 bytes (104 aligned) |
Guava | 1,240 bytes | 74 bytes (80 aligned) |

### 弱引用Keys &  Values

| Cache | Baseline | Entry |
:--: | :--: | :--:
Caffeine | 616 bytes | 130 bytes (136 aligned) |
Guava | 1,416 bytes | 82 bytes (88 aligned) |

### 弱引用 Keys & 软引用 Values

| Cache | Baseline | Entry |
:--: | :--: | :--:
Caffeine | 616 bytes | 146 bytes (152 aligned) |
Guava | 1,416 bytes | 98 bytes (104 aligned) |

### 软引用Values

| Cache | Baseline | Entry |
:--: | :--: | :--:
Caffeine | 560 bytes | 114 bytes (120 aligned) |
Guava | 1,240 bytes | 90 bytes (96 aligned)|

[1]: https://github.com/jbellis/jamm
[2]: https://github.com/ben-manes/caffeine/blob/master/caffeine/src/jmh/java/com/github/benmanes/caffeine/cache/MemoryBenchmark.java
[3]: https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentHashMap.html