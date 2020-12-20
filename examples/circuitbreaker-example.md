# 例子

R4j断路器的例子

## 创建 CircuitBreakerRegistry

使用自定义的CircuitBreakerConfig创建一个CircuitBreakerRegistry。

```java
// 为断路器创建自定义的配置
CircuitBreakerConfig circuitBreakerConfig = CircuitBreakerConfig.custom()
    .failureRateThreshold(50)
    .waitDurationInOpenState(Duration.ofMillis(1000))
    .permittedNumberOfCallsInHalfOpenState(2)
    .slidingWindowSize(2)
    .recordExceptions(IOException.class, TimeoutException.class)
    .ignoreExceptions(BusinessException.class, OtherBusinessException.class)
    .build();

// 使用自定义的全局配置创建CircuitBreakerRegistry
CircuitBreakerRegistry circuitBreakerRegistry =
  CircuitBreakerRegistry.of(circuitBreakerConfig);
```



## 创建CircuitBreaker

使用CircuitBreakerRegistry创建CircuitBreaker，使用全局默认的配置。

```java
CircuitBreaker circuitBreaker = circuitBreakerRegistry
  .circuitBreaker("name");
```



## 装饰函数接口

使用断路器对后台服务`BackendService.doSomething()`进行装饰，并且执行被装饰的supplier方法，从异常中恢复。

```java
Supplier<String> decoratedSupplier = CircuitBreaker
    .decorateSupplier(circuitBreaker, backendService::doSomething);

String result = Try.ofSupplier(decoratedSupplier)
    .recover(throwable -> "Hello from Recovery").get();
```



## 执行被装饰的函数接口

当你不想对lambda表达式进行装饰，只是想执行并被断路器保护。

```java
String result = circuitBreaker
  .executeSupplier(backendService::doSomething);
```



## 从异常中恢复

如果你想从任何被断路器记录为失败的异常中恢复，可以使用`Try.recover()`方法。只有在`Try.of()`返回`Failure<Throwable>`函子时，recover方法才会被调用。

```java
// Given
CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("testName");

// 对函数进行装饰并调用被装饰的方法
CheckedFunction0<String> checkedSupplier =
  CircuitBreaker.decorateCheckedSupplier(circuitBreaker, () -> {
    throw new RuntimeException("BAM!");
});
Try<String> result = Try.of(checkedSupplier)
        .recover(throwable -> "Hello Recovery");

// 应该是success，因为从异常中恢复了
assertThat(result.isSuccess()).isTrue();
// 结果必须和recover方法返回值匹配
assertThat(result.get()).isEqualTo("Hello Recovery");
```

如果你想在断路器把此次调用的结果诊断为异常之前进行恢复，你应该这样做:

```java
Java

Supplier<String> supplier = () -> {
            throw new RuntimeException("BAM!");
        };

Supplier<String> supplierWithRecovery = SupplierUtils
  .recover(supplier, (exception) -> "Hello Recovery");

String result = circuitBreaker.executeSupplier(supplierWithRecovery);

assertThat(result).isEqualTo("Hello Recovery")
```

`SupplierUtils`和`CallableUtils`包含一些其他方法，例如`andThen`，可用于链接函数。例如，用于检查HTTP响应的状态码，以便引发异常。

```java
Supplier<String> supplierWithResultAndExceptionHandler = SupplierUtils
  .andThen(supplier, (result, exception) -> "Hello Recovery");

Supplier<HttpResponse> supplier = () -> httpClient.doRemoteCall();
Supplier<HttpResponse> supplierWithResultHandling = SupplierUtils.andThen(supplier, result -> {
    if (result.getStatusCode() == 400) {
       throw new ClientException();
    } else if (result.getStatusCode() == 500) {
       throw new ServerException();
    }
    return result;
});
HttpResponse httpResponse = circuitBreaker
  .executeSupplier(supplierWithResultHandling);
```



## 重置断路器

断路器支持重置，能恢复原始的状态，会失去所有的统计值，这能够有效的重置滑动窗口。

```java
CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("testName");
circuitBreaker.reset();
```



## 手动转换状态

```java
CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("testName");
circuitBreaker.transitionToDisabledState();
// circuitBreaker.onFailure(...) 不会触发状态转换
//将转换到关闭状态并重新启用正常行为，保持指标
circuitBreaker.transitionToClosedState(); 
// 将从关闭状态转换到强制开启状态，从而丢失统计信息
circuitBreaker.transitionToForcedOpenState();
// circuitBreaker.onSuccess(...) 不会触发状态转换
//reset后变为关闭状态，重新开始断路器正常行为
circuitBreaker.reset(); 
```



## 重写RegistryStore

可以通过自定义实现重写内存中的RegistryStore。例如，如果要使用一个缓存，该缓存会在一段时间后删除未使用的实例。

```java
CircuitBreakerRegistry circuitBreakerRegistry = CircuitBreakerRegistry.custom()
  .withRegistryStore(new CacheCircuitBreakerRegistryStore())
  .build();
```

