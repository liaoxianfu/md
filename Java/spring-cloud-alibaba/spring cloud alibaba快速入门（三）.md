## spring cloud alibaba快速入门（三）

### spring-cloud-alibaba-dubbo 快速入门

#### 前置条件

安装好nacos

这里使用nacos作为注册中心，在spring cloud alibaba快速入门（一）中已经详细介绍了如何安装nacos,这里不做赘述。

#### 创建项目

在之前的项目中添加一个名为`spring-cloud-alibaba-dubbo-starter`的项目


在`spring-cloud-alibaba-dubbo-starter`下创建maven项目`spring-cloud-alibaba-dubbo-starter-provider`如下图所示:

![image-20200106193009212](img\image-20200106193009212.png)

同理创建`spring-cloud-alibaba-dubbo-starter-consumer`和`spring-cloud-alibaba-dubbo-starter-service`

![image-20200106193333297](img\image-20200106193333297.png)

效果如图所示。

#### 创建服务消费者和服务提供者的接口

在`spring-cloud-alibaba-dubbo-starter-service`项目下创建服务接口，这个接口是为`consumer`和`provider`提供。

创建接口如下

```java
package com.liao.spring.cloud.alibaba.dubbo.service;

public interface HelloService {
    String hello(String info);
}

```
#### 创建服务提供者

在`spring-cloud-alibaba-dubbo-starter-provider`下引入`spring-cloud-alibaba-dubbo-starter-service`

```xml
  <dependency>
            <groupId>com.liao</groupId>
            <artifactId>spring-cloud-alibaba-dubbo-starter-service</artifactId>
            <version>1.0.0-SNAPSHOT</version>
        </dependency>
```

在`spring-cloud-alibaba-dubbo-starter-provider`下引入dubbo的依赖支持

```xml
    <dependencies>
        <dependency>
            <groupId>com.liao</groupId>
            <artifactId>spring-cloud-alibaba-dubbo-starter-service</artifactId>
            <version>1.0.0-SNAPSHOT</version>
        </dependency>
        <!--spring cloud dubbo-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-dubbo</artifactId>
        </dependency>
        <!--在dubbo的服务提供者上使用，不用开启web端口-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <!--springboot的单元测试-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--服务发现-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--nacos-config-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <!--sentinel-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
    </dependencies>
```
创建服务提供者的入口类
```java
package com.liao.spring.cloud.alibaba.dubbo.service.provider;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DubboServiceProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(DubboServiceProviderApplication.class,args);
    }
}

```

创建服务提供者服务，这里以返回一个简单的字符串为例

```java
package com.liao.spring.cloud.alibaba.dubbo.service.provider.service;

import com.liao.spring.cloud.alibaba.dubbo.service.HelloService;
import org.apache.dubbo.config.annotation.Service;


@Service(version = "1.0.0")
public class HelloServiceImpl implements HelloService {
    @Override
    public String hello(String info) {
        return info;
    }
}
```

**注意**： 这里的`@Service`接口是来自`org.apache.dubbo.config.annotation.Service`这个包，而不是Spring下面的注解。实现的是`spring-cloud-alibaba-dubbo-starter-service`项目下的接口。

创建配置文件`application.yml`和`bootstrap.properties`因为这里引入了nacos的远程配置pom文件，所以必须添加`bootstrap.properties`配置文件进行远程配置中心地址的设置，否者会报错

bootstrap.properties

```properties
spring.application.name=spring-cloud-alibaba-dubbo-starter-provider
spring.cloud.nacos.config.server-addr=192.168.77.132:8848
management.endpoints.web.exposure.include=*
spring.cloud.nacos.config.file-extension=yaml
```

application.yml

```yml
dubbo:
  scan:
    # dubbo 服务扫描基准包
    base-packages: com.liao.spring.cloud.alibaba.dubbo.service.provider.service
  protocol:
    # dubbo 协议
    name: dubbo
    # dubbo 协议端口（ -1 表示自增端口，从 20880 开始）
    port: -1
  registry:
    # 挂载到 Spring Cloud 注册中心
    address: nacos://192.168.77.132:8848

spring:
  application:
    # Dubbo 应用名称
    name: spring-cloud-alibaba-dubbo-starter-provider
  main:
    # Spring Boot 2.1 需要设定
    allow-bean-definition-overriding: true
  cloud:
    nacos:
      # Nacos 服务发现与注册配置
      discovery:
        server-addr: 192.168.77.132:8848
```

启动服务；没有报错，证明启动成功，在nacos上可以发现已经注册成功

![image-20200106204116189](img\image-20200106204116189.png)



创建服务消费者

在`spring-cloud-alibaba-dubbo-starter-consumer`下引入依赖

```xml
    <dependencies>
        <dependency>
            <groupId>com.liao</groupId>
            <artifactId>spring-cloud-alibaba-dubbo-starter-service</artifactId>
            <version>1.0.0-SNAPSHOT</version>
        </dependency>

        <!--spring cloud dubbo-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-dubbo</artifactId>
        </dependency>
        <!--在dubbo的服务消费者上使用-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--springboot的单元测试-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--服务发现-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--nacos-config-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <!--sentinel-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
    </dependencies>
```

创建sptingboot入口类

```java
package com.liao.spring.cloud.alibaba.dubbo.service.consumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DubboServiceConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(DubboServiceConsumerApplication.class, args);
    }
}

```

创建配置文件`application.yml`和`bootstrap.properties`因为这里引入了nacos的远程配置pom文件，所以必须添加`bootstrap.properties`配置文件进行远程配置中心地址的设置，否者会报错

**application.yml**

```yml


dubbo:
  protocol:
    # dubbo 协议
    name: dubbo
    # dubbo 协议端口（ -1 表示自增端口，从 20880 开始）
    port: -1
  registry:
    # 挂载到 Spring Cloud 注册中心
    address: nacos://192.168.77.132:8848

spring:
  application:
    # Dubbo 应用名称
    name: spring-cloud-alibaba-dubbo-starter-consumer
  main:
    # Spring Boot 2.1 需要设定
    allow-bean-definition-overriding: true
  cloud:
    nacos:
      # Nacos 服务发现与注册配置
      discovery:
        server-addr: 192.168.77.132:8848
server:
  port: 14000
```

**bootstrap.properties**

```properties
spring.application.name=spring-cloud-alibaba-dubbo-starter-consumer
spring.cloud.nacos.config.server-addr=192.168.77.132:8848
management.endpoints.web.exposure.include=*
spring.cloud.nacos.config.file-extension=yaml
```

添加controller

```java
package com.liao.spring.cloud.alibaba.dubbo.service.consumer.controller;

import com.liao.spring.cloud.alibaba.dubbo.service.HelloService;
import org.apache.dubbo.config.annotation.Reference;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @Reference(version = "1.0.0")
    private HelloService helloService;
    @GetMapping("/{info}")
    public String hello(@PathVariable String info){
        return helloService.hello(info);
    }
}
```





启动服务，访问http://localhost:14000/123 浏览器出现123证明成功

访问nacos发现服务已经注册成功

![image-20200106204553445](img\image-20200106204553445.png)

**测试远程配置**

在nacos上添加配置如下图

![image-20200106204841236](img\image-20200106204841236.png)



点击发布，并删除`spring-cloud-alibaba-dubbo-starter-consumer`下的server配置

![image-20200106205119387](img\image-20200106205119387.png)



发现修改成功，证明远程配置加载成功。



















