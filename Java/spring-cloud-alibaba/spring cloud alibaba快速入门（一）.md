## spring cloud alibaba快速入门（一）

### nacos快速入门

#### 1、nacos介绍

[nacos github地址](https://github.com/alibaba/Nacos)

一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

#### 2、快速上手

这里使用docker-compose快速安装nacos，[具体介绍](https://github.com/nacos-group/nacos-docker)见github。

方法：（这里默认已经安装好了docker-compose 如果没有安装，请参考我的博客）

第一步 先克隆项目

```sh
git clone --depth 1 https://github.com/nacos-group/nacos-docker.git
cd nacos-docker
```

官方提供很多的部署方式，这里使用`Standalone Derby`作为测试，其他的类似。执行以下命令

```sh
docker-compose -f example/standalone-derby.yaml up
```

启动后会出现如下的界面：

![image-20200104144222706](img\image-20200104144222706.png)

访问浏览器 输入`你的ip:8848/nacos`出现

![image-20200104144413143](img\image-20200104144413143.png)



用户名`nacos`

密码：`nacos`

#### 3、将nacos作为服务注册中心，实现服务提供者和服务消费者。

##### 3.1、创建项目

创建maven项目，以`spring-cloud-alibaba-learn`为例，作为父项目。

![image-20200104145039903](img\image-20200104145039903)

pom.xml内容如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.liao</groupId>
    <artifactId>spring-cloud-alibaba-learn</artifactId>
    <packaging>pom</packaging>
    <version>1.0.0-SNAPSHOT</version>
    <modules>
        <module>spring-cloud-alibaba-service-provider</module>
        <module>spring-cloud-alibaba-service-consumer</module>
    </modules>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.1.RELEASE</version>
    </parent>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2.1.0.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

</project>
```



##### 3.2、创建服务提供者

创建项目如下

![image-20200104145516898](img\image-20200104145516898.png)

修改pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring-cloud-alibaba-learn</artifactId>
        <groupId>com.liao</groupId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>spring-cloud-alibaba-service-consumer</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
        <!--服务发现-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--引入feign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
            <version>2.2.1.RELEASE</version>
        </dependency>
    </dependencies>
</project>
```



创建入口类`ServiceProviderApplication`

```java
package com.liao.spring.cloud.service.provider;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient // 允许注册到nacos
public class ServiceProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServiceProviderApplication.class,args);
    }
}
```

配置文件application.yml

```yml
server:
  port: 10000
spring:
  application:
    name: service-provider
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.200.72:8848 # nacos的地址

```



通过nacos注册中心查看 服务列表选项 可以看到注册的服务信息

![image-20200104150342489](img\image-20200104150342489.png)



创建一个简单的业务：

```java
package com.liao.spring.cloud.service.provider.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/test")
public class HelloController {
    @GetMapping("/hello/{info}")
    public String sayHello(@PathVariable String info) {
        return "hello:"+info;
    }
}

```



访问 http://127.0.0.1:10000/test/hello/world

出现

![image-20200104151440711](img\image-20200104151440711.png)

##### 3.3、创建服务消费者

这里为了方便直接使用feign作为服务消费者，raibbo类似，可以参考官方的github示例。

创建一个`spring-cloud-alibaba-service-consumer`的maven项目

![image-20200104151741702](img\image-20200104151741702.png)

修改pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring-cloud-alibaba-learn</artifactId>
        <groupId>com.liao</groupId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>spring-cloud-alibaba-service-consumer</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
        <!--引入服务发现-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--feign熔断-->
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
    </dependencies>
</project>
```



创建springboot的入口类`ServiceConsumerApplication`

```java
package com.liao.spring.cloud.service.consumer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableDiscoveryClient // 允许注册到nacos
@EnableFeignClients //允许feign客户端
public class ServiceConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServiceConsumerApplication.class, args);
    }
}

```



application.yml

```yml
server:
  port: 11000
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.200.72:8848
  application:
    name: service-consumer
feign:
  sentinel:
    enabled: true #允许开启fallback 默认为false

```



创建服务调用者`HelloServie` alibaba中不支持在接口前加@RequestMapping("/**")作为统一的url前缀。

```java
package com.liao.spring.cloud.service.consumer.service;

import com.liao.spring.cloud.service.consumer.service.fallback.HelloFallback;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient(name = "service-provider",fallback= HelloFallback.class)
public interface HelloService {
    @GetMapping("/test/hello/{info}")
    String sayHello(@PathVariable String info);
}

```



创建controller `HelloFeignController`

```java
package com.liao.spring.cloud.service.consumer.controller;

import com.liao.spring.cloud.service.consumer.service.HelloService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
@RequestMapping("/v1")
public class HelloFeignController {
    @Resource
    private HelloService helloService;

    @GetMapping("/hello/{info}")
    public String hello(@PathVariable String info) {
        return helloService.sayHello(info);
    }
}
```

访问http://localhost:11000/v1/hello/123

出现 hello:123证明创建成功。

查看nacos 

![image-20200104153704690](img\image-20200104153704690.png)

可以看到service-consumer在nacos也注册成功。

**创建熔断机制**

在service下创建fallback包然后创建`HelloServiceFallbackImpl`类 

```java
package com.liao.spring.cloud.service.consumer.service.fallback;

import com.liao.spring.cloud.service.consumer.service.HelloService;
import org.springframework.stereotype.Component;

@Component
public class HelloFallback implements HelloService {
    @Override
    public String sayHello(String info) {
        return info + "不可用";
    }
}
```

修改`HelloService`的`@FeignClient(name = "service-provider")`为`@FeignClient(name = "service-provider",fallback = HelloServiceFallbackImpl.class)`也就是增加调用失败的回调信息。

#### 4、实现远程配置服务

在原有的服务消费者和服务提供者进行修改，将配置迁移到nacos的配置中心进行配置。

对`spring-cloud-alibaba-service-provider`进行修改

![image-20200105084620118](img\image-20200105084620118.png)

##### 4.1、增加properties远程配置

增加bootstarp.properties

```properties
spring.application.name=service-provider # 应用名称，在nacos配置中需要使用
spring.cloud.nacos.config.server-addr=192.168.200.72:8848
management.endpoints.web.exposure.include=* # 这一条配置可以加在bootstarp.properties和application.properties中，但是不能加在application.yml中 在application.yml会找不到，报错
```

![image-20200105090003820](img\image-20200105090003820.png)

点击添加右边添加按钮，添加一条配置。

![image-20200105090410346](img\image-20200105090410346.png)

添加一条DataID为`service-provider.properties` 这里的命名规则为你在`bootstarp.properties`中的`spring.application.name`中的名称，再加上`.properties`配置规则。

配置内容： 根据自己的配置进行修改添加。这里测试添加了

```properties
user.name=liao12
user.age=23
```

点击`发布`按钮，发布这一条规则。

添加测试代码：

创建`HelloController`

```java
package com.liao.spring.cloud.service.provider.controller;
@RestController
@RequestMapping("/test")
public class HelloController {
    @Value("${user.name:liao}")
    String name;

    @Value("${user.age:12}")
    int age;

    @GetMapping("/hello/{info}")
    public String sayHello(@PathVariable String info) {
        return "hello:" + info + ": " + name + ": " + age;
    }
}

```



测试远程的信息能否被加载，如果在页面上能够正确的显示远程的配置信息能够被正确的加载就，就证明远程配置成功。

启动服务，访问http://localhost:10000/test/hello/123

![image-20200105091701098](img\image-20200105091701098.png)

这里能够正确的加载信息，加载成功。



##### 4.2、增加yaml远程配置

nacos同样支持yaml配置文件，只需要在bootstarp.properties中进行增加一条配置。

```properties
spring.cloud.nacos.config.file-extension=yaml
```

在nacos上添加一条远程配置文件

![image-20200105093055414](img\image-20200105093055414.png)



**注意，配置文件名是`应用名.yaml`不是我们写的yml**

点击发布

重启服务,访问http://localhost:10000/test/hello/123

出现



实验成功。

##### 4.3、支持多环境配置

spring-cloud-starter-alibaba-nacos-config 在加载配置的时候，不仅仅加载了以 dataid 为 `${spring.application.name}.${file-extension:properties}` 为前缀的基础配置，还加载了dataid为 `${spring.application.name}-${profile}.${file-extension:properties}` 的基础配置。

例如： 我在

在application.yml中添加

```yml
spring:
  profiles:
    active: develop
```

![image-20200105095707708](img\image-20200105095707708.png)

添加如图所示的一条配置，点击发布。修改`HelloController`为

```java
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

    @GetMapping("/hello/{info}")
    public String sayHello(@PathVariable String info) {
        return "hello:" + info + ": " + name + ": " + age + "当前的环境为：" + env;
    }
}
```

访问http://localhost:10000/test/hello/123

![image-20200105100905665](img\image-20200105100905665.png)

实验成功。

如果需要切换到生产环境，只需要更改 `${spring.profiles.active}` 参数配置即可。如下所示：

```properties
spring.profiles.active=product
```

##### 4.4、自定义namespace

官方给出的概念

> 用于进行租户粒度的配置隔离。不同的命名空间下，可以存在相同的 Group 或 Data ID 的配置。Namespace 的常用场景之一是不同环境的配置的区分隔离，例如开发测试环境和生产环境的资源（如配置、服务）隔离等。

简单来说，就类似于java中包的概念，不同的包下可以有相同名称的类。在开发中可以在不同的命名空间下创建相同的配置文件。例如，在开发环境下我需要有一个`service-provider.yaml`的配置文件，在生产环境下也需要一个`service-provider.yaml`的配置文件。如果这两个在同一个命名空间下是不可以共存的，所以需要创建一个开发的命名空间一个生产的命名空，在开发的时候在项目中使用开发的命名空间的远程配置，在生产上线的时候使用生产空间，两者互不干扰。

在没有明确指定 `${spring.cloud.nacos.config.namespace}` 配置的情况下， 默认使用的是 Nacos 上 Public 这个namespae。如果需要使用自定义的命名空间，可以通过以下配置来实现：

```properties
spring.cloud.nacos.config.namespace=你配置的命名空间名称
```

> 该配置必须放在 bootstrap.properties 文件中。此外 `spring.cloud.nacos.config.namespace` 的值是 namespace 对应的 id，id 值可以在 Nacos 的控制台获取。并且在添加配置时注意不要选择其他的 namespae，否则将会导致读取不到正确的配置。

![image-20200105102307989](img\image-20200105102307989.png)

在nacos上点击命名空间，点击新建命名空间创建一个开发的命名空间

![image-20200105102446679](img\image-20200105102446679.png)


创建成功后店家配置列表

![image-20200105104105281](img\image-20200105104105281.png)

我们将之前的public环境下的配置文件导出，然后上传到develop环境下，当然也可以自检重新创建配置。

导入配置文件之后，为了显示区别，修改`service-provider-develop.yaml`配置规则为

```yml
env: develop namespace
```

在bootstarp.properties中添加

`spring.cloud.nacos.config.namespace=82e48384-c0e5-4813-91b3-9c3a228ead7f`

因为我创建的develop环境的id为`82e48384-c0e5-4813-91b3-9c3a228ead7f`。

访问http://localhost:10000/test/hello/123出现

![image-20200105104015395](img\image-20200105104015395.png)