### 自动登录

自动登录的原理是将用户的登录信息保存在用户浏览器的cookie中，当用户下次访问时实现自动校验并建立登录状态的一种机制。

Spring Security 提供了两种令牌机制

* 用散列算法加密用户必要的登录信息并生成令牌
* 数据库等持久化存储机制持久化令牌



#### 散列算法

将用户名（username）、过期时间（expirationTime）、密码（password）和散列盐值（key）进行md5运算得到令牌。再下次登录时会使用base64简单解码得到用户名、过期时间和加密散列值，然后通过用户名得到密码，接着重新计算与旧的散列值进行对比确认是否有效。

```java
hashInfo = md5Hex(username+":"+expirationTime+":"+password+":"+key);
remeberCookie = base64(username+":"+expirationTime+":"+hashInfo)
```







在Spring Security中实现该功能非常简单，只需要在javaconfig进行配置即可。这里在learn-05上进行修改。

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                // 在验证用户名和密码之前验证验证码
                .addFilterBefore(captchaCodeFilter, UsernamePasswordAuthenticationFilter.class).
               ..... // 省略其他代码
                .and()
                .rememberMe()
                // 定义过期时间 一周
                .tokenValiditySeconds(7 * 24 * 60 * 60)
                // 定义记住我 在前端的name
                .rememberMeParameter("remember-me")
                // 在cookie中存放显示的名称
                .rememberMeCookieName("remember-cookies");

    }
```

在前端登录页面添加记住我的功能。



```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<div>
    用户名<input name="username" id="username"><br>
    密码 <input type="password" id="password"><br>
    验证码<input type="text" id="kaptcha">
    <img src="/kaptcha" alt="验证码" id="img_cha"><br>
    <br>
    <label for="rememberMe">一周之内免登陆</label><input type="checkbox" id="rememberMe"/>
    <button id="btn">登录</button>
    <br>
</div>
<script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.5.1/jquery.js"></script>
<script>

    $("#img_cha").click(() => {
        console.log("hehhe");
        $('#img_cha').attr("src", "/kaptcha");
    });


    $("#btn").click(() => {
        let username = $("#username").val();
        let password = $("#password").val();
        let checked = $("#rememberMe").is(":checked");
        let captcha = $("#kaptcha").val();
        let form = {
            "username": username,
            "password": password,
            "captcha": captcha,
            "remember-me": checked,
        };
        $.ajax({
            url: "/login",
            method: "post",
            data: form,
            success: (res) => {
                if (res.code === 200) {
                    location.href = "/index"
                } else {
                    console.log(res);
                    alert(res.data);
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





#### 存在的问题

在每次服务重启之后，key的值都会进行更新导致重启之前的cookie失效而且如果是多实例部署的情况下，由于实例之间的key值不同，所以当用户访问另一个实例的时候自动登录状态就会失效。合理的用法是指定key，这里测试即使是指定了key在重启之后仍然需要进行登录。

```java
   .rememberMeCookieName("remember-cookies")
                .userDetailsService(userDetailsService()).key("fba675ec-9a6b-489a-bb8a-607f1558c18f");
```



#### 持久化令牌

持久化在交互逻辑上和散列加密是一致的，都是用户在勾选了remember-me后将生成的令牌发送给浏览器，用户在下次访问系统时读取该令牌进行认证。不同的是持久化令牌采取了更加严格的验证。

在持久化令牌方案中最核心的是`series`和`token`两个值，都是采用MD5散列过的随机字符串，不同的是`series`在用户使用密码进行重新登录后进行更新，而token则是每次会话都会进行更新。自动登录并不会将series进行更改，当令牌在未使用时就被盗用时，系统会在非法验证用户通过后刷新token。此时的合法用户的token也会失效，系统因此可以对合法用户进行提醒 用户账号可能被盗用。

创建持久化令牌的数据库表

```sql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for persistent_logins
-- ----------------------------
DROP TABLE IF EXISTS `persistent_logins`;
CREATE TABLE `persistent_logins`  (
  `username` varchar(60) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `series` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `token` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `last_used` timestamp(0) NULL DEFAULT NULL,
  PRIMARY KEY (`series`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

```



引入新的依赖

```xml
   <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.20</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
```

配置数据库连接

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 123456
    url: jdbc:mysql://localhost:3306/security?serverTimezone=Asia/Shanghai
```





实现repository

```java
    @Resource
    private DataSource dataSource;

    /**
     * save  remember-me to database
     *
     * @return {@link org.springframework.security.web.authentication.rememberme.JdbcTokenRepositoryImpl}
     */
    @Bean
    public PersistentTokenRepository tokenRepository() {
        JdbcTokenRepositoryImpl repository = new JdbcTokenRepositoryImpl();
        repository.setDataSource(dataSource);
        return repository;
    }

```



实现自定义退出



实现自定义退出

```java
@Component
public class WebLogoutSuccessHandler implements LogoutSuccessHandler {
    @Override
    public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        // 自定义的业务逻辑  如果是前后端分离可以不用跳转 返回json 由前端进行控制页面的跳转
        response.sendRedirect("/login.html");
    }
}
```





```java
 @Resource
    private WebLogoutSuccessHandler logoutSuccessHandler;

    @Override
    protected void configure(HttpSecurity http) throws Exception {

        http
                // 在验证用户名和密码之前验证验证码
                .addFilterBefore(captchaCodeFilter, UsernamePassw
                .... // 省略代码
                .and().logout()
                // 退出成功的规则 退出成功会清除session信息
//                .logoutSuccessUrl("/login.html")
                // 自定义退出功能
                .logoutSuccessHandler(logoutSuccessHandler)
                // 删除浏览器的cookie
                .deleteCookies("JSESSIONID")
        ;

    }
```



