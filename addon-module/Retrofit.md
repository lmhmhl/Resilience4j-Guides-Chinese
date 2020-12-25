# Retrofit

Android和Java的类型安全HTTP客户端。https://square.github.io/refrofit/

## 断路器

断路器http客户端调用是基于`CircuitBreakerCallAdapter`的断路器实例。

```java
// 创建断路器
private final CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("testName");

// 创建retrofit实例使用CircuitBrealer调用适配器
Retrofit retrofit = new Retrofit.Builder()
                .addCallAdapterFactory(CircuitBreakerCallAdapter.of(circuitBreaker))
                .baseUrl("http://localhost:8080/")
                .build();

// 获取你的服务实例使用断路器来创建.
RetrofitService service = retrofit.create(RetrofitService.class);
```



## 等待时间

要触发超时断开电路，应在传递到`Retrofit.Builder`。

```java
//创建断路器
private final CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("testName");

final long TIMEOUT = 300; // ms
OkHttpClient client = new OkHttpClient.Builder()
        .connectTimeout(TIMEOUT, TimeUnit.MILLISECONDS)
        .readTimeout(TIMEOUT, TimeUnit.MILLISECONDS)
        .writeTimeout(TIMEOUT, TimeUnit.MILLISECONDS)
        .build();

Retrofit retrofit = new Retrofit.Builder()
        .addCallAdapterFactory(CircuitBreakerCallAdapter.of(circuitBreaker))
        .baseUrl("http://localhost:8080/")
        .client(client)
        .build();
```

## 错误响应

```java
Retrofit retrofit = new Retrofit.Builder()
        .addCallAdapterFactory(CircuitBreakerCallAdapter.of(circuitBreaker, (r) -> r.code() < 500));
        .baseUrl("http://localhost:8080/")
        .build();
```

## 限流器

http客户端调用的速率限制基于传递给`RateLimiterCallAdapter`的配置。

```java
RateLimiter rateLimiter = RateLimiter.ofDefaults("testName");

Retrofit retrofit = new Retrofit.Builder()
        .addCallAdapterFactory(RateLimiterCallAdapter.of(rateLimiter))
        .baseUrl("http://localhost:8080/")
        .build();
```

如果在速率限制器定义的时间段内超过了调用数，则会返回一个HTTP 429响应（请求太多）。