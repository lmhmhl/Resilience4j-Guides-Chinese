# Micrometer





Resilience4j提供了 [Micrometer](http://micrometer.io/) 提供了当下最流行的监控系统例如influxDB和Promethus。在运行时依赖 `micrometer-core`。

Groovy

```groovy
repositories {
    jCenter()
}

dependencies {
    compile "io.github.resilience4j:resilience4j-micrometer:${resilience4jVersion}"
}
```

## CircuitBreaker Metrics

下面的代码片段显示了如何将CircuitBreaker指标绑定到`MeterRegistry`。它将CircuitBreakerRegistry中的所有实例全部绑定到MeterRegistry。并且有新加入的实例会动态绑定。

Java

```java
MeterRegistry meterRegistry = new SimpleMeterRegistry();
CircuitBreakerRegistry circuitBreakerRegistry =
  CircuitBreakerRegistry.ofDefaults();
CircuitBreaker foo = circuitBreakerRegistry
  .circuitBreaker("backendA");
CircuitBreaker boo = circuitBreakerRegistry
  .circuitBreaker("backendB");

TaggedCircuitBreakerMetrics
  .ofCircuitBreakerRegistry(circuitBreakerRegistry)
  .bindTo(meterRegistry)
```

指标：

| 指标名称                                         | 类型                            | 标签                                                         | 描述                                       |
| :----------------------------------------------- | :------------------------------ | :----------------------------------------------------------- | :----------------------------------------- |
| resilience4j .circuitbreaker .calls              | Timer                           | One of the following: *kind="failed" *kind="successful" * kind="ignored"  name="backendA" | 成功、失败或忽略的调用总数                 |
| resilience4j .circuitbreaker .max.buffered.calls | Gauge                           | name="backendA"                                              | 当前环形缓冲区中可以存储的最大缓冲调用数   |
| resilience4j .circuitbreaker .state              | Gauge 0 - Not active 1 - Active | One of the following: *state="closed" *state="open" *state="half_open" *state="forced_open" * state="disabled"  name="backendA" | 断路器的状态                               |
| resilience4j .circuitbreaker .failure.rate       | Gauge                           | name="backendA"                                              | 断路器故障率                               |
| resilience4j .circuitbreaker .buffered.calls     | Gauge                           | One of the following: *kind="failed" *kind="successful"  name="backendA" | 存储在环形缓冲区中的缓冲成功和失败的调用数 |
| resilience4 .circuitbreaker .not.permitted.calls | Counter                         | kind="not_permitted"  name="backendA"                        | 被断路器拒绝的调用总数。                   |
| resilience4j .circuitbreaker .slow.call.rate     | Gauge                           | name="backendA"                                              | 慢调用率                                   |



## Retry Metrics

下面的代码片段显示了如何将Retry指标绑定到`MeterRegistry`。它将RetryRegistry中的所有实例全部绑定到MeterRegistry。并且有新加入的实例会动态绑定。

Java

```java
MeterRegistry meterRegistry = new SimpleMeterRegistry();
RetryRegistry retryRegistry = RetryRegistry.ofDefaults();
Retry retry = retryRegistry.retry("backendA");

// 一次注册所有的重试实例
TaggedRetryMetrics
  .ofRetryRegistry(retryRegistry)
  .bindTo(meterRegistry);
```

指标:

| 指标名称                   | 类型  | 标签                                                         | 描述               |
| :------------------------- | :---- | :----------------------------------------------------------- | :----------------- |
| resilience4j .retry .calls | Gauge | One of the following: *kind="successful.without.retry" *kind="successful.with.retry" *kind="failed.with.retry" *kind="failed.without.retry"  name="backendA" | 按种类列出的调用数 |

## Bulkhead Metrics

下面的代码片段显示了如何将Bulkhead指标绑定到`MeterRegistry`。它将BulkheadRegistry中的所有实例全部绑定到MeterRegistry。并且有新加入的实例会动态绑定。

Java

```java
MeterRegistry meterRegistry = new SimpleMeterRegistry();
BulkheadRegistry bulkheadRegistry = BulkheadRegistry.ofDefaults();
Bulkhead bulkhead = bulkheadRegistry.bulkhead("backendA");

// 一次性注册所有的Bulkhead实例
TaggedBulkheadMetrics
  .ofBulkheadRegistry(bulkheadRegistry)
  .bindTo(meterRegistry);
```

指标：

| 指标名称                                              | 类型  | 标签            | 描述               |
| :---------------------------------------------------- | :---- | :-------------- | :----------------- |
| resilience4j .bulkhead .available .concurrent.calls   | Gauge | name="backendA" | 可用权限的数目     |
| resilience4j .bulkhead .max.allowed .concurrent.calls | Gauge | name="backendA" | 可用权限的最大数目 |

## RateLimiter Metrics

下面的代码片段显示了如何将RateLimiter指标绑定到`MeterRegistry`。它将RateLimiterRegistry中的所有实例全部绑定到MeterRegistry。并且有新加入的实例会动态绑定。

Java

```java
MeterRegistry meterRegistry = new SimpleMeterRegistry();
RateLimiterRegistry rateLimiterRegistry = RateLimiterRegistry.ofDefaults();
RateLimiter rateLimiter = rateLimiterRegistry
  .rateLimiter("backendA");

// 一次性注册所有的RateLimiter实例
TaggedRateLimiterMetrics
  .ofRateLimiterRegistry(rateLimiterRegistry)
  .bindTo(meterRegistry);
```

指标:

| 指标名称                                        | 类型  | 标签            | 描述           |
| :---------------------------------------------- | :---- | :-------------- | :------------- |
| resilience4j.ratelimiter .available.permissions | Gauge | name="backendA" | 可用权限的数目 |
| resilience4j.ratelimiter .waiting.threads       | Gauge | name="backendA" | 等待的线程数量 |

## TimeLimiter

下面的代码片段显示了如何将TimeLimiter指标绑定到`MeterRegistry`。它将TimeLimiterRegistry中的所有实例全部绑定到MeterRegistry。并且有新加入的实例会动态绑定。

Java

```java
MeterRegistry meterRegistry = new SimpleMeterRegistry();
TimeLimiterRegistry timeLimiterRegistry = TimeLimiterRegistry.ofDefaults();
TimeLimiter timeLimiter = timeLimiterRegistry
  .timeLimiter("backendA");

// 一次性注册所有的TimeLimiter实例
TaggedTimeLimiterMetrics
  .ofTimeLimiterRegistry(timeLimiterRegistry)
  .bindTo(meterRegistry);
```

指标：

| 指标名称                       | 类型    | 标签                              | 描述               |
| :----------------------------- | :------ | :-------------------------------- | :----------------- |
| resilience4j.timelimiter.calls | Counter | name="backendA" kind="successful" | 成功的调用总数     |
| resilience4j.timelimiter.calls | Counter | name="backendA" kind="failed"     | 失败的调用总数     |
| resilience4j.timelimiter.calls | Counter | name="backendA" kind="timeout"    | 发生超时的调用总数 |

## Prometheus

当你想要在Prometheus上进行发布，你需要添加下面的依赖。

Groovy

```groovy
dependencies {
    compile "io.micrometer:micrometer-registry-prometheus"
}
```

每个断路器导出以下指标：

Text

```text
# HELP resilience4j_circuitbreaker_buffered_calls The number of buffered failed calls stored in the ring buffer
# TYPE resilience4j_circuitbreaker_buffered_calls gauge
resilience4j_circuitbreaker_buffered_calls{kind="failed",name="backendA",} 0.0
resilience4j_circuitbreaker_buffered_calls{kind="successful",name="backendA",} 0.0

# HELP resilience4j_circuitbreaker_calls_total Total number of not permitted calls
# TYPE resilience4j_circuitbreaker_calls_total counter
resilience4j_circuitbreaker_calls_total{kind="not_permitted",name="backendA",} 0.0

# HELP resilience4j_circuitbreaker_state The states of the circuit breaker
# TYPE resilience4j_circuitbreaker_state gauge
resilience4j_circuitbreaker_state{name="backendA",state="half_open",} 0.0
resilience4j_circuitbreaker_state{name="backendA",state="forced_open",} 0.0
resilience4j_circuitbreaker_state{name="backendA",state="disabled",} 0.0
resilience4j_circuitbreaker_state{name="backendA",state="closed",} 1.0
resilience4j_circuitbreaker_state{name="backendA",state="open",} 0.0
resilience4j_circuitbreaker_state{name="backendA",} 0.0

# HELP resilience4j_circuitbreaker_failure_rate The failure rate of the circuit breaker
# TYPE resilience4j_circuitbreaker_failure_rate gauge
resilience4j_circuitbreaker_failure_rate{name="backendA",} 20.0

# HELP resilience4j_circuitbreaker_max_buffered_calls The maximum number of buffered calls which can be stored in the ring buffer
# TYPE resilience4j_circuitbreaker_max_buffered_calls gauge
resilience4j_circuitbreaker_max_buffered_calls{name="backendA",} 5.0

# HELP resilience4j_circuitbreaker_calls_seconds_max Total duration of calls
# TYPE resilience4j_circuitbreaker_calls_seconds_max gauge
resilience4j_circuitbreaker_calls_seconds_max{kind="successful",name="backendA",} 0.0
resilience4j_circuitbreaker_calls_seconds_max{kind="failed",name="backendA",} 0.0
resilience4j_circuitbreaker_calls_seconds_max{kind="ignored",name="backendA",} 0.0

resilience4j_circuitbreaker_calls_seconds_sum{kind="ignored",name="backendA",} 0.0

# HELP resilience4j_circuitbreaker_calls_seconds Total number of successful calls
# TYPE resilience4j_circuitbreaker_calls_seconds histogram
resilience4j_circuitbreaker_calls_seconds_count{kind="successful",name="backendA",} 0.0
resilience4j_circuitbreaker_calls_seconds_sum{kind="successful",name="backendA",} 0.0

# HELP resilience4j_circuitbreaker_calls_seconds Total number of failed calls
# TYPE resilience4j_circuitbreaker_calls_seconds histogram
resilience4j_circuitbreaker_calls_seconds_count{kind="failed",name="backendA",} 0.0
resilience4j_circuitbreaker_calls_seconds_sum{kind="failed",name="backendA",} 0.0
```

每个bulkhead导出以下指标

Text

```text
# HELP resilience4j_bulkhead_available_concurrent_calls The number of available permissions
# TYPE resilience4j_bulkhead_available_concurrent_calls gauge
resilience4j_bulkhead_available_concurrent_calls{name="backendA",} 10.0

# HELP resilience4j_bulkhead_max_allowed_concurrent_calls The maximum number available permissions
# TYPE resilience4j_bulkhead_max_allowed_concurrent_calls gauge
resilience4j_bulkhead_max_allowed_concurrent_calls{name="backendA",} 10.0
```

每个Retry导出以下指标：

Text

```text
# HELP resilience4j_retry_calls The number of successful calls without a retry attempt
# TYPE resilience4j_retry_calls gauge
resilience4j_retry_calls{kind="failed_with_retry",name="backendA",} 0.0
resilience4j_retry_calls{kind="failed_without_retry",name="backendA",} 0.0
resilience4j_retry_calls{kind="successful_without_retry",name="backendA",} 0.0
resilience4j_retry_calls{kind="successful_with_retry",name="backendA",} 0.0
```

每个Ratelimiter导出以下指标

Text

```text
# HELP resilience4j_ratelimiter_waiting_threads The number of waiting threads
# TYPE resilience4j_ratelimiter_waiting_threads gauge
resilience4j_ratelimiter_waiting_threads{name="backendA",} 0.0

# HELP resilience4j_ratelimiter_available_permissions The number of available permissions
# TYPE resilience4j_ratelimiter_available_permissions gauge
resilience4j_ratelimiter_available_permissions{name="backendA",} 50.0
```