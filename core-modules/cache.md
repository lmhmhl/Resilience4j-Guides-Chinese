# 缓存

开始resilience4j-cache的学习

## 创建和配置缓存

下面的例子展示了怎样使用缓存装饰器对lambda表达式进行装饰，缓存抽象将lambda表达式的结果放在缓存实例（JCache）中，并尝试在调用lambda表达式之前从缓存中检索先前缓存的结果。如果从分布式缓存检索缓存失败，将处理异常并调用lambda表达式。

```java
// 创建CacheContext封装为JCache实例
javax.cache.Cache<String, String> cacheInstance = Caching
  .getCache("cacheName", String.class, String.class);
Cache<String, String> cacheContext = Cache.of(cacheInstance);

// 装饰对BackendService.doSomething()的调用
CheckedFunction1<String, String> cachedFunction = Decorators
    .ofCheckedSupplier(() -> backendService.doSomething())
    .withCache(cacheContext)
    .decorate();
String value = Try.of(() -> cachedFunction.apply("cacheKey")).get();
```



## 处理缓存事件

缓存会发出CacheEvents流。事件可以是缓存命中、缓存未命中或错误。

```java
cacheContext.getEventPublisher()
    .onCacheHit(event -> logger.info(...))
    .onCacheMiss(event -> logger.info(...))
    .onError(event -> logger.info(...));
```



## Ehcache的例子

```java
compile 'org.ehcache:ehcache:3.7.1'
```

```java
// 配置缓存
this.cacheManager = Caching.getCachingProvider().getCacheManager();
this.cache = Cache.of(cacheManager
    .createCache("booksCache", new MutableConfiguration<>()));

// 从缓存中获得books
List<Book> books = Cache.decorateSupplier(cache, library::getBooks)
    .apply(BOOKS_CACHE_KEY);
```

不建议在生产中使用JCache引用实现，因为它会导致一些并发问题。使用Ehcache、Caffeine、Redisson、Hazelcast、Ignite或其他JCache实现。

