# resilience4j-reactor

Resilience4j提供了一个带有自定义Spring Reactor 操作符的模块。操作符确保下游订阅者可以获得订阅上游发布服务器的权限。支持创建类型`Mono`和`Flux`。

`io.projectreactor:reactor-core`在运行时提供。springreactor不是一个可传递的依赖项。

```java
repositories {
    jCenter()
}

dependencies {
  compile "io.github.resilience4j:resilience4j-reactor:${resilience4jVersion}"
}
```

