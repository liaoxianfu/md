## Spring Security 入门（一） 

### 基于表单的登录验证



搭建项目

spring boot ==2.1.14 Thymeleaf Spring Security



pom.xml

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

首先创建测试的页面 login.html 用来进行登录，index.html 首页 demo1.html 测试页面 一 demo2.html 测试页面2

登录页面必须放在**resource的public文件夹下** **否者无法加载**

![image-20200516181827416](D:\MarkDown\Java\spring-security\img\image-20200516181827416.png)

其他的放在templates文件夹下即可

index.html可以进行其他页面的跳转 demo1 demo2自己随意写点内容即可

login.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登录页面</title>
</head>
<body>
<h1>登录页面</h1>

<form action="/login" method="post">
    用户名<input type="text" name="username" id="username"><br>
    密码 <input type="password" name="password" id="password"><br>
    <input type="submit" value="登录">
</form>
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

<a href="/logout">退出登录</a>

</body>
</html>
```

创建Security配置类AuthoritiesConfig 继承WebSecurityConfigurerAdapter 重写



![image-20200516182458682](D:\MarkDown\Java\spring-security\img\image-20200516182458682.png)



实现加密

```java
@Configuration
public class PasswordEncoderConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

}
```

在内存中用户验证

```java
@EnableWebSecurity
public class AuthoritiesConfig extends WebSecurityConfigurerAdapter {

    @Resource
    private PasswordEncoder passwordEncoder;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                // 在内存中定义一个用户名为admin 密码为123456 角色为admin的用户
                .withUser("admin")
                .password(passwordEncoder.encode("123456"))
                .roles("admin")
                .and()
                // 在内存中定义一个用户名为demo 密码为1234 角色为user的用户
                .withUser("demo")
                .password(passwordEncoder.encode("1234"))
                .roles("user");
    }

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
                .antMatchers("/demo2").hasAnyAuthority("ROLE_admin", "ROLE_user");
    }
}

```

![image-20200516184843367](D:\MarkDown\Java\spring-security\img\image-20200516184843367.png)

使用admin页面进行登录的可以访问demo1和demo2页面  使用demo用户进行登录不能访问demo1页面 出现403禁止访问的错误页面。



自定义登录成功或者失败的页面处理逻辑

这里以ajax登录为例 自定义返回的是json信息而不是页面

定义返回信息类R

```java
@Data
public class R {
    private R() {
    }

    private int code;
    private String message;
    private Object data;

    public static R success() {
        R r = new R();
        r.setCode(200);
        r.setMessage("成功");
        return r;
    }

    public R data(Object data) {
        this.setData(data);
        return this;
    }

    public static R error() {
        R r = new R();
        r.setCode(500);
        r.setMessage("失败");
        return r;
    }
}
```

成功处理函数

```java
@Component
public class MySuccessHandler implements AuthenticationSuccessHandler {

    // 将对象转换成json

    @Resource
    ObjectMapper objectMapper;

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response
            , Authentication authentication) throws IOException, ServletException {
        response.setContentType("application/json; charset=UTF-8");
        Object principal = authentication.getPrincipal();
        R r;
        // 获取用户的具体信息
        if (principal instanceof UserDetails) {
            UserDetails userDetails = (UserDetails) principal;
            String username = userDetails.getUsername();
            r = R.success().data(username);
        }
        r = R.error().data("获取用户出错");
        response.getWriter().write(objectMapper.writeValueAsString(r));
    }
}
```

登录失败处理函数

```java
@Component
public class MyFailureHandler implements AuthenticationFailureHandler {
    @Resource
    ObjectMapper objectMapper;

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response,
                                        AuthenticationException e) throws IOException, ServletException {
        response.setContentType("application/json; charset=UTF-8");
        response.getWriter().write(objectMapper.writeValueAsString(R.error().data("登录失败")));
    }
}
```

修改AuthoritiesConfig

```java
@Override
    protected void configure(HttpSecurity http) throws Exception {
        http.
                .......
                // 默认处理成功之后跳转的页面
//                .defaultSuccessUrl("/index")
                .successHandler(successHandler)
                .failureHandler(failureHandler)
                .and()
            .....
    }
```

设置前端页面使用Ajax进行登录

```xml
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
        let username = $("#username").val()
        let password = $("#password").val()
        let form = {
            "username": username,
            "password": password,
        }
        $.ajax({
            url: "/login",
            method: "post",
            data: form,
            success: (res) => {
                if (res.code === 200) {
                    alert(JSON.stringify(res))
                    location.href = "/index"
                } else {
                    alert(JSON.stringify(res))
                    alert("用户名或者密码错误，请重新填写")
                    $("#username").val("")
                    $("#password").val("")
                }
            }
        })
    })
</script>


</body>
</html>
```





允许最大登录设备处理

```java
@Component
public class MyExpiredSessionStrategy implements SessionInformationExpiredStrategy {
    @Resource
    ObjectMapper objectMapper;
    
    @Override
    public void onExpiredSessionDetected(SessionInformationExpiredEvent event) throws IOException, ServletException {
        HttpServletResponse response = event.getResponse();
        response.setContentType("application/json; charset=UTF-8");
        response.getWriter().write(objectMapper.writeValueAsString(R.error().data("您已经在其他地方登录，被迫下线")));
    }
}

```



```java
   @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.
 ......
                .and()
                // session管理
                .sessionManagement()
                // 设置允许最大登录的机器为1
                .maximumSessions(1)
                // 设置当登录后允许新的机器登录 但是之前的会被设置为过期
                .maxSessionsPreventsLogin(false)
                // 自定义的过期处理方式
                .expiredSessionStrategy(expiredSessionStrategy);
        
    }
```

使用两个浏览器进行登录后 再操作第一次登录的那个浏览器就会被批下线

![image-20200516214258524](D:\MarkDown\Java\spring-security\img\image-20200516214258524.png)





使用数据库用户登录

RBAC表结构

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

-- ----------------------------
-- Table structure for sys_menu
-- ----------------------------
DROP TABLE IF EXISTS `sys_menu`;
CREATE TABLE `sys_menu`  (
  `id` int(11) UNSIGNED NOT NULL AUTO_INCREMENT,
  `menu_name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `url` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `menu_pid` int(11) NULL DEFAULT NULL,
  `menu_pids` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `is_leaf` tinyint(4) NULL DEFAULT NULL,
  `status` int(11) NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 6 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for sys_org
-- ----------------------------
DROP TABLE IF EXISTS `sys_org`;
CREATE TABLE `sys_org`  (
  `id` int(11) UNSIGNED NOT NULL AUTO_INCREMENT,
  `org_pid` int(11) NULL DEFAULT NULL,
  `org_pids` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `is_leaf` tinyint(4) NULL DEFAULT NULL,
  `org_name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `sort` int(11) NULL DEFAULT NULL,
  `level` int(11) NULL DEFAULT NULL,
  `status` int(11) NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 4 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for sys_role
-- ----------------------------
DROP TABLE IF EXISTS `sys_role`;
CREATE TABLE `sys_role`  (
  `id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `role_name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `role_code` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `role_desc` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `sort` int(11) NULL DEFAULT NULL,
  `status` int(11) NULL DEFAULT NULL,
  `create_time` datetime(0) NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 4 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for sys_role_menu
-- ----------------------------
DROP TABLE IF EXISTS `sys_role_menu`;
CREATE TABLE `sys_role_menu`  (
  `id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `role_id` int(11) NOT NULL,
  `menu_id` int(11) NOT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 8 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for sys_role_user
-- ----------------------------
DROP TABLE IF EXISTS `sys_role_user`;
CREATE TABLE `sys_role_user`  (
  `id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `user_id` int(11) NOT NULL,
  `role_id` int(11) NOT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 4 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for sys_user
-- ----------------------------
DROP TABLE IF EXISTS `sys_user`;
CREATE TABLE `sys_user`  (
  `id` int(11) UNSIGNED NOT NULL AUTO_INCREMENT,
  `username` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `password` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `create_datetime` datetime(0) NULL DEFAULT NULL,
  `org_id` int(11) NULL DEFAULT NULL,
  `enable` tinyint(4) NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 3 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

SET FOREIGN_KEY_CHECKS = 1;

```





使用数据库进行登录其实就是自定义登录过程，需要自己实现`org.springframework.security.core.userdetails.UserDetailsService`接口

该接口只有一个方法`loadUserByUsername(String username)`方法返回的是`UserDetails`接口

```java
public interface UserDetails extends Serializable {
// 用户的权限信息
	Collection<? extends GrantedAuthority> getAuthorities();
// 用户的密码
	String getPassword();
// 用户姓名
	String getUsername();
// 用户的账户没有过期
	boolean isAccountNonExpired();
// 用户的账户没有锁定
	boolean isAccountNonLocked();
// 指示用户的凭据（密码）是否已过期。 凭证过期防止认证。
	boolean isCredentialsNonExpired();
//指示用户是否被启用或禁用。 禁用的用户无法进行身份验证。
	boolean isEnabled();
}

```

从这里我们通过数据库可以以加载除权限集合信息外的所有基础信息，通过`MyUserDetails`类实现`UserDetails`加载基础信息。

```java
@Data
@ToString
public class MyUserDetails implements UserDetails {
    private String password;
    private String username;
    private boolean enable;
    private Collection<? extends GrantedAuthority> authorities;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return authorities;
    }

    @Override
    public String getPassword() {
        return password;
    }

    @Override
    public String getUsername() {
        return username;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return enable;
    }
}

```

通过Mybatis从数据库中查询对应的信息

```java
 MyUserDetails getUserDetailsByUsername(String username);
```

```xml
  <select id="getUserDetailsByUsername" resultType="com.liao.springsecurityform.security.MyUserDetails">
        select id,
               username,
               password,
               enable
        from security.sys_user
        where username = #{username}
    </select>
```

权限分为两类：1、指后台指定角色能够访问的url 2、角色本身就是一种权限，所以将角色也加载进去

实现MyUserDetailService

```java
public class MyUserDetailService implements UserDetailsService {
    @Resource
    UserMapper userMapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 加载基础信息
        MyUserDetails userDetails = userMapper.getUserDetailsByUsername(username);

        // 根据用户名加载角色信息
        List<String> roles = userMapper.getRoleByUserName(username);

        // 根据角色加载url
        List<String> authorities = userMapper.getAuthoritesByRoles(roles);
        // 角色是一个特殊的权限 将用户角色转换成权限
        roles = roles.stream().map(role -> "ROLE_" + role).collect(Collectors.toList());
        authorities.addAll(roles);
        // 设置用户权限
        userDetails.setAuthorities(
                AuthorityUtils.commaSeparatedStringToAuthorityList(String.join(",", authorities)));
        // 通过用户角色列表加载资源权限列表

        return userDetails;
    }
}
```

将之前的内存用户替换

```java
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailService)
                .passwordEncoder(passwordEncoder);
    }
```

动态加载权限,自定义权限规则

```java
@Slf4j
@Component("rbacService")
public class MyRBACService {
    @Resource
    private UserMapper userDao;
    public static AntPathMatcher antPathMatcher = new AntPathMatcher();
    /**
     * 判断用户是否具有request的访问权限
     */
    public boolean hasPermission(HttpServletRequest request, Authentication authentication) {

        Object principal = authentication.getPrincipal();
        if (principal instanceof UserDetails) {
            UserDetails userDetails = (UserDetails) principal;
            String username = userDetails.getUsername();
            String requestURI = request.getRequestURI();
            // 获取该用户能够有权访问的url
            List<String> urls = userDao.getUrlByUsername(username);
            return urls.stream().anyMatch(
                    // 判断以*结尾的数据能够访问任意的数据
                    url -> {
                        log.info("获得的uri为{}", requestURI);
                        if (url.endsWith("/*")){
                            String replace = url.replace("/*","");
                            return requestURI.startsWith(replace);
                        }
                        return antPathMatcher.match(url, requestURI);
                    }
            );
        }
        return false;
    }
}

```

修改配置文件

```java

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.
  ....
                // 允许admin用户访问/下所有的资源
                .antMatchers("/*").hasAnyRole("admin")
                // 使用自定义的权限控制
                .anyRequest().access("@rbacService.hasPermission(request,authentication)")
                /***************************************************************
                // 允许admin角色用户登录
                .antMatchers("/demo1").hasAnyRole("admin")
                // 角色本身也是一种特殊的权限 当用户属于一种角色后就会默认存在一个ROLE_角色名的权限
                // 也就是允许拥有ROLE_admin和ROLE_user权限的用户登录 即角色为admin或者user的用户登录
                .antMatchers("/demo2").hasAnyAuthority("ROLE_admin", "ROLE_user")
                 ***********************************************************************/
                .and()
                // session管理
                .sessionManagement()
                // 设置允许最大登录的机器为1
                .maximumSessions(1)
                // 设置当登录后允许新的机器登录 但是之前的会被设置为过期
                .maxSessionsPreventsLogin(false)
                // 自定义的过期处理方式
                .expiredSessionStrategy(expiredSessionStrategy);

    }

```

定义退出功能和记住我功能

退出登录

```java
 @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.
                // 取消csrf的限制 否者无法加载页面
                        csrf().disable()
          .....
                .and()
                .logout()
                // 退出成功的规则 退出成功会清除session信息
//                .logoutSuccessUrl("/login.html")
                // 自定义退出功能
             .logoutSuccessHandler(logoutSuccessHandler)
                // 删除浏览器的cookie
                .deleteCookies("JSESSIONID")
               ......

    }
```

logoutSuccessHandler

```java
@Component
public class MyLogoutSuccessHandler implements LogoutSuccessHandler {
    @Override
    public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        // 自定义的业务逻辑  如果是前后端分离可以不用跳转 返回json 由前端进行控制页面的跳转
        response.sendRedirect("/login.html");
    }
}
```

记住我功能

```java
 @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.
          ........
                .logout()
                // 退出成功的规则 退出成功会清除session信息
//                .logoutSuccessUrl("/login.html")
                // 自定义退出功能
                .logoutSuccessHandler(logoutSuccessHandler)
                // 删除浏览器的cookie
                .deleteCookies("JSESSIONID")
                .and()
                .rememberMe()
                // 定义过期时间 一周
                .tokenValiditySeconds(7*24*60*60)
                // 定义记住我 在前端的name
                .rememberMeParameter("rememberMe")
                // 在cookie中存放显示的名称
                .rememberMeCookieName("remember-cookies")
                // 将token存储在数据库中
                .tokenRepository(tokenRepository())
                .and()
                // 验证规则
                .authorizeRequests()
                // 允许所有用户包括未登录访问
........

    }
 @Resource
    private DataSource dataSource;

    @Bean
    public PersistentTokenRepository tokenRepository() {
        JdbcTokenRepositoryImpl repository = new JdbcTokenRepositoryImpl();
        repository.setDataSource(dataSource);
        return repository;
    }

```

图像验证码

引入pom.xml

```xml
  <dependency>
            <groupId>com.github.penggle</groupId>
            <artifactId>kaptcha</artifactId>
            <version>2.3.2</version>
  </dependency>
```

创建配置文件

kaptcha/kaptcha.properties

```properties
kaptcha.border=yes
kaptcha.border.color=105,179,90
kaptcha.textproducer.char.length=4
kaptcha.textproducer.font.names=宋体,楷体,微软雅黑
kaptcha.textproducer.font.size=40
kaptcha.textproducer.font.color=black
kaptcha.image.width=110
kaptcha.image.height=50
```

创建配置类

```java
@Configuration
@PropertySource(value = {"classpath:kaptcha/kaptcha.properties"})
public class KaptchaConfig {

    @Value("${kaptcha.border}")
    String border;
    @Value("${kaptcha.border.color}")
    String borderColor;
    @Value("${kaptcha.textproducer.char.length}")
    String charLength;
    @Value("${kaptcha.textproducer.font.names}")
    String fontName;
    @Value("${kaptcha.textproducer.font.size}")
    String fontSize;
    @Value("${kaptcha.textproducer.font.color}")
    String fontColor;
    @Value("${kaptcha.image.width}")
    String imgWidth;
    @Value("${kaptcha.image.height}")
    String imgHeight;

    @Bean
    public DefaultKaptcha getDefaultKaptcha() {
        DefaultKaptcha captchaProducer = new DefaultKaptcha();
        Properties properties = new Properties();
        properties.setProperty("kaptcha.border", border);
        properties.setProperty("kaptcha.border.color", borderColor);
        properties.setProperty("kaptcha.textproducer.font.color", fontColor);
        properties.setProperty("kaptcha.image.width", imgWidth);
        properties.setProperty("kaptcha.image.height", imgHeight);
        properties.setProperty("kaptcha.textproducer.font.size", fontSize);
        properties.setProperty("kaptcha.session.key", Constants.KAPTCHA_SESSION_KEY);
        properties.setProperty("kaptcha.textproducer.char.length", charLength);
        properties.setProperty("kaptcha.textproducer.font.names", fontName);
        Config config = new Config(properties);
        captchaProducer.setConfig(config);
        return captchaProducer;
    }
}
```



创建判断类

```java
@Data
public class KaptchaImageVo {
    private String code;
    private LocalDateTime expiredTime;

    public KaptchaImageVo(String code, int expiredAfterSeconds){
        this.code = code;
        this.expiredTime = LocalDateTime.now().plusSeconds(expiredAfterSeconds);
    }

    /**
     * 判断是否过期
     */
    public boolean isExpired(){
        return expiredTime.isBefore(LocalDateTime.now());
    }

}
```







创建返回验证码图片的URL

```java
@RestController
public class CodeController {
    @Resource
    private Producer captchaProducer;

    @RequestMapping("/kaptcha")
    public void getKaptchaImage(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpSession session = request.getSession();
        response.setDateHeader("Expires", 0);
        response.setHeader("Cache-Control", "no-store, no-cache, must-revalidate");
        response.addHeader("Cache-Control", "post-check=0, pre-check=0");
        response.setHeader("Pragma", "no-cache");
        response.setContentType("image/jpeg");
        //生成验证码
        String capText = captchaProducer.createText();
        KaptchaImageVo imageVo = new KaptchaImageVo(capText, 2 * 60);
        session.setAttribute(Constants.KAPTCHA_SESSION_KEY, imageVo);
        //向客户端写出
        BufferedImage bi = captchaProducer.createImage(capText);
        ServletOutputStream out = response.getOutputStream();
        ImageIO.write(bi, "jpg", out);
        try {
            out.flush();
        } finally {
            out.close();
        }
    }
}
```

自定义验证操作

```java
@Component
public class CaptchaCodeFilter extends OncePerRequestFilter {
    @Resource
    MyFailureHandler myFailureHandler;


    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        // 判断请求的url为 login和提交的方式post才能进行操作
        String uri = request.getRequestURI();
        // 去除空格和转换成小写
        String method = request.getMethod().trim().toLowerCase();
        // url为/login 而且
        String loginUrl = "/login";
        if (uri.equals(loginUrl)&& "post".equals(method)){
            // 捕获异常信息
            try {
                validate(request);
            }catch (SessionAuthenticationException e){
                myFailureHandler.onAuthenticationFailure(request,response,e);
                return;
            }
        }
        // 继续向下执行
        filterChain.doFilter(request, response);
    }

    private void validate(HttpServletRequest request){
        HttpSession session = request.getSession();
        // 获取前端传递回来的验证码数据
        String captcha = request.getParameter("captcha");
        // 从session中获取保存的验证码谜底信息
        KaptchaImageVo kaptchaImageVo = (KaptchaImageVo) session.getAttribute(Constants.KAPTCHA_SESSION_KEY);
        // 前端传递回来的信息为空
        if (StringUtils.isEmpty(captcha)){
            throw new SessionAuthenticationException("没有填写验证码");
        }

        if (Objects.isNull(kaptchaImageVo)){
            throw new SessionAuthenticationException("验证码不存在");
        }

        // 验证码过期了
        if (kaptchaImageVo.isExpired()){
            throw new SessionAuthenticationException("验证码已经过期了");
        }

        if (!kaptchaImageVo.getCode().equals(captcha)){
            throw new SessionAuthenticationException("验证码填写不正确");
        }
    }
}

```

注册filter

```java
@Override
    protected void configure(HttpSecurity http) throws Exception {
        http.addFilterBefore(captchaCodeFilter, UsernamePasswordAuthenticationFilter.class)
                .csrf().disable()
                .logout()
          ......
            .antMatchers("/login.html", "/login","/kaptcha").permitAll() // 需要放行/kaptcha用于生成验证码
    }
```

logign.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<!--表单登录-->
<!--<form action="/login" method="post">-->
<!--    用户名<input name="username"><br>-->
<!--    密码 <input type="password" name="password"><br>-->
<!--    <input type="submit" value="登录">-->
<!--</form>-->

<!--rest 登录-->
<div>
    用户名<input name="username" id="username"><br>
    密码 <input type="password" id="password"><br>
    验证码<input type="text" id="kaptcha">
    <img src="/kaptcha" alt="验证码" id="img_cha"><br>
    <button id="btn">登录</button>
    <br>
    一周之内免登录 <input type="checkbox" id="rememberMe" name="remember-me">
</div>
<script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.5.1/jquery.js"></script>
<script>

    $("#img_cha").click(() => {
        console.log("hehhe");
        $('#img_cha').attr("src", "/kaptcha");
    })


    $("#btn").click(() => {
        let username = $("#username").val()
        let password = $("#password").val()
        let checked = $("#rememberMe").is(":checked")
        let captcha = $("#kaptcha").val()
        let form = {
            "username": username,
            "password": password,
            "captcha": captcha,
            "remember-me": checked,
        }
        $.ajax({
            url: "/login",
            method: "post",
            data: form,
            success: (res) => {
                if (res.code === 200) {
                    location.href = "/index"
                } else {
                    console.log(res)
                    alert(res.data)
                    $("#username").val("")
                    $("#password").val("")
                }
            }
        })
    })
</script>

</body>
</html>
```
MyFailureHandler修改


```java
@Component
public class MyFailureHandler implements AuthenticationFailureHandler {
    @Resource
    ObjectMapper objectMapper;

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response,AuthenticationException e) throws IOException, ServletException {

        String errMsg = "用户名或者密码填写错误";
        // 判断是否是验证码的错误
        if (e instanceof SessionAuthenticationException) {
            errMsg = "验证码填写错误";
        }


        response.setContentType("application/json; charset=UTF-8");
        response.getWriter().write(objectMapper.writeValueAsString(R.error().data(errMsg)));
    }
}
```

