# Spring Security JWT整合



![img](D:\MarkDown\Java\spring-security\img\liucheng.jpg)

登录认证流程

使用Filter进行拦截指定的连接`/login`,通过继承`AbstractAuthenticationProcessingFilter`实现 或者继承`UsernamePasswordAuthenticationFilter` 获取的`UsernamePasswordAuthenticationToken ` 也就是`AuthenticationToken`再封装为`Authentication`

进行验证

使用`DaoAuthenticationProvider `进行验证

`DaoAuthenticationProvider `调用`UserDetailsService`进行验证，调用`setPasswordEncoder`进行加密





认证结果处理

filter将token交给provider做校验，校验的结果无非两种，成功或者失败。对于这两种结果，我们只需要实现两个Handler接口，set到Filter里面，Filter在收到Provider的处理结果后会回调这两个Handler的方法。

先来看成功的情况，针对jwt认证的业务场景，登录成功需要返回给客户端一个token。所以成功的handler的实现类中需要包含这个逻辑。

再来看失败的情况，登录失败比较简单，只需要回复一个401的Response即可。

以上整个登录的流程的组件就完整了，我们只需要把它们组合到一起就可以了。这里继承一个`AbstractHttpConfigurer`，对Filter做配置。



