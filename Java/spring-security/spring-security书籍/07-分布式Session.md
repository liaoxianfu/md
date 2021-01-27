### 整合SpringSession实现分布式Session管理

Spring Security使用提供的会话并发是基于内存实现的，在集群部署的时候如果想要使用会话并发控制可以利用Spring sesion与Redis实现，整合起来非常简单，只要引入pom文件和配置redis即可



1、引入pom文件

```xml
     <dependency>
            <groupId>org.springframework.session</groupId>
            <artifactId>spring-session-data-redis</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```



2、配置application.yml

```xml
spring:
  redis:
    port: 6379
    host: 127.0.0.1
    database: 4
```

