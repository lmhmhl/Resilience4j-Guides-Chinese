# 断路器

开始断路器相关内容的学习。

## 介绍

断路器通过有限状态机实现，有三个普通状态：关闭、开启、半开，还有两个特殊状态：禁用、强制开启。

![avatar](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/images/39cdd54-state_machine.jpg)

断路器使用滑动窗口来存储和统计调用的结果。你可以选择基于调用数量的滑动窗口或者基于时间的滑动窗口。基于访问数量的滑动窗口统计了最近N次调用的返回结果。居于时间的滑动窗口统计了最近N秒的调用返回结果。

## 基于访问数量的滑动窗口

基于访问数量的滑动窗口是通过一个有N个元素的循环数组实现。

如果滑动窗口的大小等于10，那么循环数组总是有10个统计值。滑动窗口增量更新总的统计值，随着新的调用结果被记录在环形数组中，总的统计值也随之进行更新。当环形数组满了，时间最久的元素将被驱逐，将从总的统计值中减去该元素的统计值，并该元素所在的桶进行重置。

检索快照（总的统计值）的时间复杂度为O(1)，因为快照已经预先统计好了，并且和滑动窗口大小无关。

关于此方法实现的空间需求（内存消耗）为O(n)。

## 基于时间的滑动窗口

基于时间的滑动窗口是通过有N个桶的环形数组实现。

如果滑动窗口的大小为10秒，这个环形数组总是有10个桶，每个桶统计了在这一秒发生的所有调用的结果（部分统计结果），数组中的第一个桶存储了当前这一秒内的所有调用的结果，其他的桶存储了之前每秒调用的结果。

滑动窗口不会单独存储所有的调用结果，而是对每个桶内的统计结果和总的统计值进行增量的更新，当新的调用结果被记录时，总的统计值会进行增量更新。

检索快照（总的统计值）的时间复杂度为O(1)，因为快照已经预先统计好了，并且和滑动窗口大小无关。

关于此方法实现的空间需求（内存消耗）约等于O(n)。由于每次调用结果（元组）不会被单独存储，只是对N个桶进行单独统计和一次总分的统计。

每个桶在进行部分统计时存在三个整型，为了计算，失败调用数，慢调用数，总调用数。还有一个long类型变量，存储所有调用的响应时间。

## 失败率和慢调用率阈值

当失败率大于或等于配置的阈值时，断路器的状态将从关闭变为开启，例如，当超过50%的调用失败时，断路器开启。

默认是把所有的异常看作是失败，你可以自己定义一个异常的列表，这些异常会被视为错误，其他的异常会被视为调用成功，除非是可以被忽略的异常。异常是可以被忽略的，这样它们既不算成功也不是算失败。

当慢调用的百分比大于等于配置的阈值时，断路器的状态将从关闭变为开启，例如，当超过50%的调用响应时间超过5秒时，断路器开启，这有助于在外部系统实际上没有响应之前减少它的负载。

只有在记录了最小调用次数的情况下，才能计算失败率和慢调用率。例如，如果所需调用的最小数目为10，则必须至少记录10个调用，然后才能计算失败率。如果只记录了9个调用，即使所有9个调用都失败，断路器也不会开启。

断路器在开启时，将会使用`CallNotPermittedException`来拒绝请求。在一段时间之后，断路器的状态将从开启变为半开，并且允许一定数量的调用通过，来判断后端的服务是否还是不可用或已变为可用，在所有允许通过断路器的调用完成之前，其余的调用还是被`CallNotPermittedException`拒绝。如果失败率或慢调用率大于等于配置的阈值，断路器状态将继续回到开启，如果失败率和慢调用率低于配置的阈值，断路器状态变为关闭。

断路器还支持两种特殊的状态，禁用（总是允许访问）和强制开启（总是拒绝访问）。在这两种状态下，不会产生断路器事件（除了状态转换），也不会记录任何指标。退出这些状态的唯一方法是触发状态转换或重置断路器。

断路器是线程安全的:

- 断路器的状态是原子引用。
- 断路器使用原子操作以无副作用的功能来更新状态。
- 从滑动窗口中记录调用结果和读取快照是同步的。

这意味着原子性得到了保证，在某一时刻，只允许一个线程对断路器的状态进行更改，和对滑动窗口进行操作。

但是断路器不会同步方法调用，这意味着方法调用不是核心的部分。否则，短路器将会带来大量的性能损失和瓶颈，耗时的方法调用会对整体性能/吞吐量带来巨大的负面影响。

如果有20个并发线程想要执行某个函数，并且断路器的状态为关闭，所有的线程都被允许进行方法调用，即使假设滑动窗口的大小是15，也不意味滑动窗口只允许15个调用并发的执行。如果你想要限制并发线程的数量，需要使用隔离机制，将隔离机制和断路器组合使用。

一个线程的例子:

- ![avatar](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/images/45dc011-Thread1.PNG)

> 

三个线程的例子:

- ![avatar](https://github.com/lmhmhl/Resilience4j-Guides-Chinese/blob/main/images/8d10418-Multiplethreads.PNG)

> 

## 创建一个CircuitBreakerRegistry

Resilience4j使用基于ConcurrentHashMap的`CircuitBreakerRegistry `来保证线程安全和原子性，你可以使用CircuitBreakerRegistry管理（创建和检索）断路器实例，可以使用全局默认的`CircuitBreakerConfig`配置为所有的断路器实例创建一个CircuitBreakerRegistry。

```java
CircuitBreakerRegistry circuitBreakerRegistry = 
  CircuitBreakerRegistry.ofDefaults();
```

##  创建和配置CircuitBreaker

你可以自定义`CircuitBreakerConfig`，为了创建自定义的CircuitBreakerConfig，你可以使用CircuitBreakerConfig建造器，你可以使用建造者模式来配置下面的属性。

| 配置属性                                          | 默认值                                                       | 描述                                                         |
| ------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| failureRateThreshold                              | 50                                                           | 以百分比配置失败率阈值。当失败率等于或大于阈值时，断路器状态并关闭变为开启，并进行服务降级。 |
| slowCallRateThreshold                             | 100                                                          | 以百分比的方式配置，断路器把调用时间大于`slowCallDurationThreshold`的调用视为满调用，当慢调用比例大于等于阈值时，断路器开启，并进行服务降级。 |
| slowCallDurationThreshold                         | 60000 [ms]                                                   | 配置调用时间的阈值，高于该阈值的呼叫视为慢调用，并增加慢调用比例。 |
| permittedNumberOfCallsInHalfOpenState             | 10                                                           | 断路器在半开状态下允许通过的调用次数。                       |
| maxWaitDurationInHalfOpenState                    | 0                                                            | 断路器在半开状态下的最长等待时间，超过该配置值的话，断路器会从半开状态恢复为开启状态。配置是0时表示断路器会一直处于半开状态，直到所有允许通过的访问结束。 |
| slidingWindowType                                 | COUNT_BASED                                                  | 配置滑动窗口的类型，当断路器关闭时，将调用的结果记录在滑动窗口中。滑动窗口的类型可以是count-based或time-based。如果滑动窗口类型是COUNT_BASED，将会统计记录最近`slidingWindowSize`次调用的结果。如果是TIME_BASED，将会统计记录最近`slidingWindowSize`秒的调用结果。 |
| slidingWindowSize                                 | 100                                                          | 配置滑动窗口的大小。                                         |
| minimumNumberOfCalls                              | 100                                                          | 断路器计算失败率或慢调用率之前所需的最小调用数（每个滑动窗口周期）。例如，如果minimumNumberOfCalls为10，则必须至少记录10个调用，然后才能计算失败率。如果只记录了9次调用，即使所有9次调用都失败，断路器也不会开启。 |
| waitDurationInOpenState                           | 60000 [ms]                                                   | 断路器从开启过渡到半开应等待的时间。                         |
| automaticTransition<br/>FromOpenToHalfOpenEnabled | false                                                        | 如果设置为true，则意味着断路器将自动从开启状态过渡到半开状态，并且不需要调用来触发转换。创建一个线程来监视断路器的所有实例，以便在WaitDurationInOpenstate之后将它们转换为半开状态。但是，如果设置为false，则只有在发出调用时才会转换到半开，即使在waitDurationInOpenState之后也是如此。这里的优点是没有线程监视所有断路器的状态。 |
| recordExceptions                                  | empty                                                        | 记录为失败并因此增加失败率的异常列表。<br/>除非通过ignoreExceptions显式忽略，否则与列表中某个匹配或继承的异常都将被视为失败。<br/>如果指定异常列表，则所有其他异常均视为成功，除非它们被ignoreExceptions显式忽略。 |
| ignoreExceptions                                  | empty                                                        | 被忽略且既不算失败也不算成功的异常列表。<br/>任何与列表之一匹配或继承的异常都不会被视为失败或成功，即使异常是recordExceptions的一部分。 |
| recordException                                   | throwable -> true·<br>By default all exceptions are recored as failures. | 一个自定义断言，用于评估异常是否应记录为失败。<br/>如果异常应计为失败，则断言必须返回true。如果出断言返回false，应算作成功，除非ignoreExceptions显式忽略异常。 |
| ignoreException                                   | throwable -> false<br>By default no exception is ignored.    | 自定义断言来判断一个异常是否应该被忽略，如果应忽略异常，则谓词必须返回true。<br/>如果异常应算作失败，则断言必须返回false。 |

```java
// 自定义断路器配置
CircuitBreakerConfig circuitBreakerConfig = CircuitBreakerConfig.custom()
  .failureRateThreshold(50)
  .slowCallRateThreshold(50)
  .waitDurationInOpenState(Duration.ofMillis(1000))
  .slowCallDurationThreshold(Duration.ofSeconds(2))
  .permittedNumberOfCallsInHalfOpenState(3)
  .minimumNumberOfCalls(10)
  .slidingWindowType(SlidingWindowType.TIME_BASED)
  .slidingWindowSize(5)
  .recordException(e -> INTERNAL_SERVER_ERROR
                 .equals(getResponse().getStatus()))
  .recordExceptions(IOException.class, TimeoutException.class)
  .ignoreExceptions(BusinessException.class, OtherBusinessException.class)
  .build();

// 使用默认配置创建circuitBreakerRegistry
CircuitBreakerRegistry circuitBreakerRegistry = 
  CircuitBreakerRegistry.ofDefaults();

// 使用circuitBreakerRegistry创建断路器，配置是默认配置 
CircuitBreaker circuitBreakerWithDefaultConfig = 
  circuitBreakerRegistry.circuitBreaker("name1");

// 使用circuitBreakerConfig创建注册器，进而创建断路器，配置是自定义配置
CircuitBreaker circuitBreakerWithCustomConfig = 
  CircuitBreakerRegistry.of(circuitBreakerConfig).circuitBreaker("diy");
```

你可以添加配置被多个断路器实例共享。

```java
CircuitBreakerConfig circuitBreakerConfig = CircuitBreakerConfig.custom()
  .failureRateThreshold(70)
  .build();

circuitBreakerRegistry.addConfiguration("someSharedConfig", config);

CircuitBreaker circuitBreaker = circuitBreakerRegistry
  .circuitBreaker("name", "someSharedConfig");
```



你可以重写配置。

```java
CircuitBreakerConfig defaultConfig = circuitBreakerRegistry
   .getDefaultConfig();

CircuitBreakerConfig overwrittenConfig = CircuitBreakerConfig
  .from(defaultConfig)
  .waitDurationInOpenState(Duration.ofSeconds(20))
  .build();
```



如果你不想使用CircuitBreakerRegistry来创建断路器的实例，可以直接创建断路器实例。

```java
// 创建断路器的自定义配置
CircuitBreakerConfig circuitBreakerConfig = CircuitBreakerConfig.custom()
  .recordExceptions(IOException.class, TimeoutException.class)
  .ignoreExceptions(BusinessException.class, OtherBusinessException.class)
  .build();

CircuitBreaker customCircuitBreaker = CircuitBreaker
  .of("testName", circuitBreakerConfig);
```

你还可以使用builder方法创建CircuitBreakerRegistry。

```java
io.vavr.collection.Map<String, String> circuitBreakerTags = io.vavr.collection.HashMap
            .of("key1", "value1", "key2", "value2");
       
CircuitBreakerRegistry circuitBreakerRegistry = CircuitBreakerRegistry.custom()
            .withCircuitBreakerConfig(CircuitBreakerConfig.ofDefaults()) .addRegistryEventConsumer(new RegistryEventConsumer() {
                @Override
                public void onEntryAddedEvent(EntryAddedEvent                                               entryAddedEvent) {// implementation
                }
                @Override
                public void onEntryRemovedEvent(EntryRemovedEvent                                       entryRemoveEvent) {// implementation
                }
                @Override
                public void onEntryReplacedEvent(EntryReplacedEvent                                     entryReplacedEvent) { // implementation}
            })
.withTags(circuitBreakerTags)
.build();

CircuitBreaker circuitBreaker = circuitBreakerRegistry.circuitBreaker("testName");
```

如果你想有自己的注册中心，你可以实现RegistryStore接口使用建造者模式创建对象。

```java
CircuitBreakerRegistry registry = CircuitBreakerRegistry.custom()
    .withRegistryStore(new YourRegistryStoreImplementation())
    .withCircuitBreakerConfig(CircuitBreakerConfig.ofDefaults())
    .build():
```

## 装饰和执行函数接口

您可以用断路器装饰任何Callable、Supplier、Runnable、Consumer、CheckedRunable、CheckedSupplier、CheckedConsumer或CompletionStage。
您可以使用尝试Vavr库提供的`Try.of()`或`Try.run()`，这允许使用map、flatMap、filter、recover或then链接更多函数。只有当断路器关闭或半开时，才调用链接的函数。
在下面的示例中，尝试如果函数调用成功，`Try.of()`返回Success<String>函子。如果函数抛出异常，则返回Failure<Throwable>函子，并且不继续调用map。

## 处理RegistryEvent事件

您可以在CircuitBreakerRegistry上注册事件处理者，并在创建、替换或删除断路器时执行操作。

```java
CircuitBreakerRegistry circuitBreakerRegistry = CircuitBreakerRegistry.ofDefaults();
circuitBreakerRegistry.getEventPublisher()
  .onEntryAdded(entryAddedEvent -> {
    CircuitBreaker addedCircuitBreaker = entryAddedEvent.getAddedEntry();
    LOG.info("CircuitBreaker {} added", addedCircuitBreaker.getName());
  })
  .onEntryRemoved(entryRemovedEvent -> {
    CircuitBreaker removedCircuitBreaker = entryRemovedEvent.getRemovedEntry();
    LOG.info("CircuitBreaker {} removed", removedCircuitBreaker.getName());
  });
```



## 处理CircuitBreakerEvent事件

CircuitBreakerEvent可以是状态转换、断路器重置、成功调用、记录的错误或被忽略的错误。所有事件都包含其他信息，如事件创建时间和调用的处理持续时间。如果要使用事件，则必须注册事件使用者

```java
circuitBreaker.getEventPublisher()
    .onSuccess(event -> logger.info(...))
    .onError(event -> logger.info(...))
    .onIgnoredError(event -> logger.info(...))
    .onReset(event -> logger.info(...))
    .onStateTransition(event -> logger.info(...));
// 如果你想对所有事件的进行监听并处理，可以这样做
circuitBreaker.getEventPublisher()
    .onEvent(event -> logger.info(...));
```

可以使用RxJava或RxJava2适配器将EventPublisher转换为反应流。

## 重写RegistryStore

可以通过自定义实现重写内存中的RegistryStore。例如，如果要使用一个缓存，该缓存会在一段时间后删除未使用的实例。

```java
CircuitBreakerRegistry circuitBreakerRegistry = CircuitBreakerRegistry.custom()
  .withRegistryStore(new CacheCircuitBreakerRegistryStore())
  .build();
```

