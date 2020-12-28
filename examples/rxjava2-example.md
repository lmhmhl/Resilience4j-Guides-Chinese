# 例子

## 使用CircuitBreaker装饰Flowable

下面的例子展示了如何使用自定义RxJava操作符来修饰Observable。还支持所有其他反应类型，如	`Observable`, `Flowable`, `Single`, `Maybe `和` Completable`。

`CircuitBreakerOperator`检查下游订阅者/观察者是否可以获得订阅上游发布者的权限。如果断路器开启，则`CircuitBreakerOperator`向下游用户发出`CallNotPermittedException`。

```java
CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("name");
Flowable.fromCallable(backendService::doSomething)
    .compose(CircuitBreakerOperator.of(circuitBreaker))
```



## 使用RateLimiter装饰Flowable

下面的示例演示如何使用自定义RxJava2操作符来修饰Flowable。
还支持所有其他反应类型，如`Observable`, `Flowable`, `Single`, `Maybe `和` Completable`。

`RateLimiterOperator`检查下游订阅者/观察者是否可以获得订阅上游发布者的权限。如果超过速率限制，RateLimiterOperator将向下游订阅服务器发出一个`RequestNotPermitted`。

```java
RateLimiter rateLimiter = RateLimiter.ofDefaults("name");
Flowable.fromCallable(backendService::doSomething)
    .compose(RateLimiterOperator.of(rateLimiter))
```



## 使用Bulkhead装饰Flowable

下面的示例演示如何使用自定义RxJava2操作符来修饰Flowable。
还支持所有其他反应类型，如`Observable`, `Flowable`, `Single`, `Maybe `和` Completable`。

`BulkheadOperator`检查下游订阅者/观察者是否可以获得订阅上游发布者的权限。如果bulkhead是满的，`BulkheadOperator`向下游订户发出`BulkheadFullException`。

```java
Bulkhead bulkhead = Bulkhead.ofDefaults("name");
Flowable.fromCallable(backendService::doSomething)
  .compose(BulkheadOperator.of(bulkhead));
```



## 使用Retry装饰Flowable

```java
Retry retry = Retry.ofDefaults("backendName");
Flowable.fromCallable(backendService::doSomething)
    .compose(RetryOperator.of(retry))
```

