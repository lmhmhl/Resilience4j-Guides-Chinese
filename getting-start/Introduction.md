# 介绍

Resilience4j是受到Netflix Hystrix的启发，为Java8和函数式编程所设计的轻量级容错框架。整个框架只是使用了Varr的库，不需要引入其他的外部依赖。与此相比，Netflix Hystrix对Archaius具有编译依赖，而Archaius需要更多的外部依赖，例如Guava和Apache Commons Configuration。

Resilience4j提供了提供了一组高阶函数（装饰器），包括断路器，限流器，重试机制，隔离机制。你可以使用其中的一个或多个装饰器对函数式接口，lambda表达式或方法引用进行装饰。这么做的优点是你可以选择所需要的装饰器进行装饰。

在使用Resilience4j的过程中，不需要引入所有的依赖，只引入需要的依赖即可。

## 先睹为快

下面是关于当异常调用发生时，使用断路器和重试机制对lambda表达式进行装饰，完成最多三次重试的例子。

你可以配置每次重试之间的等待时间间隔，并且可以配置自定义的服务降级算法。

下面的例子使用了Varr库中的` Try` 函子对异常进行恢复。当所有的重试失败时，调用一个lambda表达式作为服务降级方法。

```java
// 使用默认的配置创建一个断路器
CircuitBreaker circuitBreaker = CircuitBreaker
  .ofDefaults("backendService");

// 使用默认的配置创建重试机制
// 默认进行三次重试，每次重试的固定时间间隔为500毫秒
Retry retry = Retry
  .ofDefaults("backendService");

// 使用默认的配置创建隔离机制
Bulkhead bulkhead = Bulkhead
  .ofDefaults("backendService");

Supplier<String> supplier = () -> backendService
  .doSomething(param1, param2)

// 使用断路器、隔离机制、重试机制对backendService.doSomething() 进行装饰
// **注意: 你将需要 resilience4j-all 依赖
Supplier<String> decoratedSupplier = Decorators.ofSupplier(supplier)
  .withCircuitBreaker(circuitBreaker)
  .withBulkhead(bulkhead)
  .withRetry(retry)  
  .decorate();

// 执行被装饰的supplier并且从异常中恢复
String result = Try.ofSupplier(decoratedSupplier)
  .recover(throwable -> "Hello from Recovery").get();

// 只是在执行过程中，使用断路器对lambda表达式进行保护
String result = circuitBreaker
  .executeSupplier(backendService::doSomething);

// 可以使用ThreadPoolBulkhead对supplier进行异步执行
 ThreadPoolBulkhead threadPoolBulkhead = ThreadPoolBulkhead
  .ofDefaults("backendService");

// 调度程序在非阻塞的CompletableFuture上设置超时
ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(3);
TimeLimiter timeLimiter = TimeLimiter.of(Duration.ofSeconds(1));

CompletableFuture<String> future = Decorators.ofSupplier(supplier)
    .withThreadPoolBulkhead(threadPoolBulkhead)
    .withTimeLimiter(timeLimiter, scheduledExecutorService)
    .withCircuitBreaker(circuitBreaker)
    .withFallback(asList(TimeoutException.class, 
                         CallNotPermittedException.class, 
                         BulkheadFullException.class),  
                  throwable -> "Hello from Recovery")
    .get().toCompletableFuture();
```



## 模块化

在使用Resilience4j的过程中，不需要引入所有的依赖，只引入需要的依赖即可。

## 所有核心模块和装饰器

- resilience4j-all

## 核心模块

- 
  resilience4j-circuitbreaker: 熔断
- resilience4j-ratelimiter: 限流
- resilience4j-bulkhead: 隔离
- resilience4j-retry: 自动重试（同步，异步）
- resilience4j-cache: 结果缓存
- resilience4j-timelimiter: 超时处理

## 附加模块

- resilience4j-retrofit: Retrofit 适配器
- resilience4j-feign: Feign 适配器
- resilience4j-consumer: 循环缓冲区事件消费者
- resilience4j-kotlin: Kotlin 协程支持

### 框架模块

- resilience4j-spring-boot: Spring Boot Starter
- resilience4j-spring-boot2: Spring Boot 2 Starter
- resilience4j-ratpack: Ratpack Starter
- resilience4j-vertx: Vertx Future decorator

### 反应式模块

- resilience4j-rxjava2: 定制的 RxJava2 操作符
- resilience4j-reactor: 定制的 Spring Reactor 操作符

### 监控模块

- resilience4j-micrometer: Micrometer Metrics exporter
- resilience4j-metrics: Dropwizard Metrics exporter
- resilience4j-prometheus: Prometheus Metrics exporter

## 第三方模块

- [Camel Circuit Breaker](https://camel.apache.org/manual/latest/resilience4j-eip.html)
- [Spring Cloud Circuit Breaker](https://spring.io/projects/spring-cloud-circuitbreaker)
- [http4k resilience module](https://www.http4k.org/guide/modules/resilience/)

















