### springboot jwt rbac整合



#### 什么是jwt?

> Json web token (JWT), 是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准（[(RFC 7519](https://link.jianshu.com?t=https://tools.ietf.org/html/rfc7519)).该token被设计为紧凑且安全的，特别适用于分布式站点的单点登录（SSO）场景。JWT的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该token也可直接被用于认证，也可被加密。

jwt请求流程

9，

1、首先用户使用用户名和密码进行登录

2、登录验证成功后系统会使用私钥创建有一个jwt 

3、服务器将这个jwt返回给浏览器

4、浏览器在每次请求时将这个jwt 在请求投上携带

5、服务器验证这个jwt

6、返回相应的请求资源

**一般在分布式系统上进行使用。**

#### 整合springboot与jwt

pom.xml

```xml
   <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.2</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
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
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.1</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
```

#### 编写jwt工具集

```java
package com.liao.spring.security.jwt.config;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.impl.DefaultClaims;
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;

import java.util.Date;

/**
 * @author liao
 * @since 2020/5/26 10:39
 */

@Slf4j
@Data
@Component
@ConfigurationProperties(prefix = "jwt")
public class JwtTokenUtil {
    private String secret;
    private Long expiration;
    private String header;


    /**
     * 生成token
     */
    public String generateToken(UserDetails userDetails) {
        Claims claims = new DefaultClaims();
        claims.setSubject(userDetails.getUsername());
        return generateToken(claims);
    }


    /**
     * 生成jwt token
     * @author liao
     * @param claims {@link Claims}
     * @return jwt-token
     *
     */
    private String generateToken(Claims claims) {
        // 过期时间
        Date expirationDate = new Date(System.currentTimeMillis() + expiration);
        return Jwts.builder().setClaims(claims).setExpiration(expirationDate)
                .signWith(SignatureAlgorithm.HS512, secret)
                .compact();
    }


    /**
     * 通过token获取用户名
     */
    public String getUserNameFromToken(String token) {
        String userName;
        try {
            Claims claims = getClaimsFromToken(token);
            userName = claims.getSubject();
        } catch (Exception e) {
            log.error("getUserNameFromToken error error info{}", e.getMessage());
            userName = null;
        }
        return userName;
    }

    /**
     * 通过token获取claims
     */
    private Claims getClaimsFromToken(String token) {
        Claims claims = null;
        try {
            // 解析获取claims
            claims = Jwts.parser().setSigningKey(secret).parseClaimsJws(token).getBody();
        } catch (Exception e) {
            log.error("getClaimsFromToken error error info {}", e.getMessage());
        }
        return claims;
    }

    /**
     * 判断token是否过期
     */
    public boolean isTokenNotExpired(String token) {
        try {
            Claims claims = getClaimsFromToken(token);
            Date expiration = claims.getExpiration();
            return expiration.after(new Date());
        } catch (Exception e) {
            log.error("获取过期时间出错,{}", e.getMessage());
            // 获取token出错之后默认设置为过期
            return false;
        }
    }


    /**
     * 刷新token
     */
    public String refreshToken(String token) {
        try {
            Claims claims = getClaimsFromToken(token);
            return generateToken(claims);
        } catch (Exception e) {
            log.error("refreshToken error err info{}", e.getMessage());
            return null;
        }
    }

    /**
     *  验证token
     */
    public boolean validateToken(String token, UserDetails userDetails) {
        String userName = getUserNameFromToken(token);
        // 判断token是否过期和用户名是否正确
        return isTokenNotExpired(token) && userName.equals(userDetails.getUsername());
    }

}

```



#### jwt 验证授权

从请求头中获取header， 判断当请求头中发现了携带了jwt的请求头 就从jwt请求头中获取jwt，从jwt中获取获取用户名 而且判断当前验证上下文中没有该数据 然后利用获取的用户名获取userDetails。验证jwt与userDetails 判断jwt是否合法（是否过期 用户名是否正确） 验证成功后 进行授权。最后 不管是否验证成功 都进行放行 进行后续的处理。

```java
package com.liao.spring.security.jwt.config;

import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.annotation.Resource;
import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @author liao
 * @since 2020/5/26 17:58
 */
@Component
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {


    @Resource
    private JwtTokenUtil jwtTokenUtil;
    @Resource
    private JwtUserDetailServiceImpl userDetailService;


    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        // 从请求头中获取header
        String token = request.getHeader(jwtTokenUtil.getHeader());
        // 判断当请求头中发现了携带了jwt的请求头 就从jwt请求头中获取jwt
        // 从jwt中获取获取用户名 而且判断当前验证上下文中没有该数据 然后利用获取的用户名获取userDetails
        // 验证jwt与userDetails 判断jwt是否合法（是否过期 用户名是否正确） 验证成功后 进行授权
        // 最后 不管是否验证成功 都进行放行 进行后续的处理

        if (!StringUtils.isEmpty(token)) {
            String userName = jwtTokenUtil.getUserNameFromToken(token);
            // 判断用户名是否为空
            if (userName != null && SecurityContextHolder.getContext().getAuthentication() == null) {

                UserDetails userDetails = userDetailService.loadUserByUsername(userName);
                boolean validateToken = jwtTokenUtil.validateToken(token, userDetails);
                if (validateToken) {
                    // 给使用该jwt的用户进行授权
                    UsernamePasswordAuthenticationToken authenticationToken =
                            new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                    SecurityContextHolder.getContext().setAuthentication(authenticationToken);
                }
            }
        }
        filterChain.doFilter(request, response);
    }
}

```

配置验证信息

```java
package com.liao.spring.security.jwt.config;


import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;
import org.springframework.security.web.csrf.CookieCsrfTokenRepository;

import javax.annotation.Resource;

/**
 * @author liao
 * @since 2020/5/28 14:40
 */
@Configuration
public class JwtAuthConfig extends WebSecurityConfigurerAdapter {
    @Resource
    private JwtUserDetailServiceImpl jwtUserDetailService;

    @Resource
    private PasswordEncoder passwordEncoder;



    @Resource
    private JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 在用户名密码验证前进行jwt 验证
        http.addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class)
                .csrf()
                /*允许js操作cookie*/
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
                // 忽略 登录需要携带cookie信息
                .ignoringAntMatchers("/login")
                .and()
                .authorizeRequests()
                .antMatchers("/login", "/refresh").permitAll()
                .antMatchers("/*").hasAnyRole("admin")
                .anyRequest().access("@rbacService.hasPermission(request,authentication)")
                .and()
                // 不再进行Session存储
                .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS);

    }


    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(jwtUserDetailService).passwordEncoder(passwordEncoder);
    }

    @Bean
    @Override
    protected AuthenticationManager authenticationManager() throws Exception {
        return super.authenticationManager();
    }
}

```

