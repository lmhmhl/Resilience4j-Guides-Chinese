# Feign

一个Java到HTTP客户机绑定器，灵感来自于Retrofit、JAXRS-2.0和WebSocket。https://github.com/OpenFeign/feign

Resilience4j作为Feign的装饰器。类似https://github.com/OpenFeign/feign/tree/master/hystrix[HystrixFeign]，resilience4j-feign可以很容易地将“容错”模式纳入feign框架，例如断路器和速率限制器。

## 目前的特性

- 断路器
- 限流器
- 服务降级

## Decorating Feign Interfaces

 `Resilience4jFeign.builder` 是创建feign容错实例的主要的类，
它扩展了 `Feign.builder` 并且可以使用添加自定义的`InvocationHandlerFactory`进行配置。Resilience4jFeign 使用了自己的`InvocationHandlerFactory` 作为装饰器。装饰器可以使用`FeignDecorators` 进行构建。 多个装饰器可以进行组合。
下面的例子说明了怎样使用限流器和断路器装饰feign接口：

```java
public interface MyService {
            @RequestLine("GET /greeting")
            String getGreeting();
            
            @RequestLine("POST /greeting")
            String createGreeting();
        }

        CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("backendName");
        RateLimiter rateLimiter = RateLimiter.ofDefaults("backendName");
        FeignDecorators decorators = FeignDecorators.builder()
                                         .withRateLimiter(rateLimiter)
                                         .withCircuitBreaker(circuitBreaker)
                                         .build();
        MyService myService = Resilience4jFeign.builder(decorators).target(MyService.class, "http://localhost:8080/");
```

调用`MyService`实例的任何方法都将调用断路器，然后是限流器。
如果其中一个机制生效，则会抛出相应的RuntimeException，例如，`CircuitBreakerOpenException`或`RequestNotPermitted`（提示：这些机制不会扩展`FeignException`类）。



##  装饰器的顺序

装饰器的使用顺序和声明它们的顺序相对应。当构建`FeignDecorators`时，必须对此保持警惕，因为顺序会影响结果行为。

```java
FeignDecorators decoratorsA = FeignDecorators.builder()
                                         .withCircuitBreaker(circuitBreaker)
                                         .withRateLimiter(rateLimiter)
                                         .build();
                                         
        FeignDecorators decoratorsB = FeignDecorators.builder()
                                         .withRateLimiter(rateLimiter)
                                         .withCircuitBreaker(circuitBreaker)
                                         .build();
```

对于decoratorsA，限流器将在断路器之前被调用。这意味着即使断路器断开，限流器仍然会限制调用速率。decoratorsB应用相反的顺序。这意味着一旦断路器断开，速率限制器将不再起作用。

## 服务降级

可以定义在引发异常时调用的方法叫做服务降级方法。异常可能发生在HTTP请求失败时，但也可能发生在`FeignDecorators`之一激活时，例如，断路器。

```java
public interface MyService {
            @RequestLine("GET /greeting")
            String greeting();
        }

        MyService requestFailedFallback = () -> "fallback greeting";
        MyService circuitBreakerFallback = () -> "CircuitBreaker is open!";
        CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("backendName");
        FeignDecorators decorators = FeignDecorators.builder()
                                         .withFallback(requestFailedFallback, FeignException.class)
                                         .withFallback(circuitBreakerFallback, CircuitBreakerOpenException.class)
                                         .build();
        MyService myService = Resilience4jFeign.builder(decorators).target(MyService.class, "http://localhost:8080/", fallback);
```

在本例中，当抛出`FeignException`时（通常是在HTTP请求失败时）调用`requestFailedFallback`，而只有在`CircuitBreakerOpenException`的情况下才会调用`circuitBreakerFallback`。检查`FeignDecorators`类以获得更多降级的方法。

所有降级方法必须实现“目标”中声明的相同接口（Resilience4jFeign.Builder#target)方法，否则将引发IllegalArgumentException。

可以指定多个降级方法来处理同一个异常，当上一个降级方法调用失败时，将调用下一个降级方法。

如果需要，降级方法可以使用抛出的异常。如果降级方法可能根据异常有不同的行为，或者只是记录异常，那么这很有用。
请注意，对于抛出的每个异常，都将实例化这样的降级方法。

```java
public interface MyService {
            @RequestLine("GET /greeting")
            String greeting();
        }

        public class MyFallback implements MyService {
            private Exception cause;

            public MyFallback(Exception cause) {
                this.cause = cause;
            }

            public String greeting() {
                if (cause instanceOf FeignException) {
                    return "Feign Exception";
                } else {
                    return "Other exception";
                }
            }
        }

        FeignDecorators decorators = FeignDecorators.builder()
                .withFallbackFactory(MyFallback::new)
                .build();
```



