# bulkhead例子

## 创建BulkheadRegistry

使用自定义的配置创建bulkhead。

```java
// 为bulkhead创建自定义配置
BulkheadConfig config = BulkheadConfig.custom()
        .maxConcurrentCalls(10)
        .maxWaitDuration(Duration.ofMillis(1))
        .build();

// 使用自定义的配置创建BulkheadRegistry
BulkheadRegistry bulkheadRegistry =
        BulkheadRegistry.of(config);
```



## 创建Bulkhead

从BulkheadRegistry中，使用全局默认的配置创建Bulkhead。

```java
Bulkhead bulkhead = bulkheadRegistry
  .bulkhead("name");
```



## 装饰函数式接口

使用Bulkhead装饰`BackendService.doSomething()`,如果出现异常，从异常中恢复。

```java
Supplier<String> decoratedSupplier = Bulkhead
    .decorateSupplier(retry, backendService::doSomething);

String result = Try.ofSupplier(decoratedSupplier)
    .recover(throwable -> "Hello from Recovery").get();
```

