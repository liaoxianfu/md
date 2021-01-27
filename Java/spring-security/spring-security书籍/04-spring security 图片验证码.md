### 图片验证码

为了防止恶意用户无限制的重试，往往在登录页面引入验证码机制。我们在`learn-03`项目的基础上进行改进引入验证码。



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
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <!--加载图片验证-->
        <dependency>
            <groupId>com.github.penggle</groupId>
            <artifactId>kaptcha</artifactId>
            <version>2.3.2</version>
        </dependency>
    </dependencies>

```



为了实现验证码首先需要一个获取图形验证码的API。在这里使用开源工具kaptcha

创建kaptcha的配置文件 kaptcha.properties

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

创建配置类 KaptchaConfig

```java
@Configuration
@PropertySource(value = {"classpath:kaptcha/kaptcha.properties"}) // 配置文件存放的路径
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



创建验证码验证以及过期时间配置 KaptchaImageVo

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

创建获取验证码图片的网址

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
        // 设置验证码的有效时间
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



创建过滤器

```java
@Component
public class CaptchaCodeFilter extends OncePerRequestFilter {
    @Resource
    private LoginFailHandler loginFailHandler;


    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        // 判断请求的url为 login和提交的方式post才能进行操作
        String uri = request.getRequestURI();
        // 去除空格和转换成小写
        String method = request.getMethod().trim().toLowerCase();
        // url为/login 而且
        String loginUrl = "/login";
        if (uri.equals(loginUrl) && "post".equals(method)) {
            // 捕获异常信息
            try {
                validate(request);
            } catch (SessionAuthenticationException e) {
                loginFailHandler.onAuthenticationFailure(request, response, e);
                return;
            }
        }
        // 继续向下执行
        filterChain.doFilter(request, response);
    }

    private void validate(HttpServletRequest request) {
        HttpSession session = request.getSession();
        // 获取前端传递回来的验证码数据
        String captcha = request.getParameter("captcha");
        // 从session中获取保存的验证码谜底信息
        KaptchaImageVo kaptchaImageVo = (KaptchaImageVo) session.getAttribute(Constants.KAPTCHA_SESSION_KEY);
        // 前端传递回来的信息为空
        if (StringUtils.isEmpty(captcha)) {
            throw new SessionAuthenticationException("没有填写验证码");
        }

        if (Objects.isNull(kaptchaImageVo)) {
            throw new SessionAuthenticationException("验证码不存在");
        }

        // 验证码过期了
        if (kaptchaImageVo.isExpired()) {
            throw new SessionAuthenticationException("验证码已经过期了");
        }

        if (!kaptchaImageVo.getCode().equals(captcha)) {
            throw new SessionAuthenticationException("验证码填写不正确");
        }
    }
}

```



修改失败的处理方式 加入验证码失败的提示

```java
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException e) throws IOException, ServletException {


        String errMsg = "用户名或者密码填写错误";
        // 判断是否是验证码的错误 因为 CaptchaCodeFilter抛出的是SessionAuthenticationException异常
        if (e instanceof SessionAuthenticationException) {
            errMsg = "验证码填写错误";
        }

        response.setContentType("application/json; charset=UTF-8");
        response.getWriter().write(objectMapper.writeValueAsString(R.error().data(errMsg)));
    }
```







修改spring security的配置文件加如验证码的过滤器

```java
   @Resource
    private CaptchaCodeFilter captchaCodeFilter;

@Override
    protected void configure(HttpSecurity http) throws Exception {
        http.addFilterBefore(captchaCodeFilter, UsernamePasswordAuthenticationFilter.class)
                .csrf().disable()
                .logout()
          ......
            .antMatchers("/login.html", "/login","/kaptcha").permitAll() // 需要放行/kaptcha用于生成验证码
    }
```



