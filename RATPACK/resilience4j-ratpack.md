# resilience4j-ratpack



## 设置

添加Resilience4j的Ratpack的依赖。

Groovy

```groovy
repositories {
    jCenter()
}

dependencies {
    compile "io.github.resilience4j:resilience4j-ratpack:${resilience4jVersion}"
}
```

## 样例

Ratpack的设置和使用的 [demo](https://github.com/resilience4j/resilience4j-ratpack-demo)。

## 配置

你可以配置断路器、重试、限流器、隔离、线程池隔离，使用Ratpack的 `application.yml` 文件。

YAML

```yaml
resilience4j:
    circuitbreaker:
        instances:
            backendA:
                ringBufferSizeInClosedState: 100
            backendB:
                ringBufferSizeInClosedState: 10
                ringBufferSizeInHalfOpenState: 3
                waitDurationInOpenState: PT50S
                failureRateThreshold: 50
                eventConsumerBufferSize: 10
                recordFailurePredicate: io.github.robwin.exception.RecordFailurePredicate

    retry:
        instances:
            backendA:
                maxRetryAttempts: 3
                waitDuration: PT10S
                enableExponentialBackoff: true
                exponentialBackoffMultiplier: 2
                retryExceptions:
                    - java.io.IOException
                ignoreExceptions:
                    - io.github.robwin.exception.BusinessException
            backendB:
                maxRetryAttempts: 3
                waitDuration: PT10S
                retryExceptions:
                    - java.io.IOException
                ignoreExceptions:
                    - io.github.robwin.exception.BusinessException

    bulkhead:
        instances:
            backendA:
                maxConcurrentCall: 10
            backendB:
                maxWaitDuration: PT0.01S
                maxConcurrentCall: 20

    threadpoolbulkhead:
      instances:
        backendC:
          threadPoolProperties:
            maxThreadPoolSize: 1
            coreThreadPoolSize: 1
            queueCapacity: 1

    ratelimiter:
        instances:
            backendA:
                limitForPeriod: 10
                limitRefreshPeriod: PT1S
                timeoutDuration: 0
                eventConsumerBufferSize: 100
            backendB:
                limitForPeriod: 6
                limitRefreshPeriod: PT0.5S
                timeoutDuration: PT3S
```

您还可以覆盖默认配置、定义共享配置并在Ratpack中覆盖它们应用程序.yml配置文件。
例如：

YAML

```yaml
resilience4j:
    circuitbreaker:
        configs:
            default:
                ringBufferSizeInClosedState: 100
                ringBufferSizeInHalfOpenState: 10
                waitDurationInOpenState: 10000
                failureRateThreshold: 60
                eventConsumerBufferSize: 10
            someShared:
                ringBufferSizeInClosedState: 50
                ringBufferSizeInHalfOpenState: 10
        instances:
            backendA:
                baseConfig: default
                waitDurationInOpenState: 5000
            backendB:
                baseConfig: someShared
```

## 注解

Ratpack库提供自动配置的注释和AOP特性。
RateLimiter, Retry, CircuitBreaker 和 Bulkhead 注释支持同步返回类型和CompletableFuture等异步类型，以及Raptack的Promise和Spring Reactor的Flux和Mono等反应类型。

隔离注释有一个type属性决定使用哪种方式实现隔离，默认情况下它是信号量，但是如果您可以通过在注释中设置type属性来切换到线程池类型：

java

```java
@Bulkhead(name = BACKEND, type = Bulkhead.Type.THREADPOOL)
 public CompletableFuture<String> doSomethingAsync() throws InterruptedException {
        Thread.sleep(500);
        return CompletableFuture.completedFuture("Test");
    }
```

resilient4j支持的AOP方面的示例：

```java
@CircuitBreaker(name = BACKEND, fallbackMethod = "fallback")
@RateLimiter(name = BACKEND)
@Bulkhead(name = BACKEND)
@Retry(name = BACKEND, fallbackMethod = "fallback")
public Promise<String> method(String param1) {
   return Promise.error(new NumberFormatException());
 }

private Promise<String> fallback(String param1, IllegalArgumentException e) {
  return Promise.just("test");
}

private Promise<String> fallback(String param1, RuntimeException e) {
  return Promise.just("test");
}
```

降级方法应该放在同一个类中，并且必须具有相同的方法签名（和可选的异常参数）。

如果有多个fallbackMethod方法，则将调用最匹配的方法，例如：

如果尝试从`NumberFormatException`恢复，将调用签名字符串`String fallback(String parameter, IllegalArgumentException exception)}`的方法。

只有当多个方法具有相同的返回类型并且希望一次性为它们定义相同的回退方法时，才可以使用异常参数定义一个全局回退方法。

## Events Endpoint

发出的断路器、重试、限流器、隔离事件存储在一个单独的循环事件消费缓冲区中。事件使用者缓冲区的大小可以在应用程序.yml文件（eventConsumerBufferSize）。

 `/circuitbreaker/states` 列出所有断路器实例的状态。`/circuitbreaker/events`列出所有断路器实例的事件列表。事件端点还可用于重试、速率限制和隔离。

Text

```text
{
"circuitBreakerEvents":[
  {
    "circuitBreakerName": "backendA",
    "type": "ERROR",
    "creationTime": "2019-01-10T15:39:17.117-05:00[America/Chicago]",
    "errorMessage": "java.io.IOException: 500 This is a remote exception",
    "durationInMs": 0
  },
  {
    "circuitBreakerName": "backendA",
    "type": "SUCCESS",
    "creationTime": "2019-01-10T15:39:20.518-05:00[America/Chicago]",
    "durationInMs": 0
  },
  {
    "circuitBreakerName": "backendB",
    "type": "ERROR",
    "creationTime": "2019-01-10T15:41:31.159-05:00[America/Chicago]",
    "errorMessage": "java.io.IOException: 500 This is a remote exception",
    "durationInMs": 0
  },
  {
    "circuitBreakerName": "backendB",
    "type": "SUCCESS",
    "creationTime": "2019-01-10T15:41:33.526-05:00[America/Chicago]",
    "durationInMs": 0
  }
]
}
```