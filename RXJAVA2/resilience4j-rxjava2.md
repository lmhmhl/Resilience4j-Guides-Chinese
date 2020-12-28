# resilience4j-rxjava2

Resilience4j 提供了一个模块，使用自定义的RxJava2操作符。这些操作符能够判断下流的订阅者或观察者是否允许对上游的发布者进行订阅，反应式类型支持`Observable`, `Flowable`, `Single`, `Maybe` 和`Completable`。

模块期望`io.reactivex.rxjava2`：rxjava已在运行时提供。RxJava2不是可传递的依赖关系。

```groovy
repositories {
    jCenter()
}

dependencies {
  compile "io.github.resilience4j:resilience4j-rxjava2:${resilience4jVersion}"
}
```

