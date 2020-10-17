模拟器中包含一系列的驱逐策略和数据分发生成器。这将帮助我们了解各个驱逐策略是否适合使用场景。   

### 使用方法
在配置完所需要的 [配置][config]之后，在IDE中运行[模拟器][simulator]。或者也可以选择在命令行中使用 `gradlew simulator:run`命令。支持以下几种追踪格式：   

| 仓库| 体积| 位置
|:--|:--|:--|
| LIRS | 小| [git 仓库][lirs]
| UCSD | 小| [git 仓库][ucsd]
| ARC | 大| [作者主页][arc]
| UMass | 大| [UMass 仓库][umass]
| WikiBench | 大| [WikiBench][wikibench]

### 采样报告
由于批处理和数据广播的原因，时间指标只有在独立运行各个策略的时候才有比较的价值。   
```
╔══════════════════════╤══════════╤════════╤════════╤══════════╤═══════════╤══════════════════╤══════════╗
║ Policy               │ Hit rate │ Hits   │ Misses │ Requests │ Evictions │ Steps            │ Time     ║
╠══════════════════════╪══════════╪════════╪════════╪══════════╪═══════════╪══════════════════╪══════════╣
║ opt.Clairvoyant      │ 58.79 %  │ 9,323  │ 6,535  │ 15,858   │ 6,035     │ ?                │ 278.0 ms ║
╟──────────────────────┼──────────┼────────┼────────┼──────────┼───────────┼──────────────────┼──────────╢
║ sketch.WindowTinyLfu │ 56.11 %  │ 8,898  │ 6,960  │ 15,858   │ 6,460     │ 15,858 (100 %)   │ 315.2 ms ║
╟──────────────────────┼──────────┼────────┼────────┼──────────┼───────────┼──────────────────┼──────────╢
║ irr.Lirs             │ 55.97 %  │ 8,876  │ 6,982  │ 15,858   │ 6,482     │ 27,689 (174 %)   │ 311.0 ms ║
╟──────────────────────┼──────────┼────────┼────────┼──────────┼───────────┼──────────────────┼──────────╢
║ adaptive.Arc         │ 49.39 %  │ 7,833  │ 8,025  │ 15,858   │ 7,525     │ 15,858 (100 %)   │ 166.3 ms ║
╟──────────────────────┼──────────┼────────┼────────┼──────────┼───────────┼──────────────────┼──────────╢
║ linked.Lru           │ 46.51 %  │ 7,375  │ 8,483  │ 15,858   │ 7,983     │ 15,858 (100 %)   │ 128.2 ms ║
╟──────────────────────┼──────────┼────────┼────────┼──────────┼───────────┼──────────────────┼──────────╢
║ linked.Fifo          │ 40.72 %  │ 6,457  │ 9,401  │ 15,858   │ 8,901     │ 15,858 (100 %)   │ 163.6 ms ║
╚══════════════════════╧══════════╧════════╧════════╧══════════╧═══════════╧══════════════════╧══════════╝
```

[akka]: http://akka.io
[config]: https://github.com/ben-manes/caffeine/blob/master/simulator/src/main/resources/reference.conf
[simulator]: https://github.com/ben-manes/caffeine/blob/master/simulator/src/main/java/com/github/benmanes/caffeine/cache/simulator/Simulator.java
[lirs]: https://github.com/ben-manes/caffeine/tree/master/simulator/src/main/resources/com/github/benmanes/caffeine/cache/simulator/parser/lirs
[arc]: http://researcher.watson.ibm.com/researcher/view_person_subpage.php?id=4700
[ucsd]: https://github.com/ben-manes/caffeine/tree/master/simulator/src/main/resources/com/github/benmanes/caffeine/cache/simulator/parser/address
[umass]: http://traces.cs.umass.edu/index.php/Storage/Storage
[wikibench]: http://www.wikibench.eu