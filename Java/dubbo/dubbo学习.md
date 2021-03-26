# dubbo学习

1、整合springboot



2、dubbo配置

- JVM System Properties，-D参数

- Externalized Configuration，外部化配置（dubbo2.7新增）

- ServiceConfig、ReferenceConfig等编程接口采集的配置

- 本地配置文件dubbo.properties

  

![覆盖关系](D:\MarkDown\Java\dubbo\img\configuration.jpg)

优先级依次降低、优先级高的会覆盖优先级低的配置。

## 启动时检查：

Dubbo 缺省会在启动时检查依赖的服务是否可用，不可用时会抛出异常，阻止 Spring 初始化完成，以便上线时，能及早发现问题，默认 `check="true"`。

可以通过 `check="false"` 关闭检查，比如，测试时，有些服务不关心，或者出现了循环依赖，必须有一方先启动。

另外，如果你的 Spring 容器是懒加载的，或者通过 API 编程延迟引用服务，请关闭 check，否则服务临时不可用时，会抛出异常，拿到 null 引用，如果 `check="false"`，总是会返回引用，当服务恢复时，能自动连上。

再springboot中进行关闭

#### 全局关闭

官网放在了dubbo.properties idea不会进行提示 可以放在application.properties有提示

```properties
# 默认不检查
dubbo.consumer.check=false
```

官网给出的

```properties
dubbo.reference.com.foo.BarService.check=false
dubbo.reference.check=fale
```

`dubbo.reference.check=false`，强制改变所有 reference 的 check 值，就算配置中有声明，也会被覆盖。（==实测不起作用==）

`dubbo.consumer.check=false`，是设置 check 的缺省值，如果配置中有显式的声明，如：<dubbo:reference check="true"/>，不会受影响。

#### 服务级关闭

使用注解进行关闭

```java
 @DubboReference(check = false)
 private UserService userService;
```

## 集群容错

在集群调用失败时，Dubbo 提供了多种容错方案，缺省为 failover 重试。

![cluster](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/cluster.jpg)

各节点关系：

- 这里的 `Invoker` 是 `Provider` 的一个可调用 `Service` 的抽象，`Invoker` 封装了 `Provider` 地址及 `Service` 接口信息
- `Directory` 代表多个 `Invoker`，可以把它看成 `List` ，但与 `List` 不同的是，它的值可能是动态变化的，比如注册中心推送变更
- `Cluster` 将 `Directory` 中的多个 `Invoker` 伪装成一个 `Invoker`，对上层透明，伪装过程包含了容错逻辑，调用失败后，重试另一个
- `Router` 负责从多个 `Invoker` 中按路由规则选出子集，比如读写分离，应用隔离等
- `LoadBalance` 负责从多个 `Invoker` 中选出具体的一个用于本次调用，选的过程包含了负载均衡算法，调用失败后，需要重选。

### Failover Cluster

失败自动切换，当出现失败，重试其它服务器 [[1\]](http://dubbo.apache.org/zh-cn/docs/user/demos/fault-tolerent-strategy.html#fn1)。通常用于读操作，但重试会带来更长延迟。可通过 `retries="2"` 来设置重试次数(不含第一次)。

全局配置

```properties
dubbo.consumer.registries=2
```

服务配置

```java
@DubboReference(retries = 3)
private UserService userService;
```

### Failfast Cluster

快速失败，只发起一次调用，失败立即报错。通常用于==非幂等性==的写操作，比如==新增记录==。

### Failsafe Cluster

失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作。

### Failback Cluster

失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作。

### Forking Cluster

并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。可通过 `forks="2"` 来设置最大并行数。

### Broadcast Cluster

广播调用所有提供者，逐个调用，任意一台报错则报错 [[2\]](http://dubbo.apache.org/zh-cn/docs/user/demos/fault-tolerent-strategy.html#fn2)。通常用于通知所有提供者更新缓存或日志等本地资源信息。



### 设置容错方案

全局配置

```properties
# 指定上面的容错方案即可 默认是
dubbo.consumer.cluster=failfast 
```

服务级配置

```java
@DubboReference(cluster = "failfast")
private UserService userService;
```





### 负载均衡

在集群负载均衡时，Dubbo 提供了多种均衡策略，缺省为 `random` 随机调用。



## 负载均衡策略

### Random LoadBalance

- **随机**，按权重设置随机概率。
- 在一个截面上碰撞的概率高，但调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重。

### RoundRobin LoadBalance

- **轮询**，按公约后的权重设置轮询比率。
- 存在慢的提供者累积请求的问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上。

### LeastActive LoadBalance

- **最少活跃调用数**，相同活跃数的随机，活跃数指调用前后计数差。
- 使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差会越大。

### ConsistentHash LoadBalance

- **一致性 Hash**，相同参数的请求总是发到同一提供者。
- 当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动。
- 算法参见：http://en.wikipedia.org/wiki/Consistent_hashing
- 缺省只对第一个参数 Hash，如果要修改，请配置 ``
- 缺省用 160 份虚拟节点，如果要修改，请配置 ``



## 线程模型

如果事件处理的逻辑能迅速完成，并且不会发起新的 IO 请求，比如只是在内存中记个标识，则直接在 IO 线程上处理更快，因为减少了线程池调度。

但如果事件处理逻辑较慢，或者需要发起新的 IO 请求，比如需要查询数据库，则必须派发到线程池，否则 IO 线程阻塞，将导致不能接收其它请求。

如果用 IO 线程处理事件，又在事件处理过程中发起新的 IO 请求，比如在连接事件中发起登录请求，会报“可能引发死锁”异常，但不会真死锁。





## 直连提供者

在开发及测试环境下，经常需要绕过注册中心，只测试指定服务提供者，这时候可能需要点对点直连，点对点直连方式，将以服务接口为单位，忽略注册中心的提供者列表，A 接口配置点对点，不影响 B 接口从注册中心获取列表。

![/user-guide/images/dubbo-directly.jpg](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/dubbo-directly.jpg)

`2.0` 以上版本自动加载 ${user.home}/dubbo-resolve.properties文件，不需要配置 。==**注意** 为了避免复杂化线上环境，不要在线上使用这个功能，只应在测试阶段使用。==

也可以使用注解，但是==强烈不推荐==

```java
 @DubboReference(url = "dubbo://127.0.0.1:20880")
    private UserService userService;
```

## 只订阅

为方便开发测试，经常会在线下共用一个所有服务可用的注册中心，这时，如果一个正在开发中的服务提供者注册，可能会影响消费者不能正常运行。

可以让服务提供者开发方，只订阅服务(开发的服务可能依赖其它服务)，而不注册正在开发的服务，通过直连测试正在开发的服务。

![/user-guide/images/subscribe-only.jpg](D:\MarkDown\Java\dubbo\img\subscribe-only.jpg)

```properties
dubbo.registry.address=zookeeper://192.168.77.200:2181
dubbo.registry.register=false
```

## 只注册



如果有两个镜像环境，两个注册中心，有一个服务只在其中一个注册中心有部署，另一个注册中心还没来得及部署，而两个注册中心的其它应用都需要依赖此服务。这个时候，可以让服务提供者方只注册服务到另一注册中心，而不从另一注册中心订阅服务。

禁用订阅配置

```xml
<dubbo:registry id="hzRegistry" address="10.20.153.10:9090" />
<dubbo:registry id="qdRegistry" address="10.20.141.150:9090" subscribe="false" />
```

或者

```xml
<dubbo:registry id="hzRegistry" address="10.20.153.10:9090" />
<dubbo:registry id="qdRegistry" address="10.20.141.150:9090?subscribe=false" />
```

## 多协议

不同服务在性能上适用不同协议进行传输，比如大数据用短连接协议，小数据大并发用长连接协议

使用java api配置

```java
    @Bean
    public ProtocolConfig setProtocolConfig() {
        ProtocolConfig config = new ProtocolConfig();
        config.setName("dubbo");
        config.setPort(20880);
        return config;
    }

  @Bean
    public ProtocolConfig setProtocolConfig2() {
        ProtocolConfig config = new ProtocolConfig();
        config.setName("rmi");
        config.setPort(8880);
        return config;
    }
```

在service impl指定协议

```java
@DubboService(protocol = {"rmi"}) //可以指定多个
public class UserServiceImpl implements UserService {
    @Override
    public User findUserById(String userId) {
        System.out.println("333333333");


        return new User("liao", userId);
    }
}

```



## 多注册中心

http://dubbo.apache.org/zh-cn/docs/user/demos/multi-registry.html



## 服务分组

当一个接口有多种实现时，可以用 group 区分。

在==服务提供者上==指定需要不同的实现上指定版本

```java
@DubboService(group = "gp2") // 指定版本
public class UserServiceImpl implements UserService {
    @Override
    public User findUserById(String userId) {
        System.out.println("gp2222222222222");


        return new User("liao", userId);
    }
}

```

在消费者上指定需要消费的接口的group

```java
    @DubboReference(group = "gp2")
    private UserService userService;
```



## 多版本

当一个接口实现，出现不兼容升级时，可以用版本号过渡，版本号不同的服务相互间不引用。

可以按照以下的步骤进行版本迁移：

1. 在低压力时间段，先升级一半提供者为新版本
2. 再将所有消费者升级为新版本
3. 然后将剩下的一半提供者升级为新版本

服务提供者

```java
@DubboService(version = "1.0.0")
public class UserServiceImpl implements UserService {
    @Override
    public User findUserById(String userId) {
        System.out.println("version 1.0.0");
        return new User("liao", userId);
    }
}
```







```java
@DubboService(version = "2.0.0")
public class UserServiceImpl implements UserService {
    @Override
    public User findUserById(String userId) {
        System.out.println("version 2.0.0");
        return new User("liao", userId);
    }
}

    
    
```



服务消费者

```java
 @DubboReference(version = "1.0.0") // 指定版本 version = "*" 不区分版本
    private UserService userService;
```







## 令牌验证

通过令牌验证在注册中心控制权限，以决定要不要下发令牌给消费者，可以防止消费者绕过注册中心访问提供者，另外通过注册中心可灵活改变授权方式，而不需修改或升级提供者

![/user-guide/images/dubbo-token.jpg](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/dubbo-token.jpg)

