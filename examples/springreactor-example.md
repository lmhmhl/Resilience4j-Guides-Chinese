# 例子

## 使用CircuitBreaker装饰Mono 或Flux

下面的示例演示如何使用自定义Reactor操作符装饰Mono。也支持Flux。

`CircuitBreakerOperator`检查下游订阅者/观察者是否可以获得订阅上游发布者的权限。如果断路器开启，则`CircuitBreakerOperator`向下游用户发出`CallNotPermittedException`。

```java
CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("name");
Mono.fromCallable(backendService::doSomething)
    .transformDeferred(CircuitBreakerOperator.of(circuitBreaker))
```

## 使用RateLimiter装饰Mono 或Flux

下面的示例演示如何使用自定义Reactor操作符装饰Mono。Flux也得到了支持。

`RateLimiterOperator`检查下游订阅者/观察者是否可以获得订阅上游发布者的权限。如果超过速率限制，`RateLimiterOperator`可以延迟从上游请求数据，也可以向下游订户发出R`equestNotAllowed`错误。

```java
RateLimiter rateLimiter = RateLimiter.ofDefaults("name");
Mono.fromCallable(backendService::doSomething)
    .transformDeferred(RateLimiterOperator.of(rateLimiter))
```

## 使用Bulkhead装饰Mono 或Flux

下面的示例演示如何使用自定义Reactor操作符装饰Mono。通量也得到了支持。

`BulkheadOperator`检查下游订阅者/观察者是否可以获得订阅上游发布者的权限。如果bulkhead是满的，``BulkheadOperatr`向下游订户发出`BulkheadFullException`。

```java
Bulkhead bulkhead = Bulkhead.ofDefaults("name");
Mono.fromCallable(backendService::doSomething)
  .transformDeferred(BulkheadOperator.of(bulkhead));
```

## 使用Retry装饰Mono 或Flux

```java
Retry retry = Retry.ofDefaults("backendName");
Mono.fromCallable(backendService::doSomething)
    .transformDeferred(RetryOperator.of(retry))
```

