### 认证与授权

在上一节，我们自定义了内存中的用户，定义了两种角色分别是`admin`和`demo`.在spring security中默认的角色是`user`。

基于内存的多角色支持。在上一节中我们使用了在`Security`中重写`protected void configure(AuthenticationManagerBuilder auth)`方法 进行多用户的支持。这里我们采用一种新的方式。通过自定义实现一个`UserDetailService` 通过继承树我们可以看到`InMemoryUserDetailsManager`通过实现`UserDetailsManager`、而`UserDetailsManager`继承了`UserDetailsService`。因此我们可以使用`InMemoryUserDetailsManager`创建内存对象。

```java
    @Override
    @Bean
    public UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        manager.createUser(User.withUsername("admin")
                .password(passwordEncoder.encode("123456"))
                .roles("admin").build()
        );
        manager.createUser(User.withUsername("demo")
                .password(passwordEncoder.encode("1234"))
                .roles("demo").build()
        );
        return manager;
    }

```



自定义基于数据库的RBAC权限控制

RBAC数据库设计



#### 准备工作

建表

```sql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;


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



创建 用户-角色-权限表 用户和角色是多对多的关系 角色和menu（这里使用url代指）也是多对多的关系。

这里创建两个用户`admin`和`demo` 两个权限 `admin` 和 `common` 其中admin权限 拥有所有的权限 也就是能够访问所有的url common只能访问部分的数据。

创建项目 `learn-04` 

pom.xml

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
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
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

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.20</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.3</version>
        </dependency>
    </dependencies>
```



创建springboot项目准备步骤

启动文件

```java
@MapperScan("com.liao.security.learn04.mapper")// 指定mapper包路径
@SpringBootApplication
public class RbacApplication {
    public static void main(String[] args) {
        SpringApplication.run(RbacApplication.class, args);
    }
}
```

创建配置文件 application.yml

```yml
server:
  port: 8084
mybatis:
  mapper-locations: classpath:mapper/*.xml
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 123456
    url: jdbc:mysql://localhost:3306/security?serverTimezone=Asia/Shanghai
```



 创建登录页面

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

准备mapper和对应的xml文件 `UserMapper.java`和`UserMapper.xml`

```java
public interface UserMapper {
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.liao.security.learn04.mapper.UserMapper">
    <resultMap id="BaseResultMap" type="com.liao.security.learn04.domain.User">
        <!--@mbg.generated-->
        <!--@Table sys_user-->
        <id column="id" jdbcType="INTEGER" property="id"/>
        <result column="username" jdbcType="VARCHAR" property="username"/>
        <result column="password" jdbcType="VARCHAR" property="password"/>
        <result column="create_datetime" jdbcType="TIMESTAMP" property="createDatetime"/>
        <result column="org_id" jdbcType="INTEGER" property="orgId"/>
        <result column="enable" jdbcType="TINYINT" property="enable"/>
    </resultMap>
    <sql id="Base_Column_List">
        <!--@mbg.generated-->
        id, username, `password`, create_datetime, org_id, `enable`
    </sql>
</mapper>
```



创建基础的配置类 PasswordEncoderConfig 和 统一返回 R

```java
@Configuration
public class PasswordEncoderConfig {
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

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



自定义登录成功和失败的Handler

```java
@Configuration
public class LoginFailHandler implements AuthenticationFailureHandler {
    @Resource
    private ObjectMapper objectMapper;

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException e) throws IOException, ServletException {
        R data = R.error().data("登录失败");
        response.setContentType("application/json; charset=UTF-8");
        response.getWriter().write(objectMapper.writeValueAsString(data));
    }
}
```

```java
@Configuration
public class LoginSuccessHandler implements AuthenticationSuccessHandler {

    /**
     * 对象转换为json
     */
    @Resource
    private ObjectMapper objectMapper;

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication)
            throws IOException, ServletException {

        R success = R.success().data("登录成功");
        response.setContentType("application/json;charset=utf-8");
        response.getWriter().write(objectMapper.writeValueAsString(success));
    }
}

```



创建spring security的配置类

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {}
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {}

}
```



#### 配置自定义登录以及权限控制

实现UserDetail

```java
@Data
@ToString
public class WebUserDetails implements UserDetails {
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



创建RBAC查询接口 在Usermapper下创建相应的接口

```java
public interface UserMapper {

    WebUserDetails getUserDetailsByUsername(String username);

    List<String> getRoleByUserName(String username);

    List<String> getAuthoritesByRoles(@Param("roles") List<String> roles);

    List<String> getUrlByUsername(String username);
}
```

在UserMapper.xml下创建对应的sql

```xml
    <select id="getUserDetailsByUsername" resultType="com.liao.security.learn04.domain.WebUserDetails">
        select id,
               username,
               password,
               enable
        from security.sys_user
        where username = #{username}

    </select>
    <select id="getRoleByUserName" resultType="java.lang.String">
        SELECT role_code
        FROM sys_user
                 LEFT JOIN sys_role_user ON sys_role_user.user_id = sys_user.id
                 LEFT JOIN sys_role ON sys_role_user.role_id = sys_role.id
        WHERE username = #{username}
    </select>
    <select id="getAuthoritesByRoles" resultType="java.lang.String">
        SELECT
        url
        FROM
        sys_menu
        LEFT JOIN sys_role_menu ON sys_menu.id = sys_role_menu.menu_id
        LEFT JOIN sys_role ON sys_role_menu.role_id = sys_role.id
        WHERE
        role_code IN
        <foreach collection="roles" item="role" open="(" close=")" separator=",">
            #{role}
        </foreach>
    </select>
    <select id="getUrlByUsername" resultType="java.lang.String">
        SELECT url
        FROM sys_menu
        LEFT JOIN sys_role_menu ON sys_menu.id = sys_role_menu.menu_id
        LEFT JOIN sys_role ON sys_role_menu.role_id = sys_role.id
        LEFT JOIN sys_role_user ON sys_role_user.role_id = sys_role.id
        LEFT JOIN sys_user ON sys_role_user.user_id = sys_user.id
        WHERE username = #{username}
    </select>
```



实现UserDetailService

```java
@Component
public class WebUserDetailService implements UserDetailsService {
    @Resource
    private UserMapper userMapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
// 加载基础信息
        WebUserDetails userDetails = userMapper.getUserDetailsByUsername(username);

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



创建RBAC权限控制

```java
@Slf4j
@Component("rbacService")
public class WebRbacService {

    @Resource
    private UserMapper userDao;
    public static AntPathMatcher antPathMatcher = new AntPathMatcher();

    /**
     * 判断用户是否具有request的访问权限
     */
    public boolean hasPermission(HttpServletRequest request, Authentication authentication) {

        boolean match = false;
        Object principal = authentication.getPrincipal();
        if (principal instanceof UserDetails) {
            UserDetails userDetails = (UserDetails) principal;
            String username = userDetails.getUsername();
            String requestUri = request.getRequestURI();
            // 获取该用户能够有权访问的url
            List<String> urls = userDao.getUrlByUsername(username);
            match = urls.stream().anyMatch(
                    // 判断以*结尾的数据能够访问任意的数据
                    url -> {
                        log.info("获得的uri为{}", requestUri);
                        if (url.endsWith("/*")) {
                            String replace = url.replace("/*", "");
                            return requestUri.startsWith(replace);
                        }
                        return antPathMatcher.match(url, requestUri);
                    }
            );
        }
        return match;
    }

}

```



配置Springsecurity javaconfig

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Resource
    private WebUserDetailService userDetailService;

    @Resource
    private PasswordEncoder passwordEncoder;


    @Resource
    private LoginSuccessHandler successHandler;

    @Resource
    private LoginFailHandler failHandler;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 自定义用户的加载规则
        auth.userDetailsService(userDetailService).passwordEncoder(passwordEncoder);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                // 取消csrf的限制 否则无法加载
                .csrf().disable()
                // 登录页面
                .formLogin().loginPage("/login.html")
                // 登录处理url
                .loginProcessingUrl("/login")
                // 登录成功和失败的Handler
                .successHandler(successHandler).failureHandler(failHandler)
                .and()
                // 认证规则
                .authorizeRequests()
                // 允许所有人访问这两个网址
                .antMatchers("/login.html","/login").permitAll()
                // 认证后跳转的url
                .antMatchers("/index").authenticated()
                // 允许admin用户访问/下所有的资源
                .antMatchers("/*").hasAnyRole("admin")
                // 使用自定义的权限控制
                .anyRequest().access("@rbacService.hasPermission(request,authentication)");
    }
}
```



