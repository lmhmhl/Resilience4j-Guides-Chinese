# Gradle

这些项目需要使用JDK 8。该项目在JCenter和Maven Central上发布。
如果您使用Gradle，您可以包括resilient4j模块，如下所示。

```groovy
repositories {
    jCenter()
}

dependencies {
  compile "io.github.resilience4j:resilience4j-circuitbreaker:${resilience4jVersion}"
  compile "io.github.resilience4j:resilience4j-ratelimiter:${resilience4jVersion}"
  compile "io.github.resilience4j:resilience4j-retry:${resilience4jVersion}"
  compile "io.github.resilience4j:resilience4j-bulkhead:${resilience4jVersion}"
  compile "io.github.resilience4j:resilience4j-cache:${resilience4jVersion}"
  compile "io.github.resilience4j:resilience4j-timelimiter:${resilience4jVersion}"
}
```

## 快照

```groovy
repositories {
   maven { url 'http://oss.jfrog.org/artifactory/oss-snapshot-local/' }
}
```



