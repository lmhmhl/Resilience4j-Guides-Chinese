# 隔离机制

开始resilience4j-bulkhead的学习

## 介绍

Resilience4j提供了两种隔离的实现方式，可以限制并发执行的数量。

- `SemaphoreBulkhead`使用了信号量
- `FixedThreadPoolBulkhead`使用了有界队列和固定大小线程池

`SemaphoreBulkhead`可以在各种线程和I/O模型上正常工作。与Hystrix不同，它不提供基于shadow的thread选项。由客户端来确保正确的线程池大小与隔离配置一致。

## 创建BulkheadRegistry

就像CircuitBreaker模块，这个模块提供了一个内存中的`BulkheadRegistry`，和一个`ThreadPoolBulkheadRegistry`，你可以使用它们管理（创建和检索）隔离实例。

```java
BulkheadRegistry bulkheadRegistry = BulkheadRegistry.ofDefaults();

ThreadPoolBulkheadRegistry threadPoolBulkheadRegistry = 
  ThreadPoolBulkheadRegistry.ofDefaults();
```

## 创建和配置BulkHead

你可以提供一个自定义的全局BulkheadConfig，你可以使用BulkheadConfig建造者模式来创建自定义的全局BulkheadConfig，可以使用builder来配置下面的属性。

| 配置属性           | 默认值 | 描述                                                         |
| ------------------ | ------ | ------------------------------------------------------------ |
| maxConcurrentCalls | 25     | 隔离允许线程并发执行的最大数量                               |
| maxWaitDuration    | 0      | 当达到并发调用数量时，新的线程执行时将被阻塞，这个属性表示最长的等待时间。 |

```java
//为Bulkhead创建自定义的配置
BulkheadConfig config = BulkheadConfig.custom()
    .maxConcurrentCalls(150)
    .maxWaitDuration(Duration.ofMillis(500))
    .build();

// 使用自定义全局配置创建BulkheadRegistry
BulkheadRegistry registry = BulkheadRegistry.of(config);

// 使用默认的配置从registry中创建Bulkhead
Bulkhead bulkheadWithDefaultConfig = registry.bulkhead("name1");

// 使用自定义的配置从regidtry中创建bulkhead
Bulkhead bulkheadWithCustomConfig = registry.bulkhead("name2", custom);
```



## 创建和执行函数式接口

正如你所猜的一样，和断路器相似，Bulkhead可以作为装饰器为函数式接口进行装饰。你可以使用bulkhead装饰任意的`Callable`, `Supplier`, `Runnable`, `Consumer`, `CheckedRunnable`, `CheckedSupplier`, `CheckedConsumer` 或 `CompletionStage`。

```java
Bulkhead bulkhead = Bulkhead.of("name", config);

// 使用bulkhead装饰函数
CheckedFunction0<String> decoratedSupplier = Bulkhead
  .decorateCheckedSupplier(bulkhead, () -> "This can be any method which returns: 'Hello");

// 使用map链接其他的函数
Try<String> result = Try.of(decoratedSupplier)
  .map(value -> value + " world'");

// 如果所有的函数执行成功将返回Success<String>函子
assertThat(result.isSuccess()).isTrue();
assertThat(result.get()).isEqualTo("This can be any method which returns: 'Hello world'");
assertThat(bulkhead.getMetrics().getAvailableConcurrentCalls()).isEqualTo(1);
```

```java
ThreadPoolBulkheadConfig config = ThreadPoolBulkheadConfig.custom()
    .maxThreadPoolSize(10)
    .coreThreadPoolSize(2)
    .queueCapacity(20)
    .build();

ThreadPoolBulkhead bulkhead = ThreadPoolBulkhead.of("name", config);

CompletionStage<String> supplier = ThreadPoolBulkhead
    .executeSupplier(bulkhead, backendService::doSomething);
```



## 处理RegistryEvent事件

您可以在BulkheadRegistry上注册事件使用者，并在创建、替换或删除隔板时执行操作。

```java
BulkheadRegistry registry = BulkheadRegistry.ofDefaults();
registry.getEventPublisher()
  .onEntryAdded(entryAddedEvent -> {
    Bulkhead addedBulkhead = entryAddedEvent.getAddedEntry();
    LOG.info("Bulkhead {} added", addedBulkhead.getName());
  })
  .onEntryRemoved(entryRemovedEvent -> {
    Bulkhead removedBulkhead = entryRemovedEvent.getRemovedEntry();
    LOG.info("Bulkhead {} removed", removedBulkhead.getName());
  });
```

##  处理BulkheadEvents事件

```java
BulkheadRegistry registry = BulkheadRegistry.ofDefaults();
registry.getEventPublisher()
  .onEntryAdded(entryAddedEvent -> {
    Bulkhead addedBulkhead = entryAddedEvent.getAddedEntry();
    LOG.info("Bulkhead {} added", addedBulkhead.getName());
  })
  .onEntryRemoved(entryRemovedEvent -> {
    Bulkhead removedBulkhead = entryRemovedEvent.getRemovedEntry();
    LOG.info("Bulkhead {} removed", removedBulkhead.getName());
  });
```

## 处理BulkheadEvent事件

隔离发出隔离事件流。发出的事件有两种类型：允许执行、拒绝执行和完成执行。如果要使用这些事件，则必须注册事件使用者。

```java
bulkhead.getEventPublisher()
    .onCallPermitted(event -> logger.info(...))
    .onCallRejected(event -> logger.info(...))
    .onCallFinished(event -> logger.info(...));
```

