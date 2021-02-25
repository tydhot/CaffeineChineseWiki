在一开始，请先[sign the Contributor License Agreement](https://cla-assistant.io/ben-manes/caffeine)。

#### IDE
这个项目使用代码生成技术来配置缓存的特定配置减少内存开销。但是不幸的是，IDE可能没有意识到需要在编译的时候执行这个任务。你需要在命令行中重新生成并导入，使源文件在你的项目下被生成。      

```gradle
gradlew build -x test
```

##### 代码格式
需要遵循[Google的Java代码规范](https://google.github.io/styleguide/javaguide.html) 并可以导入至[Eclipse](https://raw.githubusercontent.com/google/styleguide/gh-pages/eclipse-java-google-style.xml) 或者 [Intellij](https://raw.githubusercontent.com/google/styleguide/gh-pages/intellij-java-google-style.xml)。     

#### Java Microbenchmark Harness
基准测试[JMH](https://github.com/melix/jmh-gradle-plugin) 可以通过以下命令执行。     

```gradle
gradlew jmh -PincludePattern=[class-name pattern]
```

#### Java Object Layout
[JOL](http://openjdk.java.net/projects/code-tools/jol) 可以通过以下命令执行。    

```gradle
gradlew [object-layout task] -PclassName=[class-name]
```

为了方便期间，项目的包名应该在类名之前。     

#### 静态分析
静态代码分析任务默认情况下不会被开启，需要通过以下命令以执行。    

```gradle
gradlew clean build -x test -Dcheckstyle -Dfindbugs -Dpmd
```

#### 参数化测试
缓存的单元测试可以选择所有缓存的配置进行运行。一个测试方法可以使用`@CacheSpec`注解，并使用`CacheProvider`将以所有可能的组合执行。测试用例可以以接受`CacheContext`作为参数来进行配置。     

参数化测试可以利用自动验证缓存内部数据结构的优势来检测缺陷。当一个成功的用例执行完毕后将会运行`CacheValidationListener`，而如果在这过程中检测到了错误将会结束测试并携带失败信息。     

```java
@Listeners(CacheValidationListener.class)
@Test(dataProviderClass = CacheProvider.class)
public final class CacheTest {

  @CacheSpec(
    keys = { ReferenceType.STRONG, ReferenceType.WEAK },
    values = { ReferenceType.STRONG, ReferenceType.WEAK, ReferenceType.SOFT },
    maximumSize = { MaximumSize.DISABLED, MaximumSize.FULL, MaximumSize.UNREACHABLE })
  @Test(dataProvider = "caches")
  public void getIfPresent_notFound(
      Cache<Integer, Integer> cache, CacheContext context) {
    // 这个测试将会至少通过72种不同的配置运行
    // (2中key引用类型) * (3种value类型) * (3种最大容量策略) * (4 种添加方式)
    assertThat(cache.getIfPresent(context.getAbsentKey()), is(nullValue());
    assertThat(cache.stats(), both(hasMissCount(1)).and(hasHitCount(0)));
  }
}
```

### 性能分析工具
[![YourKit](http://www.yourkit.com/images/yklogo.png)](http://www.yourkit.com) [![JProfiler](https://www.ej-technologies.com/images/product_banners/jprofiler_large.png)](https://www.ej-technologies.com/products/jprofiler/overview.html)

[YourKit](http://www.yourkit.com) & [JProfiler](https://www.ej-technologies.com) 为开源项目提供了所有特性的Java剖析工具。