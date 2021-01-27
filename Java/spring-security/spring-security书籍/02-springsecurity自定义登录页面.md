### 自定义登录页面



在上一节中我们使用了springsecurity中自带的登录页面 但是页面比较简陋。spring security也提供了自定义登录页面的选项。

在`spring-security-learn`项目下新建一个子项目`learn-02`



#### 0、准备工作

learn-02 pom.xml

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-config</artifactId>
        </dependency>
        <!--用于解析模板页面-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
         <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>

```



`com.liao.securtity.learn02.LoginFormApplication.java`启动文件

```java
@SpringBootApplication
public class LoginFormApplication {
    public static void main(String[] args) {
        SpringApplication.run(LoginFormApplication.class, args);
    }
}

```



页面准备

在`resources`目录下创建`pubilic`和`templates`两个目录 。

`pubilic` springboot不会拦截页面 可以直接访问 存放login.html

`templates` springboot存放模板文件的地方 会进行拦截 存放 demo1.html demo2.html index.html

login.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登录页面</title>
</head>
<body>
<form action="/login" method="post">
    <input name="username" type="text"><br/>
    <input name="password" type="password">
    <input type="submit" value="提交">
</form>

</body>
</html>
```



demo1.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
demo1
</body>
</html>
```



demo02.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
demo2
</body>
</html>
```



index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<a href="/demo1">demo1</a>
<br>
<a href="/demo2">demo2</a>
<br/>
<a href="/logout">退出登录</a>

</body>
</html>
```





#### 1、创建自定义的配置文件

创建`config`包在该包下创建自定义的javaConfig文件

`com.liao.securtity.learn02.config.SecurityConfig.java`继承 `WebSecurityConfigurerAdapter`

```java
@EnableWebSecurity // 自定义security配置
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
}
```

* 定义内存中的用户
* 定义不同用户的访问权限

为了简单期间 我们现在内存中定义两个用户 `admin`和`demo` 在spring security 设计师具有rbac权限的。现在先简单的理解为不同的用户能够访问的页面权限是不一样的。

并且在spring security中是不允许直接使用明文密码的，必须将明文密码进行加密后再使用。在spring security中提供了`PasswordEncoder`加密接口。可以通过实现该接口自定义加密方式。当然spring security 也帮我们实现了常用的加密方式，其中常用的是`BCryptPasswordEncoder`

```java
public interface PasswordEncoder {
    String encode(CharSequence var1);

    boolean matches(CharSequence var1, String var2);

    default boolean upgradeEncoding(String encodedPassword) {
        return false;
    }
}

```



定义加密方式为`BCryptPasswordEncoder`的组件	`com.liao.securtity.learn02.config.PasswordConfig`

```java
@Configuration
public class PasswordConfig {

    /**
     * 自定义用户密码的加密方式 常用的是BCryptPasswordEncoder
     * @return BCryptPasswordEncoder
     */
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}
```

实现在内存中的角色用户

```java
@EnableWebSecurity // 自定义security配置
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Resource
    private PasswordEncoder passwordEncoder;
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                // 定义用户名
                .withUser("admin")
                // 使用自定义的passwordEncoder加密密码
                .password(passwordEncoder.encode("123456"))
                .roles("admin")
                .and()
                .withUser("demo")
                .password(passwordEncoder.encode("1234"))
                .roles("demo");
    }
}

```

在SecurityConfig类实现自定义的security配置

```java
@EnableWebSecurity // 自定义security配置
public class SecurityConfig extends WebSecurityConfigurerAdapter {

@Override
    protected void configure(HttpSecurity http) throws Exception {
        http.
                // 取消csrf的限制 否者无法加载页面
                        csrf().disable()
                .formLogin().loginPage("/login.html")
                // 进行登录处理的URL 也就是form表单的action登录url 需要在antMatchers中允许访问 不进行拦截
                .loginProcessingUrl("/login")
                // 默认处理成功之后跳转的页面
                .defaultSuccessUrl("/index")
                .and()
                // 验证规则
                .authorizeRequests()
                // 允许所有用户包括未登录访问
                .antMatchers("/login.html", "/login").permitAll()
                // 允许已经访问的人进行登录
                .antMatchers("/index").authenticated()
                // 允许admin角色用户登录
                .antMatchers("/demo1").hasAnyRole("admin")
                // 角色本身也是一种特殊的权限 当用户属于一种角色后就会默认存在一个ROLE_角色名的权限
                // 也就是允许拥有ROLE_admin和ROLE_user权限的用户登录 即角色为admin或者user的用户登录
                .antMatchers("/demo2").hasAnyAuthority("ROLE_admin", "ROLE_demo");

    }
 .....// 省略内存用户的代码   
}
```

启动项目 

我们可以发现 admin可以访问demo1和demo2页面 但是demo用户只能访问demo2页面 不能访问demo1页面。



#### 2、 修改项目为Restful风格

在`learn-02`的基础上进行修改

创建统一返回模板





创建自定义的登录成功失败的Handler



登录成功

```java
@Configuration
public class LoginSuccessHandler implements AuthenticationSuccessHandler {
    @Resource
    private ObjectMapper objectMapper;

    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication)
            throws IOException, ServletException {

        R success = R.success().data("登录成功");
        response.setContentType("application/json;charset=utf-8");
        response.getWriter().write(objectMapper.writeValueAsString(success));
    }
}

```

登录失败

```java

@Configuration
public class LoginFailHandler implements AuthenticationFailureHandler {
    @Resource
    private ObjectMapper objectMapper;

    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException e) throws IOException, ServletException {
        R data = R.error().data("登录失败");
        response.setContentType("application/json; charset=UTF-8");
        response.getWriter().write(objectMapper.writeValueAsString(data));
    }
}
```



在SecurityConfig类实现自定义的security配置

```java
@EnableWebSecurity // 自定义security配置
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Resource
    private PasswordEncoder passwordEncoder;

    @Resource
    private LoginSuccessHandler successHandler;

    @Resource
    private LoginFailHandler failHandler;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.
                // 取消csrf的限制 否者无法加载页面
                        csrf().disable()
                .formLogin()
                .loginPage("/login.html")
                // 进行登录处理的URL 也就是form表单的action登录url 需要在antMatchers中允许访问 不进行拦截
                .loginProcessingUrl("/login")
                .successHandler(successHandler) // 自定义登录成功的处理方式
                .failureHandler(failHandler) // 自定义失败的处理方式
                .and()
                // 验证规则
                .authorizeRequests()
                // 允许所有用户包括未登录访问
                .antMatchers("/login.html", "/login").permitAll()
                // 允许已经访问的人进行登录
                .antMatchers("/index").authenticated()
                // 允许admin角色用户登录
                .antMatchers("/demo1").hasAnyRole("admin")
                // 角色本身也是一种特殊的权限 当用户属于一种角色后就会默认存在一个ROLE_角色名的权限
                // 也就是允许拥有ROLE_admin和ROLE_user权限的用户登录 即角色为admin或者user的用户登录
                .antMatchers("/demo2").hasAnyAuthority("ROLE_admin", "ROLE_demo");

    }

.... // 省略代码
}

```



修改login.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登录页面</title>
</head>
<body>
<h1>登录页面</h1>

<div>
    用户名<input type="text" name="username" id="username"><br>
    密码 <input type="password" name="password" id="password"><br>
    <button id="btn">登录</button>
</div>

<script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.5.1/jquery.js"></script>
<script>
    $("#btn").click(() => {
        let username = $("#username").val();
        let password = $("#password").val();
        let form = {
            "username": username,
            "password": password,
        };
        $.ajax({
            url: "/login",
            method: "post",
            data: form,
            success: (res) => {
                if (res.code === 200) {
                    alert(JSON.stringify(res));
                    location.href = "/index"
                } else {
                    alert(JSON.stringify(res));
                    alert("用户名或者密码错误，请重新填写");
                    $("#username").val("");
                    $("#password").val("")
                }
            }
        })
    })
</script>


</body>
</html>
```









