# 限流器例子

esilience4j-ratelimiter的例子

## 创建RateLimiterRegistry

```java
// 创建限流器的自定义配置
RateLimiterConfig config = RateLimiterConfig.custom()
  .timeoutDuration(TIMEOUT)
  .limitRefreshPeriod(REFRESH_PERIOD)
  .limitForPeriod(LIMIT)
  .build();

// 使用自定义的全局配置创建RatelimiterRegistry
RateLimiterRegistry registry = RateLimiterRegistry.of(config);
```



## 重载RegistryStore

您可以通过自定义实现重写在内存中的RegistryStore。例如，如果要使用一个缓存，该缓存会在一段时间后删除未使用的实例。

```java
RateLimiterRegistry rateLimiterRegistry = RateLimiterRegistry.custom()
  .withRegistryStore(new CacheRateLimiterRegistryStore())
  .build();
```

