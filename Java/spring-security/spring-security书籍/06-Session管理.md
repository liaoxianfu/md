### Session管理

Session是为无状态的http实现用户状态可维持的一种解方案。



#### 防止session固定攻击

防止session固定攻击的方式很简单，只需要用户在登录之后生成新的session即可，在继承`WebSecurityConfigAdapter` springsecurity 就默认开启了这个配置。

其中`sessionMagemanet`是一个会话管理配置器，其中防御会话固定攻击的策略有四种：

* none 不做任何变动，登录之后仍然沿用旧的session
* newSession 登录之后创建一个新的session
* migrateSession 创建新的Session并将旧的Session的数据复制过来
* changeSessionId不创建新的会话，而是使用Servlet容器提供会话固定保护

默认使用migrateSession策略，如有必要可以进行修改。在springsecurity java配置文件中进行修改

```java
...
   .and()
                // session管理
                .sessionManagement()
                .sessionFixation().none() // 修改session策略

....
```

在spring security中即使没有进行配置，spring security也会默认拦截Session固定攻击。spring security的http防火墙会拦截非法的url。当访问非法url时会重定向到错误页面。



#### Session过期



除了防御Session固定攻击之外，还可以通过Spring Security配置一些会话过期策略，例如会话过期后跳转到指定的url。

```java
.sessionManagement()
//                .sessionFixation().none()
                .invalidSessionUrl("/login.html")
```

或者通过实现InvalidSessionStrategy接口自定义过期策略

```java
@Component
public class MyInvalidSessionStrategy implements InvalidSessionStrategy {
    @Override
    public void onInvalidSessionDetected(HttpServletRequest request, HttpServletResponse response) throws IOException, ServletException {
        response.setContentType("application/json;charset=utf-8");
        response.getWriter().println("Session无效");
    }
}

```

在security配置代码中进行配置

```java
..... 
.and().sessionManagement().invalidSessionStrategy(new MyInvalidSessionStrategy())
     
     .....
```

默认情况下30分钟Session就会失效。可以在`application.yml`中进行配置

```yml
server:
  port: 8085
  servlet:
    session:
      timeout: 60 # 配置Session超时时间

```

会话的过期时间最少是60秒 如果小于这个时间 springboot会默认修正为60s

#### 会话并发控制

在javaconfig中配置允许登录的最大设备数

```java
     .and()
             .sessionManagement()
                // 设置允许登录的最大设备数目
                .maximumSessions(1);
```

如果没有额外的配置，默认新登录的用户会挤掉旧的登录用户。自定义Session失效的提示信息。

自定义已失效的Session的处理方式

```java
@Component
public class ExpireSessionStrategy implements SessionInformationExpiredStrategy {
    @Resource
    private ObjectMapper objectMapper;
    @Override
    public void onExpiredSessionDetected(SessionInformationExpiredEvent event) throws IOException, ServletException {
        HttpServletResponse response = event.getResponse();
        response.setContentType("application/json; charset=UTF-8");
        response.getWriter().write(objectMapper.writeValueAsString(R.error().data("您已经在其他地方登录，被迫下线")));
    }
}

```

在配置文件中进行配置

```java
     // 自定义过期处理策略            
.expiredSessionStrategy(expireSessionStrategy);

```





#### 集群会话解决方案

常见的解决方案有三种：

* session 保持
* Session复制
* Session共享

Session保持也被称之为粘滞会话，通常采用IP哈希负载均衡将来自相同客户端的请求转发到相同的服务器上。Session保持虽然避免了集群会话问题，但是会存在负载失衡问题。例如 在某个营业部使用的同一个IP出口，那么实际上该营业部访问的都会分配到同一个服务器。

Session复制是将集群之间的Session通过复制保证每一台服务器上的Session都是相同的，达到各个实例之间的会话状态都是一致的做法，这样做是非常不可取的，因为是无法达到实时同步而且存在大量Session的情况下会消耗带宽和资源。



Session共享是将Session单独抽取出来存放到独立的数据容器中，各个服务器进行共享。例如存放在redis。由于所有的服务器都去数据容器中取Session，因此不存在Session不一致的问题而且与服务器分离后不存在因为服务器停止造成Session丢失的问题。

当然Session共享并不是完全没有缺点的，独立的数据容器增加了网络交互，数据容器的网络、IO等都能成为性能的瓶颈，为了避免这些情况在内网中使用Redis是最佳的选项。









