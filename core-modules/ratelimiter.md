限流器

开始resilience4j限流器的学习。

## 介绍

限流是一种必不可少的技术，可以帮助您的API进行扩展，并建立服务的高可用性和可靠性。但是，这项技术还附带了一堆不同的选项，比如如何处理检测到的多余流量，或者您希望限制什么类型的请求。您可以简单地拒绝这个超限请求，或者构建一个队列以稍后执行它们，或者以某种方式组合这两种方法。

## 原理

Resilience4j提供了一个限流器，它将从epoch开始的所有纳秒划分为多个周期。每个周期的持续时间`RateLimiterConfig.limitRefreshPeriod`。在每个周期开始时，限流器将活动权限数设置为`RateLimiterConfig.limitForPeriod`。期间，
对于限流器的调用者，它看起来确实是这样的，但是对于`AtomicRateLimiter`实现，如果RateLimiter未被经常使用，则会在后台进行一些优化，这些优化将跳过此刷新。

限流器的默认实现是`AtomicRateLimiter`，它通过原子引用管理其状态。这个`AtomicRateLimiter`状态完全不可变，并且具有以下字段：

- activeCycle -上次调用的周期号
- activePermissions -在上次调用结束后，可用的活跃权限数。如果保留了某些权限，则可以为负。
- nanosToWait - 最后一次调用要等待的纳秒数

还有一个使用信号量的`SemaphoreBasedRateLimiter`和一个调度程序，它将在每个`RateLimiterConfig#limitRefreshPeriod`之后刷新活动权限数。

![avatar](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/images/44ca055-rate_limiter.png)

> 

## 创建RateLimiterRegistry

与断路器模块一样，此模块提供内存中的RateLimiterRegistry，可用于管理（创建和检索）限流器实例。

```java
RateLimiterRegistry rateLimiterRegistry = RateLimiterRegistry.ofDefaults();
```

## 创建和配置限流器

您可以提供自定义全局RateLimiterConfig。RateBuilder可以在自定义的LimiterConfig中创建RateLimiterConfig。可以使用生成器配置以下属性。

| 属性               | 默认值  | 描述                                                         |
| ------------------ | ------- | ------------------------------------------------------------ |
| timeoutDuration    | 5秒     | 线程等待权限的默认等待时间                                   |
| limitRefreshPeriod | 500纳秒 | 限流器每隔limitRefreshPeriod刷新一次，将允许处理的最大请求数量重置为limitForPeriod。 |
| limitForPeriod     | 50      | 在一次刷新周期内，允许执行的最大请求数                       |

例如，您希望将某个方法的调用率限制为每秒不超过10个请求。

```java
RateLimiterConfig config = RateLimiterConfig.custom()
  .limitRefreshPeriod(Duration.ofSeconds(1))
  .limitForPeriod(10)
  .timeoutDuration(Duration.ofMillis(25))
  .build();

// 创建Registry
RateLimiterRegistry rateLimiterRegistry = RateLimiterRegistry.of(config);

// 使用registry
RateLimiter rateLimiterWithDefaultConfig = rateLimiterRegistry
  .rateLimiter("name1");

RateLimiter rateLimiterWithCustomConfig = rateLimiterRegistry
  .rateLimiter("name2", config);
```



## 装饰和执行函数式接口

正如你所猜测的，RateLimiter有各种各样的高阶修饰函数，就像断路器一样。您可以用速率限制器装饰任何`Callable、Supplier、Runnable、Consumer、CheckedRunable、CheckedSupplier、CheckedConsumer`或`CompletionStage`。

```java
// 使用限流器装饰BackendService.doSomething()
CheckedRunnable restrictedCall = RateLimiter
    .decorateCheckedRunnable(rateLimiter, backendService::doSomething);

Try.run(restrictedCall)
    .andThenTry(restrictedCall)
    .onFailure((RequestNotPermitted throwable) -> LOG.info("Wait before call it again :)"));
```

可以使用changeTimeoutDuration和changeLimitForPeriod方法在运行时更改速率限制器参数。新的超时持续时间不会影响当前正在等待权限的线程。新的限制不会影响当前期间的权限，只会从下一个时间段应用。

```java
// 装饰对 BackendService.doSomething()的调用
CheckedRunnable restrictedCall = RateLimiter
    .decorateCheckedRunnable(rateLimiter, backendService::doSomething);

// 在第二次刷新周期中，限制器将获得100个权限
rateLimiter.changeLimitForPeriod(100);
```

## 处理RegistryEvents事件

您可以在RateLimiterRegistry上注册事件使用者，并在创建、替换或删除RateLimiter时执行操作。

```java
RateLimiterRegistry registry = RateLimiterRegistry.ofDefaults();
registry.getEventPublisher()
  .onEntryAdded(entryAddedEvent -> {
    RateLimiter addedRateLimiter = entryAddedEvent.getAddedEntry();
    LOG.info("RateLimiter {} added", addedRateLimiter.getName());
  })
  .onEntryRemoved(entryRemovedEvent -> {
    RateLimiter removedRateLimiter = entryRemovedEvent.getRemovedEntry();
    LOG.info("RateLimiter {} removed", removedRateLimiter.getName());
  });
```

## 处理RateLimiterEvents事件

限流器发出一个RateLimiterEvents流。事件可以是权限获取成功或获取失败。所有事件都包含其他信息，如事件创建时间和速率限制器名称。如果要使用事件，则必须注册事件使用者。

```java
rateLimiter.getEventPublisher()
    .onSuccess(event -> logger.info(...))
    .onFailure(event -> logger.info(...));
```

可以使用RxJava、RxJava2或projectreactor适配器将EventPublisher转换为反应流。

```java
ReactorAdapter.toFlux(rateLimiter.getEventPublisher())
    .filter(event -> event.getEventType() == FAILED_ACQUIRE)
    .subscribe(event -> logger.info(...))
```

## 重写RegistryStore

您可以通过自定义实现重写i在内存中的RegistryStore。例如，如果您想使用一个缓存，在一段时间后删除未使用的实例。

```java
RateLimiterRegistry rateLimiterRegistry = RateLimiterRegistry.custom()
  .withRegistryStore(new CacheRateLimiterRegistryStore())
  .build();
```

