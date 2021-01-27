## spring cloud alibaba快速入门 （二）

### Sentinel负载均衡和流量监控

#### 1、sentinel控制台

从https://github.com/alibaba/Sentinel/releases下载最新版的控制台jar包。

执行

```sh
java -Dserver.port=8080 -Dcsp.sentinel.dashboard.server=localhost:8080 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard-版本号.jar
```

访问 http://127.0.0.1:8080/

用户名和密码都是`sentinel`

![image-20200105114755793](img\image-20200105114755793.png)

证明创建成功。

#### 2、在springcloud 中引入sentinel

##### 2.1、服务提供者引入

在`spring-cloud-alibaba-service-provider`中添加pom

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

在需要进行限流的地方使用`@SentinelResource` 注解用来标识资源是否被限流、降级。

在`spring-cloud-alibaba-service-provider`中添加配置

```java
package com.liao.spring.cloud.service.provider.controller;

import com.alibaba.csp.sentinel.annotation.SentinelResource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/test")
public class HelloController {
    @Value("${user.name:liao}")
    String name;

    @Value("${user.age:12}")
    int age;
    /*增加的修改，测试develop配置是否加载*/
    @Value("${env:none}")
    String env;

    /*Sentinel 引入流量监控*/
    @SentinelResource("hello")
    @GetMapping("/hello/{info}")
    public String sayHello(@PathVariable String info) {
        return "hello:" + info + ": " + name + ": " + age + "当前的环境为：" + env;
    }
}
```

在application.yml中增加配置

```yml
spring:
  cloud:
    sentinel:
      transport:
        port: 8719
        dashboard: localhost:8080 #sentinel的控制台地址
```

先访问一下http://localhost:10000/test/hello/123让sentinel中的先进行注册。在sentinel中

![image-20200105142921036](img\image-20200105142921036.png)



监控成功。

![image-20200105143055338](img\image-20200105143055338.png)

可以对对应的链路进行对应的操作。

##### 2.2、服务消费者引入（对feign的支持）

同理在`spring-cloud-alibaba-service-consumer`中添加依赖 因为之前在feign中进行熔断的时候需要sentinel所以无需再进行引入

```xml
  <!--feign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
            <version>2.2.1.RELEASE</version>
        </dependency>
        <!--熔断与流量监控-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
```

在配置文件application.yml中添加

```yml
spring:
  cloud:
    sentinel:
      transport:
        port: 8719
        dashboard: localhost:8080 #sentinel的控制台地址
```

重启服务，访问http://localhost:11000/test/hello/1

查看sentinel 如图：

![image-20200105144528412](img\image-20200105144528412.png)



在这里引入成功











