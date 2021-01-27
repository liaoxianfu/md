### spring boot与spring security整合



#### 创建项目

为了后续项目的统一管理，在这里创建同意管理的父工程 springsecurity-learn 作为统一的pom依赖管理。

父pom文件内容

```xml
    <dependencyManagement>
        <!--统一管理springboot项目工程-->
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.1.17.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```





创建子项目``learn-01`

项目pom文件

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-web</artifactId>
        </dependency>
         <!--必须添加 会自动配置-->
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-config</artifactId>
        </dependency>
       
    </dependencies>

```



创建springboot application

创建`com.liao.securtity.learn01.HelloSecurityApplication.java`文件作为springboot的启动类

```java
@SpringBootApplication
public class HelloSecurityApplication {
    public static void main(String[] args) {
        SpringApplication.run(HelloSecurityApplication.class, args);
    }
}
```



创建测试路由`com.liao.securtity.learn01.controller.HelloController.java`文件

```java

@RestController
public class HelloController {

    @GetMapping("hello")
    public String hello() {
        return "spring security";
    }
}
```



创建配置文件

resources/application.yml

```yml
server:
  port: 8001
```

打开浏览器访问可以看到

![image-20200924115313296](D:\MarkDown\Java\spring-security\spring-security书籍\img\image-20200924115313296.png)



默认的用户名是`user`

密码为在控制台打印的一串uuid

![image-20200924153118637](D:\MarkDown\Java\spring-security\spring-security书籍\img\image-20200924153118637.png)





自定义登录的用户名和密码

在配置文件中`applicantion.yml`自定义

```yml
spring:
  security:
    user:
      name: root
      password: 123456
```















