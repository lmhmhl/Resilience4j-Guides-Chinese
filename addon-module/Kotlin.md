# Kotlin

开始resilience4j-kotlin的学习

## 介绍

Kotlin协程的集成。此模块添加了用于执行和装饰`suspend`函数的扩展，以及用于协程反应原语流的自定义运算符。

为限流器、重试、断路器、限时器和基于信号量的隔离提供接受Kotlin 的suspend函数的扩展函数。目前没有为ThreadPoolBulkhead或缓存提供扩展。

对于流，提供了用于限流器、重试、断路器、限时器和基于信号量的隔离的操作符。目前没有为ThreadPoolBulkhead或缓存提供扩展。

## 设置

将Resilience4j的Kotlin模块与所需的核心模块一起添加到编译依赖项中：

```groovy
repositories {
    jCenter()
} 

dependencies {
    compile "io.github.resilience4j:resilience4j-kotlin:${resilience4jVersion}"
  // 还要依赖于所需的核心模块—例如，重试：:
    compile "io.github.resilience4j:resilience4j-retry:${resilience4jVersion}"
}
```

## 用法

为断路器、限流器、重试和限时器声明了两个扩展函数：一个用于执行挂起函数，另一个用于修饰挂起函数。

## 挂起函数的用法

为断路器、流限制器、重试和限时器声明了两个扩展函数：一个用于执行挂起函数，另一个用于修饰挂起函数。

```kotlin
val circuitBreaker = CircuitBreaker.ofDefaults()
val result = circuitBreaker.executeSuspendFunction {
   // call suspending functions here
}

val function = circuitBreaker.decorateSuspendFunction {
   // call suspending functions here
}
val result = function()
```

suspend函数在使用常规方法会阻塞的地方挂起。例如，在执行给定函数之前，需要延迟以适应速率限制的对速率限制器的调用将挂起。
不会对给定挂起函数的协程上下文进行任何更改。

## 流的用法

对于每个`CircuitBreaker`、`RateLimiter`、`Retry`和`TimeLimiter`，都定义了`Flow<T>`的扩展。运算符链的顺序很重要，并且与resilience原语的求值顺序直接相关。考虑以下示例flowOf（1）.retry（…）.timeLimiter（…），所有重试尝试必须在提供的timeLimiter的持续时间内完成。

```kotlin
val retry = Retry.ofDefaults()
val rateLimiter = RateLimiter.ofDefaults()
val timeLimiter = TimeLimiter.ofDefaults()
val circuitBreaker = CircuitBreaker.ofDefaults()
  
flowOf(1, 2, 3)
  .retry(retry)
  .rateLimiter(rateLimiter)
  .timeLimiter(timeLimiter)
  .circuitBreaker(circuitBreaker)
  .collect { println(it) }
```

## 隔离

使用隔离装饰suspend函数或流需要特别注意`maxWaitTime`的配置值。如果`maxWaitTime`非零，则调用将阻塞，直到达到最大等待时间或获得权限。此阻塞仅限于调度程序`Dispatchers.IO`. 尽管这有助于减轻使用阻塞API所带来的一些性能影响，但不建议将此扩展函数用于具有非零最大等待时间的隔离。

如果协同路由作用域被取消，无论是正常还是异常，所获得的权限都将被释放。之后不会记录成功或失败事件。

没有为基于线程池的隔离提供扩展功能。

## 断路器

用断路器装饰挂起函数或流不会添加任何额外的挂起点。在修饰流的情况下，如果断路器状态为打开，则尝试收集将引发`CallNotPermittedException`。

如果协同路由作用域被取消，无论是正常还是异常，所获得的权限都将被释放。之后不会记录成功或失败事件。

## 限流器

使用速率限制器修饰的Suspend函数使用`delay()`在执行修饰函数（如果达到速率限制）之前暂停。

如果达到速率限制，则用RateLimiter修饰的流将在尝试收集时调用`delay()`。

## 重试

用Retry修饰的`Suspend`函数和流使用`delay()`在两次重试之间挂起。

## 限时器

`TimeLimiter`扩展函数只需使用kotlin协同例程中的`withTimeout()`，使用接收方配置中的超时。具体来说，这意味着：

超时将引发`TimeoutCancellationException`，而不是像用于非挂起函数的方法那样引发`TimeoutException`。
当超时发生时，协程被取消，而不是像非挂起函数的方法那样中断线程。
超时后，给定的块只能在可取消的挂起函数调用时停止。
`cancelRunningFuture`配置设置被忽略-超时时，即使`cancelRunningFuture`设置为`false`，挂起函数也始终被取消。
用TimeLimiter修饰的流将确保一旦收集开始，整个流在配置的限制内被消耗。

