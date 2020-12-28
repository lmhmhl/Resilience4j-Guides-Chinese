## Resilience4j 是一个Java™的容错框架

Resilience4j是一个轻量级容错框架，设计灵感来源于Netflix 的Hystrix框架，为函数式编程所设计。

Resilience4j 提供了一组高阶函数（装饰器），包括断路器，限流器，重试，隔离，可以对任何的函数式接口，lambda表达式，或方法的引用进行增强，并且这些装饰器可以进行叠加。这样做的好处是，你可以根据需要选择特定的装饰器进行组合。

```java
Supplier<String> supplier = () -> service.sayHelloWorld(param1);

String result = Decorators.ofSupplier(supplier)
  .withBulkhead(Bulkhead.ofDefaults("name"))
  .withCircuitBreaker(CircuitBreaker.ofDefaults("name"))
  .withRetry(Retry.ofDefaults("name"))
  .withFallback(asList(CallNotPermittedException.class, BulkheadFullException.class),  
      throwable -> "Hello from fallback")
  .get()

```

在使用时，你不需要引入所有和Resilience4j相关的包，只需要引入所需要的即可。

## 目录

### 开始

* [介绍](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/getting-start/Introduction.md)

* [和Netflix Hystrix的比较](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/getting-start/Comparison-to-Netflix-Hystrix.md)

* [Maven](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/getting-start/Maven.md)

* [Gradle](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/getting-start/Gradle.md)

### 核心模块

* [断路器](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/core-modules/CircuitBreaker.md)
  * [例子](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/examples/circuitbreaker-example.md)

* [隔离](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/core-modules/bulkhead.md)
  * [例子](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/examples/bulkhead-example.md)

* [限流器](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/core-modules/ratelimiter.md)
  * [例子](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/examples/ratelimiter-example.md)

* [重试](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/core-modules/retry.md)
  * [例子](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/examples/retry-example.md)

* [限时](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/core-modules/timelimiter.md)

* [缓存](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/core-modules/cache.md)

### 附加模块

* [Kotlin](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/addon-module/Kotlin.md)
* [Feign](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/addon-module/Feign.md)
* [Retrofit](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/addon-module/Retrofit.md)

### RXJAVA2

* [resilience4j-RXJAVA](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/RXJAVA2/resilience4j-rxjava2.md)
  * [例子](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/examples/rxjava2-example.md)

### Spring Reactor

* [resilience4j-reactor](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/SPRINGREACTOR/resilience4j-reactor.md)
  * [例子](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/examples/springreactor-example.md)

### SPRING BOOT 2

* [resilience4j-spring-boot2](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/SPRINGBOOT2/resilience4j-spring-boot2.md)

### SPRING CLOUD

* [resilience4j-spring-cloud2](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/SPRINGCLOUD/Spring%20Cloud.md)

### RATPACK

* [resilience4j-ratpack](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/RATPACK/resilience4j-ratpack.md)

### METRICS

* [resilience4j-micrometer](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/MERTRIC/Micrometer.md)
* [Grafana](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/MERTRIC/Grafana.md)



