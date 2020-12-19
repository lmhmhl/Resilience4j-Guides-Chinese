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

目录

介绍

和Netflix Hystrix的比较

Maven

Gradle

核心模块

断路器

隔离

限流器

重试

限时

缓存