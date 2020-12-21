# 重试

开始Resilience4j-retry模块的学习



## 创建 RetryRegistry

就像断路器模块一样，这么模块提供了在内存中的`RetryRegistry`，你可以使用这个管理（创建和检索）Retry实例。

```java
RetryRegistry retryRegistry = RetryRegistry.ofDefaults();
```



## 创建和配置重试

你可以提供一个自定义的全局RetryConfig，为了创建一个自定义的全局RetryConfig，可以使用建造者模式对RetryConfig进行配置。

- 最大的重试次数 。
- 连续两次重试之间的时间间隔。
- 自定义断言机制，评估一个响应是否可以触发重试机制。
- 自定义断言机制，评估一个异常是否可以出发重试机制。
- 定义一个异常的列表，这些异常能够触发重试机制。
- 定义一个异常的列表，这些异常应该被忽略并且不会触发重试机制。

| 属性                      | 默认值                        | 描述                                                         |
| ------------------------- | ----------------------------- | ------------------------------------------------------------ |
| maxAttempts               | 3                             | 最大重试次数                                                 |
| waitDuration              | 500 [ms]                      | 两次重试之间的时间间隔                                       |
| intervalFunction          | numOfAttempts -> waitDuration | 修改重试间隔的函数。默认情况下，等待时间保持不变。           |
| retryOnResultPredicate    | result -> false               | 配置用于计算是否应重试的断言。如果要重试，断言必须返回true，否则返回false。 |
| retryOnExceptionPredicate | throwable -> true             | 配置一个断言，判断某个异常发生时，是否要进行重试。如果要重试，断言必须返回true，否则必须返回false。 |
| retryExceptions           | empty                         | 配置一个Throwable类型的列表，被记录为失败类型，需要进行重试，支持子类型。 |
| ignoreExceptions          | empty                         | 配置一个Throwable类型的列表，被记录为忽略类型，不会进行重试，支持子类型。 |

```java
RetryConfig config = RetryConfig.custom()
  .maxAttempts(2)
  .waitDuration(Duration.ofMillis(1000))
  .retryOnResult(response -> response.getStatus() == 500)
  .retryOnException(e -> e instanceof WebServiceException)
  .retryExceptions(IOException.class, TimeoutException.class)
  .ignoreExceptions(BusinessException.class, OtherBusinessException.class)
  .build();

// 使用自定义的配置创建RetryRegistry
RetryRegistry registry = RetryRegistry.of(config);

// 使用默认的配置从Registry中获取和创建一个Retry
Retry retryWithDefaultConfig = registry.retry("name1");

// 使用自定义的配置从Registry中获取和创建一个Retry
RetryConfig custom = RetryConfig.custom()
    .waitDuration(Duration.ofMillis(100))
    .build();

Retry retryWithCustomConfig = registry.retry("name2", custom);
```



## 装饰和执行函数式接口

正如你所想的一样，重试有所有高阶装饰函数，就像断路器一样。你可以使用Retry对`Callable`, `Supplier`, `Runnable`, `Consumer`, `CheckedRunnable`, `CheckedSupplier`, `CheckedConsumer` 或者`CompletionStage`进行装饰。

```java
// 假设有一个HellowWorldService并抛出异常
HelloWorldService  helloWorldService = mock(HelloWorldService.class);
given(helloWorldService.sayHelloWorld())
  .willThrow(new WebServiceException("BAM!"));

// 使用默认的配置创建重试
Retry retry = Retry.ofDefaults("id");
// 使用Retry对HelloWorldService进行装饰
CheckedFunction0<String> retryableSupplier = Retry
  .decorateCheckedSupplier(retry, helloWorldService::sayHelloWorld);

// 进行方法调用
Try<String> result = Try.of(retryableSupplier)
  .recover((throwable) -> "Hello world from recovery function");

// helloWorldService的sayHelloWorld()方法将被调用3次
BDDMockito.then(helloWorldService).should(times(3)).sayHelloWorld();
// 异常应该被recovery方法处理
assertThat(result.get()).isEqualTo("Hello world from recovery function");
```



## 处理发出的RegistryEvent事件

你可以注册事件的处理者，并且在Retry创建、替换和删除时进行响应。



```java
RetryRegistry registry = RetryRegistry.ofDefaults();
registry.getEventPublisher()
  .onEntryAdded(entryAddedEvent -> {
    Retry addedRetry = entryAddedEvent.getAddedEntry();
    LOG.info("Retry {} added", addedRetry.getName());
  })
  .onEntryRemoved(entryRemovedEvent -> {
    Retry removedRetry = entryRemovedEvent.getRemovedEntry();
    LOG.info("Retry {} removed", removedRetry.getName());
  })
```



## 使用自定义的IntervalFunction

如果不想在重试之间使用固定的等待时间，可以配置IntervalFunction，该函数用于计算每次尝试的等待时间。Resilience4j提供了几种工厂方法来简化IntervalFunction的创建。

```java
IntervalFunction defaultWaitInterval = IntervalFunction
  .ofDefaults();

// 当您只配置waitDuration时,此间隔函数在内部使用。
IntervalFunction fixedWaitInterval = IntervalFunction
  .of(Duration.ofSeconds(5));

IntervalFunction intervalWithExponentialBackoff = IntervalFunction
  .ofExponentialBackoff();

IntervalFunction intervalWithCustomExponentialBackoff = IntervalFunction
  .ofExponentialBackoff(IntervalFunction.DEFAULT_INITIAL_INTERVAL, 2d);

IntervalFunction randomWaitInterval = IntervalFunction
  .ofRandomized();

// 用自定义的intervalFunction覆盖默认的intervalFunction
RetryConfig retryConfig = RetryConfig.custom()
  .intervalFunction(intervalWithExponentialBackoff)
  .build();
```

