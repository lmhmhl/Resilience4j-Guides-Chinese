# Spring Cloud

## 设置

添加Resilience4j的依赖到Spring Cloud 2 start中。

Spring Cloud允许你使用Spring Cloud Config 作为注册中心，在运行时管理更新的属性信息。

此模块需要`org.springframework.boot:spring-boot-starter-actuator` 和`org.springframework.boot:spring-boot-starter-aop`模块。

Groovy

```groovy
repositories {
    jCenter()
}

dependencies {
    compile "io.github.resilience4j:resilience4j-spring-cloud2:${resilience4jVersion}"
    compile('org.springframework.boot:spring-boot-starter-actuator')
    compile('org.springframework.boot:spring-boot-starter-aop')
    compile('org.springframework.cloud:spring-cloud-starter-config')  
}
```

The configuration is similar to the Spring Boot 2 Starter. 

## 样例

SpringCloud的[demo](https://github.com/resilience4j/resilience4j-spring-cloud2-demo).



