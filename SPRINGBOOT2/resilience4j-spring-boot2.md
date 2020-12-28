# resilience4j-spring-boot2



## 配置

添加Resilience4j的springboot2的依赖。

模块期望`org.springframework.boot:spring-boot-starter-actuator`和`org.springframework.boot:spring-boot-starter-aop`在运行时被提供。如果你使用webflux, 你也需要引入`io.github.resilience4j:resilience4j-reactor`。

Groovy

```groovy
repositories {
    jcenter()
}

dependencies {
  compile "io.github.resilience4j:resilience4j-spring-boot2:${resilience4jVersion}"
  compile('org.springframework.boot:spring-boot-starter-actuator')
  compile('org.springframework.boot:spring-boot-starter-aop')
}
```

## 样例

Spring Boot 2 设置和启动的例子。 [demo](https://github.com/resilience4j/resilience4j-spring-boot2-demo)

## 配置

您可以在springboot中配置断路器、重试、限流器、隔离、线程池隔离和限时器实例。使用`application.yml`进行配置。

YAML

```yaml
resilience4j.circuitbreaker:
    instances:
        backendA:
            registerHealthIndicator: true
            slidingWindowSize: 100
        backendB:
            registerHealthIndicator: true
            slidingWindowSize: 10
            permittedNumberOfCallsInHalfOpenState: 3
            slidingWindowType: TIME_BASED
            minimumNumberOfCalls: 20
            waitDurationInOpenState: 50s
            failureRateThreshold: 50
            eventConsumerBufferSize: 10
            recordFailurePredicate: io.github.robwin.exception.RecordFailurePredicate
            
resilience4j.retry:
    instances:
        backendA:
            maxRetryAttempts: 3
            waitDuration: 10s
            enableExponentialBackoff: true
            exponentialBackoffMultiplier: 2
            retryExceptions:
                - org.springframework.web.client.HttpServerErrorException
                - java.io.IOException
            ignoreExceptions:
                - io.github.robwin.exception.BusinessException
        backendB:
            maxRetryAttempts: 3
            waitDuration: 10s
            retryExceptions:
                - org.springframework.web.client.HttpServerErrorException
                - java.io.IOException
            ignoreExceptions:
                - io.github.robwin.exception.BusinessException
                
resilience4j.bulkhead:
    instances:
        backendA:
            maxConcurrentCalls: 10
        backendB:
            maxWaitDuration: 10ms
            maxConcurrentCalls: 20
            
resilience4j.thread-pool-bulkhead:
  instances:
    backendC:
      maxThreadPoolSize: 1
      coreThreadPoolSize: 1
      queueCapacity: 1
        
resilience4j.ratelimiter:
    instances:
        backendA:
            limitForPeriod: 10
            limitRefreshPeriod: 1s
            timeoutDuration: 0
            registerHealthIndicator: true
            eventConsumerBufferSize: 100
        backendB:
            limitForPeriod: 6
            limitRefreshPeriod: 500ms
            timeoutDuration: 3s
            
resilience4j.timelimiter:
    instances:
        backendA:
            timeoutDuration: 2s
            cancelRunningFuture: true
        backendB:
            timeoutDuration: 1s
            cancelRunningFuture: false
```

您还可以覆盖默认配置，定义共享配置并在springboot中覆盖它们应用程序`application.yml`配置文件。

YAML

```yaml
resilience4j.circuitbreaker:
    configs:
        default:
            slidingWindowSize: 100
            permittedNumberOfCallsInHalfOpenState: 10
            waitDurationInOpenState: 10000
            failureRateThreshold: 60
            eventConsumerBufferSize: 10
            registerHealthIndicator: true
        someShared:
            slidingWindowSize: 50
            permittedNumberOfCallsInHalfOpenState: 10
    instances:
        backendA:
            baseConfig: default
            waitDurationInOpenState: 5000
        backendB:
            baseConfig: someShared
```



Properties 文件。

```text
resilience4j.bulkhead.instances.backendA.maxConcurrentCalls=10

resilience4j.bulkhead.instances.backendB.maxWaitDuration=10ms
resilience4j.bulkhead.instances.backendB.maxConcurrentCalls=20

resilience4j.circuitbreaker.instances.backendA.registerHealthIndicator=true
resilience4j.circuitbreaker.instances.backendA.slidingWindowSize=100

resilience4j.circuitbreaker.instances.backendB.registerHealthIndicator=true
resilience4j.circuitbreaker.instances.backendB.slidingWindowSize=10
resilience4j.circuitbreaker.instances.backendB.permittedNumberOfCallsInHalfOpenState=3
resilience4j.circuitbreaker.instances.backendB.slidingWindowType=TIME_BASED
resilience4j.circuitbreaker.instances.backendB.minimumNumberOfCalls=20
resilience4j.circuitbreaker.instances.backendB.waitDurationInOpenState=50s
resilience4j.circuitbreaker.instances.backendB.failureRateThreshold=50
resilience4j.circuitbreaker.instances.backendB.eventConsumerBufferSize=10
resilience4j.circuitbreaker.instances.backendB.recordFailurePredicate=io.github.robwin.exception.RecordFailurePredicate

resilience4j.ratelimiter.instances.backendA.limitForPeriod=10
resilience4j.ratelimiter.instances.backendA.limitRefreshPeriod=1s
resilience4j.ratelimiter.instances.backendA.timeoutDuration=0
resilience4j.ratelimiter.instances.backendA.registerHealthIndicator=true
resilience4j.ratelimiter.instances.backendA.eventConsumerBufferSize=100

resilience4j.ratelimiter.instances.backendB.limitForPeriod=6
resilience4j.ratelimiter.instances.backendB.limitRefreshPeriod=500ms
resilience4j.ratelimiter.instances.backendB.timeoutDuration=3s

resilience4j.retry.instances.backendA.maxRetryAttempts=3
resilience4j.retry.instances.backendA.waitDuration=10s
resilience4j.retry.instances.backendA.enableExponentialBackoff=true
resilience4j.retry.instances.backendA.exponentialBackoffMultiplier=2
resilience4j.retry.instances.backendA.retryExceptions[0]=org.springframework.web.client.HttpServerErrorException
resilience4j.retry.instances.backendA.retryExceptions[1]=java.io.IOException
resilience4j.retry.instances.backendA.ignoreExceptions[0]=io.github.robwin.exception.BusinessException

resilience4j.retry.instances.backendB.maxRetryAttempts=3
resilience4j.retry.instances.backendB.waitDuration=10s
resilience4j.retry.instances.backendB.retryExceptions[0]=org.springframework.web.client.HttpServerErrorException
resilience4j.retry.instances.backendB.retryExceptions[1]=java.io.IOException
resilience4j.retry.instances.backendB.ignoreExceptions[0]=io.github.robwin.exception.BusinessException

resilience4j.thread-pool-bulkhead.instances.backendC.maxThreadPoolSize=1
resilience4j.thread-pool-bulkhead.instances.backendC.coreThreadPoolSize=1
resilience4j.thread-pool-bulkhead.instances.backendC.queueCapacity=1

resilience4j.timelimiter.instances.backendA.timeoutDuration=2s
resilience4j.timelimiter.instances.backendA.cancelRunningFuture=true

resilience4j.timelimiter.instances.backendB.timeoutDuration=1s
resilience4j.timelimiter.instances.backendB.cancelRunningFuture=false
```

## Override configuration via code

您还可以通过对特定实例名称使用Customizer来覆盖特定断路器、隔离、重试、限流器或限时器实例的配置。下面的示例演示了如何在配置的断路器backendA对上面的YAML文件进行覆盖。

Java

```java
@Bean
public CircuitBreakerConfigCustomizer testCustomizer() {
    return CircuitBreakerConfigCustomizer
        .of("backendA", builder -> builder.slidingWindowSize(100));
}
```

Resilience4j有自己的customizer类型，可以按上图所示使用：

| Resilienc4j类型    | 自定义类                           |
| :----------------- | :--------------------------------- |
| Circuit breaker    | CircuitBreakerConfigCustomizer     |
| Retry              | RetryConfigCustomizer              |
| Rate limiter       | RateLimiterConfigCustomizer        |
| Bulkhead           | BulkheadConfigCustomizer           |
| ThreadPoolBulkhead | ThreadPoolBulkheadConfigCustomizer |
| Time Limiter       | TimeLimiterConfigCustomizer        |

## 注解

Spring Boot2提供了AOP的注解可以进行自动配置。

限流器、重试、断路器和隔离注解支持同步返回类型和异步类型（如CompletableFuture）和反应类型（如Spring Reactor的`Flux`和`Mono`）（如果您导入了适当的包，如resilience4j Reactor）。

隔离注解有一个type属性来定义将使用哪个隔离机制实现。默认情况下，它是信号量，但您可以通过在注释中设置type属性来切换到线程池：

```java
@Bulkhead(name = BACKEND, type = Bulkhead.Type.THREADPOOL)
public CompletableFuture<String> doSomethingAsync() throws InterruptedException {
        Thread.sleep(500);
        return CompletableFuture.completedFuture("Test");
 }
```

resilience4j 支持AOP

Java

```java
@CircuitBreaker(name = BACKEND, fallbackMethod = "fallback")
@RateLimiter(name = BACKEND)
@Bulkhead(name = BACKEND)
@Retry(name = BACKEND, fallbackMethod = "fallback")
@TimeLimiter(name = BACKEND)
public Mono<String> method(String param1) {
    return Mono.error(new NumberFormatException());
}

private Mono<String> fallback(String param1, IllegalArgumentException e) {
    return Mono.just("test");
}

private Mono<String> fallback(String param1, RuntimeException e) {
    return Mono.just("test");
}
```

## 服务降级方法

所有的fallback方法应该放在同一个类中，并且必须具有相同的方法签名，并且只有一个额外的异常参数。

如果有多个降级方法，则将根据异常类型调用最匹配的方法，例如：

如果尝试从`NumberFormatException`恢复，将调用签名字符串String `fallback(String parameter, IllegalArgumentException exception)}`的方法。

只有当多个方法具有相同的返回类型并且只想为它们定义一次相同的降级方法时，才能使用异常参数定义一个全局回退方法。`



## 切面的顺序

Resilience4j的切面执行顺序：

```
Retry ( CircuitBreaker ( RateLimiter ( TimeLimiter ( Bulkhead ( Function ) ) ) ) )
```

所以`Retry` 是最后执行.
如果需要不同的顺序，则必须使用函数链接样式而不是Spring注释样式，或者使用以下属性显式设置方面顺序：

Text

```text
- resilience4j.retry.retryAspectOrder
- resilience4j.circuitbreaker.circuitBreakerAspectOrder
- resilience4j.ratelimiter.rateLimiterAspectOrder
- resilience4j.timelimiter.timeLimiterAspectOrder
```

例如要使断路器在重试完成其工作后启动，必须将`RetryAspectOrder`属性设置为比`circuitBreakerAspectOrder`值更大的值（更高的值=更高的优先级）。

YAML

```yaml
resilience4j:
  circuitbreaker:
    circuitBreakerAspectOrder: 1
  retry:
    retryAspectOrder: 2
```

## 注册事件消费者

您可以添加RegistryEventConsumer Bean，以便将事件使用者添加到新创建的实例中。例如，您可以将`RegistryEventConsumer`添加到`RetryRegistry`，以便将日志事件使用者注册到每个新创建的重试实例。

Java

```java
@Bean
public RegistryEventConsumer<Retry> myRetryRegistryEventConsumer() {

    return new RegistryEventConsumer<Retry>() {
        @Override
        public void onEntryAddedEvent(EntryAddedEvent<Retry> entryAddedEvent) {
            entryAddedEvent.getAddedEntry().getEventPublisher().onEvent(event -> LOG.info(event.toString()));
        }

        @Override
        public void onEntryRemovedEvent(EntryRemovedEvent<Retry> entryRemoveEvent) {

        }

        @Override
        public void onEntryReplacedEvent(EntryReplacedEvent<Retry> entryReplacedEvent) {

        }
    };
}
```

重试机制的日志。

Text

```text
2020-10-26T13:00:19.807034700+01:00[Europe/Berlin]: Retry 'backendA', waiting PT0.1S until attempt '1'. Last attempt failed with exception 'org.springframework.web.client.HttpServerErrorException: 500 This is a remote exception'.

2020-10-26T13:00:19.912028800+01:00[Europe/Berlin]: Retry 'backendA', waiting PT0.1S until attempt '2'. Last attempt failed with exception 'org.springframework.web.client.HttpServerErrorException: 500 This is a remote exception'.

2020-10-26T13:00:20.023250+01:00[Europe/Berlin]: Retry 'backendA' recorded a failed retry attempt. Number of retry attempts: '3'. Giving up. Last exception was: 'org.springframework.web.client.HttpServerErrorException: 500 This is a remote exception'.
```

## Metrics endpoint

断路器、重试、限流器、隔离和限时器度量将自动发布在度量端点上。要检索可用度量的名称，发送请求到`/actuator/metrics`。相关细节请看 [Actuator Metrics documentation](https://docs.spring.io/spring-boot/docs/current/actuator-api/html/#metrics) 。



JSON

```json
{
    "names": [
        "resilience4j.circuitbreaker.calls",
        "resilience4j.circuitbreaker.buffered.calls",
        "resilience4j.circuitbreaker.state",
        "resilience4j.circuitbreaker.failure.rate"
        ]
}
```

为了查看相关指标，发送请求`/actuator/metrics/{metric.name}`

例如: `/actuator/metrics/resilience4j.circuitbreaker.calls`

JSON

```json
{
    "name": "resilience4j.circuitbreaker.calls",
    "measurements": [
        {
            "statistic": "VALUE",
            "value": 3
        }
    ],
    "availableTags": [
        {
            "tag": "kind",
            "values": [
                "not_permitted",
                "successful",
                "failed"
            ]
        },
        {
            "tag": "name",
            "values": [
                "backendB",
                "backendA"
            ]
        }
    ]
}
```

如果你想要发步断路器到Prometheus平台，需要添加`io.micrometer:micrometer-registry-prometheus`依赖发送GET方法`/actuator/prometheus`来查看相关指标.  [Micrometer Getting Started](https://resilience4j.readme.io/docs/micrometer)



## Health endpoint

Spring Boot Actuator 运行状况信息可用于检查正在运行的应用程序的状态。当被保护的系统发生严重的问题是将发出警报。
默认情况下，断路器或限流器运行状况指示器处于禁用状态，但您可以通过配置启用它们。当断路器开启时，由于应用程序状态为DOWN，所以运行状况指示灯被禁用。这可能不是你想要的。

YAML

```yaml
management.health.circuitbreakers.enabled: true
management.health.ratelimiters.enabled: true

resilience4j.circuitbreaker:
  configs:
    default:
      registerHealthIndicator: true


resilience4j.ratelimiter:
  configs:
    default:
      registerHealthIndicator: true
```

关闭的断路器状态UP，开启状态为DOWN，半断开状态映射为UNKONWN。

例如：

JSON

```json
{
  "status": "UP",
  "details": {
    "circuitBreakers": {
      "status": "UP",
      "details": {
        "backendB": {
          "status": "UP",
          "details": {
            "failureRate": "-1.0%",
            "failureRateThreshold": "50.0%",
            "slowCallRate": "-1.0%",
            "slowCallRateThreshold": "100.0%",
            "bufferedCalls": 0,
            "slowCalls": 0,
            "slowFailedCalls": 0,
            "failedCalls": 0,
            "notPermittedCalls": 0,
            "state": "CLOSED"
          }
        },
        "backendA": {
          "status": "UP",
          "details": {
            "failureRate": "-1.0%",
            "failureRateThreshold": "50.0%",
            "slowCallRate": "-1.0%",
            "slowCallRateThreshold": "100.0%",
            "bufferedCalls": 0,
            "slowCalls": 0,
            "slowFailedCalls": 0,
            "failedCalls": 0,
            "notPermittedCalls": 0,
            "state": "CLOSED"
          }
        }
      }
    }
  }
}
```

## Events endpoint

发出的断路器、重试、限流器、隔离和时间限制器的事件存储在一个单独的循环事件消费者缓冲区中。事件使用者缓冲区的大小可以在应用程序.yml文件（eventConsumerBufferSize）。

`/actuator/circuitbreakers`列出所有断路器实例的名称。端点还可用于重试、限流器、隔离和时间限制器。
例如：

JSON

```json
{
    "circuitBreakers": [
      "backendA",
      "backendB"
    ]
}
```

默认情况下，`/actuator/circuitbreakerevents`列出所有断路器实例的最近100个已发出事件。端点还可用于重试、速率限制器、隔板和限时器。

JSON

```json
{
    "circuitBreakerEvents": [
        {
            "circuitBreakerName": "backendA",
            "type": "ERROR",
            "creationTime": "2017-01-10T15:39:17.117+01:00[Europe/Berlin]",
            "errorMessage": "org.springframework.web.client.HttpServerErrorException: 500 This is a remote exception",
            "durationInMs": 0
        },
        {
            "circuitBreakerName": "backendA",
            "type": "SUCCESS",
            "creationTime": "2017-01-10T15:39:20.518+01:00[Europe/Berlin]",
            "durationInMs": 0
        },
        {
            "circuitBreakerName": "backendB",
            "type": "ERROR",
            "creationTime": "2017-01-10T15:41:31.159+01:00[Europe/Berlin]",
            "errorMessage": "org.springframework.web.client.HttpServerErrorException: 500 This is a remote exception",
            "durationInMs": 0
        },
        {
            "circuitBreakerName": "backendB",
            "type": "SUCCESS",
            "creationTime": "2017-01-10T15:41:33.526+01:00[Europe/Berlin]",
            "durationInMs": 0
        }
    ]
}
```

## Event Streaming endpoint

将自动生成以下端点，并将事件生成为服务器发送事件（SSE）

cURL

```curl
/actuator/stream-circuitbreaker-events

/actuator/stream-circuitbreaker-events/{circuitbreakername}
 
/actuator/stream-circuitbreaker-events/{circuitbreakername}/{errorType}
```

## Hystrix Event Streaming endpoint

将自动生成以下端点，并将事件作为服务器发送事件（SSE）生成。

该SSE数据可以很容易地映射到hystrix兼容数据格式（特定的K V对），并可用于涡轮机或hystrix仪表板或vizceral中。作为hystrix SSE的一部分生成的数据正在用状态数据充实事件端点

这是一个桥梁，以支持传统的hystrix生态系统的监测工具，特别是那些从hystrix迁移到resilence4j继续使用hystrix生态工具的人。

cURL

```curl
/actuator/hystrix-stream-circuitbreaker-events

/actuator/hystrix-stream-circuitbreaker-events/{circuitbreakername}

/actuator/hystrix-stream-circuitbreaker-events/{circuitbreakername}/{errorType}
```