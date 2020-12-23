# 限时器

开始限时器的学习。

## 创建TimeLimiterRegistry

就像断路器模块一样，这个模块提供了一个内存中的`TimeLimiterRegistry`，你可以使用它管理（创建和检索）限时器实例。

```java
TimeLimiterRegistry timeLimiterRegistry = TimeLimiterRegistry.ofDefaults();
```

## 创建和配置TimeLimiter

你可以创建一个自定义的全局TimeLimiterConfig，通过使用TimeLimiterConfig建造者模式创建一个TimeLimiterConfig。通过builder可以配置：

- 超时时间
- 如果此时正在进行异步调用，是否要取消。

```java
TimeLimiterConfig config = TimeLimiterConfig.custom()
   .cancelRunningFuture(true)
   .timeoutDuration(Duration.ofMillis(500))
   .build();

// 使用自定义的全局配置创建一个TimeLimiterRegistry
TimeLimiterRegistry timeLimiterRegistry = TimeLimiterRegistry.of(config);

//registry使用默认的配置创建一个TimeLimiter
TimeLimiter timeLimiterWithDefaultConfig = registry.timeLimiter("name1");

// 使用自定义的配置创建一个TimeLimiter实例
TimeLimiterConfig config = TimeLimiterConfig.custom()
   .cancelRunningFuture(false)
   .timeoutDuration(Duration.ofMillis(1000))
   .build();

TimeLimiter timeLimiterWithCustomConfig = registry.timeLimiter("name2", config);
```



## 装饰与执行函数式接口

限时器有高阶函数可以对`CompletionStage`或`Future`进行装饰，为了限制执行的时间。

```java
// 当我有一个耗时的任务helloWorldService.sayHelloWorld()时
HelloWorldService helloWorldService = mock(HelloWorldService.class);

// 创建一个限时器实例
TimeLimiter timeLimiter = TimeLimiter.of(Duration.ofSeconds(1));
// 需要一个调度器对非阻塞CompletableFuture进行调度，控制超时时间
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(3);

// 返回CompletableFuture类型的非阻塞变量
CompletableFuture<String> result = timeLimiter.executeCompletionStage(
  scheduler, () -> CompletableFuture.supplyAsync(helloWorldService::sayHelloWorld)).toCompletableFuture();

// 阻塞方式，实际上是调用了future.get(timeoutDuration, MILLISECONDS)
String result = timeLimiter.executeFutureSupplier(
  () -> CompletableFuture.supplyAsync(() -> helloWorldService::sayHelloWorld));
```



