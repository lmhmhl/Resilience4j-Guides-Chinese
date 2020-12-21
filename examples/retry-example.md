# 重试例子



# 创建一个RetryRegistry

使用自定义的RetryConfig创建一个RetryRegistry。

```java
RetryConfig config = RetryConfig.custom()
    .maxAttempts(2)
    .waitDuration(Duration.ofMillis(100))
    .retryOnResult(response -> response.getStatus() == 500)
    .retryOnException(e -> e instanceof WebServiceException)ß
    .retryExceptions(IOException.class, TimeoutException.class)
    .ignoreExceptions(BunsinessException.class, OtherBunsinessException.class)
    .build();
    
// 使用自定义的全局配置创建RetryRegistry
RetryRegistry registry = RetryRegistry.of(config);
```

