Spring Security
===

1、  开启
---

- 添加依赖

```xml
<dependency>    
    <groupId>org.springframework.boot</groupId>    
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

- 默认配置Basic认证

  ```yml
  security:  
  	basic:    
  		enabled: true
  ```

  

- 原理：看看就好

  ![QQæªå¾20180707111356.png](https://mrbird.cc/img/QQ%E6%88%AA%E5%9B%BE20180707111356.png)

2、  用户自定义
---

### 2.1、  用户自定义认证

- 过程

  需要实现`UserDetailService`

  ```java
  @Configuration
  public class UserDetailService implements UserDetailsService {    
      @Autowired
      //密码加密工具
      private PasswordEncoder passwordEncoder;    
      @Override    
      public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
          
          // 模拟一个用户，替代数据库获取逻辑        
          MyUser user = new MyUser();        
          user.setUserName(username);        
          user.setPassword(this.passwordEncoder.encode("123456"));        
          // 输出加密后的密码       
          System.out.println(user.getPassword());        
          return new User(
              username, 
              user.getPassword(), 
              user.isEnabled(),                
              user.isAccountNonExpired(), 
              user.isCredentialsNonExpired(),               		                                   user.isAccountNonLocked(),             
              AuthorityUtils.
              commaSeparatedStringToAuthorityList("admin")//admin 权限
          );    
      }
  }
  ```

- 替换登录页

  在src/main/resources/resources目录下定义一个login.html

  ```java
  @Override
  protected void configure(HttpSecurity http) throws Exception {    
      http.formLogin() // 表单登录            
      // http.httpBasic() // HTTP Basic            
      .loginPage("/login.html")             
      .loginProcessingUrl("/login")            
      .and()            
      .authorizeRequests()//授权配置            
      .antMatchers("/login.html").permitAll()            
      .anyRequest()  // 所有请求            
      .authenticated(); // 都需要认证
      .and().csrf().disable();
  }
  ```

`.loginPage("/login.html")`指定了跳转到登录页面的请求URL

`.loginProcessingUrl("/login")`对应页面表单的`action="/login"`，

`.antMatchers("/login.html").permitAll()`表示跳转到登录页面的请求不被拦截，否则会进入无限循环。



### 2.2、  用户访问资源控制

- 静态页面如果没有登录 直接跳转登陆页面 否则返回JSON数据  状态码401

  - 配置

    ```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {    
    	http.formLogin() // 表单登录            
        // http.httpBasic() // HTTP Basic            
        .loginPage("/authentication/require")//登录跳转URL         
        .loginProcessingUrl("/login") // 处理表单登录 URL            
        .and()            
        .authorizeRequests() // 授权配置            
        .antMatchers("/authentication/require", "/login.html")
        .permitAll() // 登录跳转 URL 无需认证            
        .anyRequest()  // 所有请求            
        .authenticated() // 都需要认证            
        .and()
        .csrf().disable();
  }
    ```

    
  
  - 控制器`BrowserSecurityController`

```java
@RestController
public class BrowserSecurityController {    
    
    private RequestCache requestCache = new HttpSessionRequestCache();    
    private RedirectStrategy redirectStrategy = new DefaultRedirectStrategy();  
    
    @GetMapping("/authentication/require")    
    @ResponseStatus(HttpStatus.UNAUTHORIZED)    
    public String requireAuthentication(HttpServletRequest request, HttpServletResponse response) throws IOException {
        //获取请求对象信息
        SavedRequest savedRequest = requestCache.getRequest(request, response);        		  
        if (savedRequest != null) {
            
            String targetUrl = savedRequest.getRedirectUrl();
            //请求资源是以html结尾的资源  
            if (StringUtils.endsWithIgnoreCase(targetUrl, ".html")){                                	//则重定向至login.html	
                redirectStrategy.sendRedirect(request, response, "/login.html");  
            }
        }        
        return "访问的资源需要身份认证！";    
    }
}
```

​		`HttpSessionRequestCache`为Spring Security提供的用于缓存请求的对象，通过调用它的`getRequest`方法可以获取到本次请求的HTTP信息。

​		`DefaultRedirectStrategy`的`sendRedirect`为Spring Security提供的用于处理重定向的方法。



### 2.3、  处理用户登录成功逻辑

- 处理成功逻辑

  - 改变默认的处理成功逻辑

    1. 配置：重写`AuthenticationSuccessHandler`

       ```java
       @Component
       public class MyAuthenticationSucessHandler implements  AuthenticationSuccessHandler {
           
       	@Override    
           public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {  
               
             response.setContentType("application/json;charset=utf-8");        		    response.getWriter().write(mapper.writeValueAsString(authentication));    
           }
     }
       ```

       
  
    2. 生效：在`BrowserSecurityConfig`的`configure`中配置
  
       ```java
       @Configuration
       public class BrowserSecurityConfig extends WebSecurityConfigurerAdapter {
           //注入自己的配置对象
           @Autowired
           private MyAuthenticationSucessHandler authenticationSucessHandler;
       
           @Bean
           public PasswordEncoder passwordEncoder() {
               return new BCryptPasswordEncoder();
           }
       
           @Override
           protected void configure(HttpSecurity http) throws Exception {
               http.formLogin() // 表单登录
                       // http.httpBasic() // HTTP Basic
                       .loginPage("/authentication/require") // 登录跳转 URL
                       .loginProcessingUrl("/login") // 处理表单登录 URL
                       .successHandler(authenticationSucessHandler) // 处理登录成功
                       .and()
                       .authorizeRequests() // 授权配置
                       .antMatchers("/authentication/require", "/login.html")   
                       .permitAll() // 登录跳转 URL 无需认证
                       .anyRequest()  // 所有请求
                       .authenticated() // 都需要认证
                     .and().csrf().disable();
           }
     }
       ```

       
  
    3. 结果：用户登录成功之后返回：
  
    ```json
    {
      "authorities": [
        {
          "authority": "admin"
        }
      ],
      "details": {
        "remoteAddress": "0:0:0:0:0:0:0:1",
        "sessionId": "8D50BAF811891F4397E21B4B537F0544"
      },
      "authenticated": true,
      "principal": {
        "password": null,
        "username": "mrbird",
        "authorities": [
          {
            "authority": "admin"
          }
        ],
        "accountNonExpired": true,
        "accountNonLocked": true,
        "credentialsNonExpired": true,
        "enabled": true
      },
    "credentials": null,
      "name": "mrbird"
  }
    ```

  - 改变登录成功之后的跳转页面
  
    1. 修改配置文件
  
       ```java
       @Component
       public class MyAuthenticationSucessHandler implements AuthenticationSuccessHandler {    
           
           private RedirectStrategy redirectStrategy = new DefaultRedirectStrategy();    
           @Override    
         public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException {
               //配置跳转至index
             redirectStrategy.sendRedirect(request, response, "/index");    
           }
     }
       ```
  
    2. 定义一个成功请求的跳转controller
    
       ```java
       @RestController
       public class TestController {    
           @GetMapping("index")    
           public Object index(){        
               return SecurityContextHolder.getContext().getAuthentication();    
           }
       }
       ```



### 2.4、  处理用户登录失败逻辑

1. 实现`org.springframework.security.web.authentication.AuthenticationFailureHandler`的`onAuthenticationFailure`方法

   ```java
   @Component
   public class MyAuthenticationFailureHandler implements AuthenticationFailureHandler {    
       @Override    
       public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response,AuthenticationException exception) throws IOException {  
        
          
       }
   }
   ```

2. `AuthenticationException exception`

   AuthenticationException有很多实现类 分别对应不同异常

   ```java
   public abstract class AuthenticationException extends RuntimeException {
       public AuthenticationException(String msg, Throwable t) {
           super(msg, t);
       }
   
       public AuthenticationException(String msg) {
           super(msg);
       }
   }
   ```

3. 如果我们需要在登录失败的时候返回失败信息，可以这样处理：

   ```java
   @Component
   public class MyAuthenticationFailureHandler implements AuthenticationFailureHandler {
       
       @Autowired    
       private ObjectMapper mapper;    
       
       @Override    
       public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response,AuthenticationException exception) throws IOException {
           
           //定义状态码HttpStatus.INTERNAL_SERVER_ERROR.value()=500 
           response.setStatus(HttpStatus.INTERNAL_SERVER_ERROR.value());     
           //设置格式
           response.setContentType("application/json;charset=utf-8"); 
           //写入: mapper.writeValueAsString(exception.getMessage()):坏的凭证
           response.getWriter()
               .write(mapper.writeValueAsString(exception.getMessage()));    
       }
   }
   ```

4. 配置：使上面设置生效

   ```java
   @Configuration
   public class BrowserSecurityConfig extends WebSecurityConfigurerAdapter {
       
       @Autowired    
       private MyAuthenticationSucessHandler authenticationSucessHandler;                 
       //注入你的配置对象
       @Autowired    
       private MyAuthenticationFailureHandler authenticationFailureHandler;
       
       @Override    
       protected void configure(HttpSecurity http) throws Exception {          http.formLogin() // 表单登录                
           // http.httpBasic()                
           .loginPage("/authentication/require") // 登录跳转 URL                
           .loginProcessingUrl("/login") // 处理表单登录 URL                
           .successHandler(authenticationSucessHandler) // 处理登录成功                
           .failureHandler(authenticationFailureHandler) // 处理登录失败               
           .and()                
           .authorizeRequests() // 授权配置                
           .antMatchers("/authentication/require", "/login.html")
           .permitAll() // 登录跳转 URL 无需认证                
           .anyRequest()  // 所有请求                
           .authenticated() // 都需要认证                
           .and().csrf().disable();    
          }
   }
   ```




## 3、  自定义验证码

### 3.1、  根据随机数生成验证码

- 依赖

  ```xml
   <dependency>    
       <groupId>org.springframework.social</groupId>    
       <artifactId>spring-social-config</artifactId>
  </dependency>
  ```

- 验证码对象

  ```java
  public class ImageCode {  
      
      private BufferedImage image;    
      private String code;    
      private LocalDateTime expireTime; 
      
      public ImageCode(BufferedImage image, String code, int expireIn) {        
          this.image = image;        
          this.code = code;        
          this.expireTime = LocalDateTime.now().plusSeconds(expireIn);    
      }    
      
      public ImageCode(BufferedImage image, String code, LocalDateTime expireTime) {    		  this.image = image;        
          this.code = code;
          this.expireTime = expireTime;
      }    
      
      boolean isExpire() {        
          return LocalDateTime.now().isAfter(expireTime);    
      }    // get,set 略
  }
  ```



- 生成验证码

  ```java
  `@RestControllerpublic class ValidateController {    public final static String SESSION_KEY_IMAGE_CODE = "SESSION_KEY_IMAGE_CODE";    private SessionStrategy sessionStrategy = new HttpSessionSessionStrategy();    @GetMapping("/code/image")    public void createCode(HttpServletRequest request, HttpServletResponse response) throws IOException {        ImageCode imageCode = createImageCode();        sessionStrategy.setAttribute(new ServletWebRequest(request), SESSION_KEY, imageCode);        ImageIO.write(imageCode.getImage(), "jpeg", response.getOutputStream());    }}`
  ```





### 3.2、  图片将验证码图片显示到登录页面

- <img src="/code/image"/>

```html
<span style="display: inline">    
    <input type="text" name="imageCode" placeholder="验证码" style="width: 50%;"/>    
    <img src="/code/image"/>
</span>
```

- 放开请求验证码请求

  ```java
  .antMatchers(
      "/authentication/require",        
      "/login.html",        
      "/code/image").permitAll() // 无需认证的请求路径
  ```



### 3.3、  认证流程中加入验证码校验

- 未检验通过异常

```java
public class ValidateCodeException extends AuthenticationException {    
	private static final long serialVersionUID = 5022575393500654458L;                       ValidateCodeException(String message) {        
        super(message);
    }
}
```

配置

```java
@Component

//只会执行一次的Filter:OncePerRequestFilter
public class ValidateCodeFilter extends OncePerRequestFilter {
    @Autowired    
    private AuthenticationFailureHandler authenticationFailureHandler;    
    private SessionStrategy sessionStrategy = new HttpSessionSessionStrategy();
    
    @Override    
    protected void doFilterInternal(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse,FilterChain filterChain) throws ServletException, IOException { 
        //请求方法名称:/login  请求类型Post  
        if (StringUtils.equalsIgnoreCase("/login", httpServletRequest.getRequestURI())  && StringUtils.equalsIgnoreCase(httpServletRequest.getMethod(),"post")) {            
            
        //进入验证验证码逻辑  报的异常是ValidateCodeException  就是我们上面定义的异常
        try {                
            validateCode(new ServletWebRequest(httpServletRequest));           
        } catch (ValidateCodeException e) {
            //产生异常  验证码验证没有通过  直接返回
            authenticationFailureHandler
                .onAuthenticationFailure(httpServletRequest, httpServletResponse, e); 
            return;           
        }     
    }        
        //进入接下来的Filter
        filterChain.doFilter(httpServletRequest, httpServletResponse);   
	}   
    
    
    //从sessionStrategy拿到请求后台的 验证码
    private void validateCode(ServletWebRequest servletWebRequest) throws ServletRequestBindingException {
 
        //验证码
ImageCode codeInSession = (ImageCode) sessionStrategy.getAttribute(
    servletWebRequest, ValidateController.SESSION_KEY_IMAGE_CODE);
        
        //获取前端验证码
String codeInRequest = ServletRequestUtils.getStringParameter(servletWebRequest.getRequest(), "imageCode");

        //各种自定义异常
        if (StringUtils.isBlank(codeInRequest)) {
            throw new ValidateCodeException("验证码不能为空！");
        }
        if (codeInSession == null) {
            throw new ValidateCodeException("验证码不存在！");
        }
        if (codeInSession.isExpire()) {
            sessionStrategy.removeAttribute(servletWebRequest, ValidateController.SESSION_KEY_IMAGE_CODE);
            throw new ValidateCodeException("验证码已过期！");
        }
        if (!StringUtils.equalsIgnoreCase(codeInSession.getCode(), codeInRequest)) {
            throw new ValidateCodeException("验证码不正确！");
        }
        
        //验证通过 移除此次请求验证码
        sessionStrategy.removeAttribute(servletWebRequest, 									ValidateController.SESSION_KEY_IMAGE_CODE);
        
 	}	
}
```

- 添加验证码Filter至  `UsernamePasswordAuthenticationFilter`前面

```java
 http.
 	addFilterBefore(
 	validateCodeFilter, 
 	UsernamePasswordAuthenticationFilter.class) // 添加验证码校验过滤器                      
```



## 4、  记住我

### 4.1、  原理

1. 当用户勾选了记住我选项并登录成功
2. Spring Security会生成一个token标识，然后将该token标识持久化到数据库
3. Spring Security生成一个与该token相对应的cookie返回给浏览器
4. 当用户过段时间再次访问系统时，如果该cookie没有过期，Spring Security便会根据cookie包含的信息从数据库中获取相应的token信息，然后帮用户自动完成登录操作。



### 4.2、  Token持久化

- ​	配置`spring-boot-starter-jdbc`和`mysql-connector-java`

  ```xml
  <dependency>    
      <groupId>org.springframework.boot</groupId>    
      <artifactId>spring-boot-starter-jdbc</artifactId>
  </dependency>
  
  <dependency>    
      <groupId>mysql</groupId>    
      <artifactId>mysql-connector-java</artifactId>
  </dependency>
  ```

- 配置个token持久化对象:

  ```java
  @Autowired    
  private DataSource dataSource;    
  
  @Bean    
  public PersistentTokenRepository persistentTokenRepository() {        
      JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();       
      jdbcTokenRepository.setDataSource(dataSource);
      //启动时候不创表（删除token）
      jdbcTokenRepository.setCreateTableOnStartup(false);        
      return  jdbcTokenRepository;    
  }
  ```

- 在`BrowserSecurityConfig`的`configure`方法中开启

  ```java
   .and()
   .rememberMe()                
   .tokenRepository(persistentTokenRepository()) // 配置 token 持久化仓库           
   .tokenValiditySeconds(3600) // remember 过期时间，单为秒                
   .userDetailsService(userDetailService) // 处理自动登录逻辑
  ```



## 5、  短信验证码登陆

### 5.1、  短信验证码生成

- 短信验证码对象

  ```java
  public class SmsCode 
  {    
      private String code;    
      private LocalDateTime expireTime;    
      public SmsCode(String code, int expireIn) {        
          this.code = code;        
          this.expireTime = LocalDateTime.now().plusSeconds(expireIn);    
      }    
      
      public SmsCode(String code, LocalDateTime expireTime) {        
          this.code = code;        
          this.expireTime = expireTime;    
      }    
      
      boolean isExpire() {        
          return LocalDateTime.now().isAfter(expireTime);    
      }    // get,set略
  }
  ```



- 发送短信

  ```java
  @RestController
  public class ValidateController {    
      public final static String SESSION_KEY_SMS_CODE = "SESSION_KEY_SMS_CODE";    
      private SessionStrategy sessionStrategy = new HttpSessionSessionStrategy(); 
      
      @GetMapping("/code/sms")    
      public void createSmsCode(HttpServletRequest request, HttpServletResponse response, String mobile) throws IOException {        
          
          SmsCode smsCode = createSMSCode();    
          //在session 存请求信息+电话号码+验证码
          sessionStrategy.setAttribute(
              new ServletWebRequest(request), SESSION_KEY_SMS_CODE + mobile, smsCode
          );        
          // 输出验证码到控制台代替短信发送服务      
          System.out.println("您的登录验证码为：" + smsCode.getCode() + "，有效时间为60秒");    }    
     
      private SmsCode createSMSCode() {        
          String code = RandomStringUtils.randomNumeric(6);        
          return new SmsCode(code, 60);    
      }
  }
  ```



### 5.2、  验证码登录

- 用户名密码登陆逻辑

![QQæªå¾20180730220603.png](https://mrbird.cc/img/QQ%E6%88%AA%E5%9B%BE20180730220603.png)

1. Spring Security使用`UsernamePasswordAuthenticationFilter`过滤器来拦截用户名密码认证请求。

2. 将用户名和密码封装成一个`UsernamePasswordToken`对象交给`AuthenticationManager`处理。

3. `AuthenticationManager`将挑出一个支持处理该类型Token的`AuthenticationProvider`（这里为`DaoAuthenticationProvider`，`AuthenticationProvider`的其中一个实现类）来进行认证。

4. 认证过程中`DaoAuthenticationProvider`将调用`UserDetailService`的`loadUserByUsername`方法来处理认证。

5. 如果认证通过（即`UsernamePasswordToken`中的用户名和密码相符）则返回一个`UserDetails`类型对象，并将认证信息保存到Session中，认证后我们便可以通过`Authentication`对象获取到认证的信息了。

   

- 短信登陆仿照用户名密码登陆

  ![QQæªå¾20180730224103.png](https://mrbird.cc/img/QQ%E6%88%AA%E5%9B%BE20180730224103.png)



1. 自定义了一个名为`SmsAuthenticationFitler`的过滤器来拦截短信验证码登录请求，
2. 并将手机号码封装到一个叫`SmsAuthenticationToken`的对象中。
3. 在Spring Security中，认证处理都需要通过`AuthenticationManager`来代理，所以这里我们依旧将`SmsAuthenticationToken`交由`AuthenticationManager`处理。
4. 接着我们需要定义一个支持处理`SmsAuthenticationToken`对象的`SmsAuthenticationProvider`，
5. `SmsAuthenticationProvider`调用`UserDetailService`的`loadUserByUsername`方法来处理认证。
6. 与用户名密码认证不一样的是，这里是通过`SmsAuthenticationToken`中的手机号去数据库中查询是否有与之对应的用户，如果有，则将该用户信息封装到`UserDetails`对象中返回并将认证后的信息保存到`Authentication`对象中。





​		为了实现这个流程，我们需要定义`SmsAuthenticationFitler`、`SmsAuthenticationToken`和`SmsAuthenticationProvider`，并将这些组建组合起来添加到Spring Security中

- `SmsAuthenticationToken`  仿照`UsernamePasswordAuthenticationToken`

  ```java
  public class SmsAuthenticationToken extends AbstractAuthenticationToken {    
      
      private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;    
      
      private final Object principal;    
      
      //认证前 存放用户手机号
      public SmsAuthenticationToken(String mobile) {        
          super(null);        
          this.principal = mobile;        
          setAuthenticated(false);    
      }    
      
      //认证后  存放用户权限信息
      public SmsAuthenticationToken(Object principal, Collection<? extends GrantedAuthority> authorities) {        
          super(authorities);        
          this.principal = principal;        
          super.setAuthenticated(true); // must use super, as we override    
      }    
      
      @Override    
      public Object getCredentials() {        
          return null;    
      }   
      
      public Object getPrincipal() {        
          return this.principal;    
      }    
      
      public void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException {        
          
          if (isAuthenticated) { 
              throw new IllegalArgumentException("Cannot set this token to trusted - use constructor which takes a GrantedAuthority list instead");  
          }        
          
          super.setAuthenticated(false);    
      }    
      
      @Override    
      public void eraseCredentials() {        
          super.eraseCredentials();    
      }
  }
  ```



- `SmsAuthenticationFilter` 参照 `UsernamePasswordAuthenticationFilter`

```java
public class SmsAuthenticationFilter extends AbstractAuthenticationProcessingFilter {    
    
    public static final String MOBILE_KEY = "mobile";    
    private String mobileParameter = MOBILE_KEY;    
    private boolean postOnly = true;    
    
    public SmsAuthenticationFilter() {        
        super(new AntPathRequestMatcher("/login/mobile", "POST"));    
    }    
    
    public Authentication attemptAuthentication(HttpServletRequest request,                   HttpServletResponse response) throws AuthenticationException {        
        
        //使用Post进行认证 不支持其他方法
        if (postOnly && !request.getMethod().equals("POST")) {            
            
            throw new AuthenticationServiceException(
                "Authentication method not supported: " + request.getMethod());        
        }        

        String mobile = obtainMobile(request);        
        
        if (mobile == null) {           
            mobile = "";        
        }        
        
        mobile = mobile.trim();  
        //将收集号码封装至SmsAuthenticationToken对象   
        SmsAuthenticationToken authRequest = new SmsAuthenticationToken(mobile);   
        
        //暂不明确
        setDetails(request, authRequest);  
        
        //SmsAuthenticationFilter将SmsAuthenticationToken交给AuthenticationManager处理。
        return this.getAuthenticationManager().authenticate(authRequest);    
    }    
    
    protected String obtainMobile(HttpServletRequest request) {        
        return request.getParameter(mobileParameter);    
    }    
    
    
    //暂不明确
    protected void setDetails(HttpServletRequest request,
                              SmsAuthenticationToken authRequest) {
        
        authRequest.setDetails(authenticationDetailsSource.buildDetails(request));    
    
    }    
    
    public void setMobileParameter(String mobileParameter) {
        
        Assert.hasText(mobileParameter, "mobile parameter must not be empty or null");  
        this.mobileParameter = mobileParameter;    
    }    
    
    public void setPostOnly(boolean postOnly) {        
        this.postOnly = postOnly;    
    }    
    
    public final String getMobileParameter() {        
        return mobileParameter;    
    }
}
```



- SmsAuthenticationProvider

```java
public class SmsAuthenticationProvider implements AuthenticationProvider { 
    
    private UserDetailService userDetailService;    
    
    //编写具体的身份认证逻辑
    @Override    
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        
        
        //取token
        SmsAuthenticationToken authenticationToken = (SmsAuthenticationToken) authentication;
        
        //使用手机号码调用loadUserByUsername，
        //该方法在用户名密码类型的认证中
        //主要逻辑是通过用户名查询用户信息，如果存在该用户并且密码一致则认证成功
        //在短信验证码认证的过程中，该方法需要通过手机号去查询用户，如果存在该用户则认证通过
        //认证通过后 通过带权限的token构造函数构造一个认证通过的Token，包含了用户信息和用户权限。
        //此方法是短信验证码正确之后的操作
        UserDetails userDetails = userDetailService.loadUserByUsername(
            //从token中获取号码
            (String) authenticationToken.getPrincipal()
        );  
        
        
        if (userDetails == null)           
            throw new InternalAuthenticationServiceException("未找到与该手机号对应的用户");        
        SmsAuthenticationToken authenticationResult = 
            new SmsAuthenticationToken(userDetails, userDetails.getAuthorities());        
        authenticationResult.setDetails(authenticationToken.getDetails());        
        return authenticationResult;    
    } 
    
    
    //指定了支持处理的Token类型为SmsAuthenticationToken
    @Override    
    boolean supports(Class<?> aClass) {        
        return SmsAuthenticationToken.class.isAssignableFrom(aClass);    
    }    
    
    public UserDetailService getUserDetailService() {        
        return userDetailService;    
    }    
    
    public void setUserDetailService(UserDetailService userDetailService) {        
        this.userDetailService = userDetailService;    
    }
}
```



- SmsCodeFilter  验证短信验证码是否正确

```java
@Componentpublic 
class SmsCodeFilter extends OncePerRequestFilter {
   
    @Autowired    
    private AuthenticationFailureHandler authenticationFailureHandler;    
    
    private SessionStrategy sessionStrategy = new HttpSessionSessionStrategy();    
    
    @Override    
    protected void doFilterInternal(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse,FilterChain filterChain) throws ServletException, IOException {        
        
        //指定链接/login/mobile 指定方法Post
        if (
    	StringUtils.equalsIgnoreCase("/login/mobile", httpServletRequest.getRequestURI())         && StringUtils.equalsIgnoreCase(httpServletRequest.getMethod(), "post")) {
            
            //判断验证码是否正确
            try {                
                validateCode(new ServletWebRequest(httpServletRequest));            
            } catch (ValidateCodeException e) {
               //认证失败 直接返回
                authenticationFailureHandler.onAuthenticationFailure(
                    httpServletRequest, 
                    httpServletResponse,
                    e);                
                return;            
            }        
        }   
        
        //下一步认证
        filterChain.doFilter(httpServletRequest, httpServletResponse);    
    }    
    
    
    //校验验证码
    private void validateSmsCode(ServletWebRequest servletWebRequest) throws ServletRequestBindingException {        
        
        String smsCodeInRequest = 
            ServletRequestUtils.getStringParameter(servletWebRequest.getRequest(), "smsCode");        
        String mobile = 
            ServletRequestUtils.getStringParameter(servletWebRequest.getRequest(), "mobile");        
        
        ValidateCode codeInSession = (ValidateCode) sessionStrategy.getAttribute(servletWebRequest, FebsConstant.SESSION_KEY_SMS_CODE + mobile);        
        
        if (StringUtils.isBlank(smsCodeInRequest)) {            
            throw new ValidateCodeException("验证码不能为空！");        
        }
        
        if (codeInSession == null) {            
            throw new ValidateCodeException("验证码不存在，请重新发送！");        
        }        
        
        if (codeInSession.isExpire()) {           
            sessionStrategy.removeAttribute(servletWebRequest, FebsConstant.SESSION_KEY_SMS_CODE + mobile);            
            throw new ValidateCodeException("验证码已过期，请重新发送！");        
        }        
        if (!StringUtils.equalsIgnoreCase(codeInSession.getCode(), smsCodeInRequest)) {   
            throw new ValidateCodeException("验证码不正确！");       
        }       
        
        sessionStrategy.removeAttribute(
            servletWebRequest, FebsConstant.SESSION_KEY_SMS_CODE + mobile);    
    }
}
```



- 配置所有组件 组成短信登陆流程

```java
@Component
public class SmsAuthenticationConfig extends SecurityConfigurerAdapter<DefaultSecurityFilterChain, HttpSecurity> { 
    
    @Autowired    
    private AuthenticationSuccessHandler authenticationSuccessHandler;    
    
    @Autowired    
    private AuthenticationFailureHandler authenticationFailureHandler;    
    
    @Autowired    
    
    private UserDetailService userDetailService;    
    
    @Override
    public void configure(HttpSecurity http) throws Exception {

        //配置Filter
        SmsAuthenticationFilter smsAuthenticationFilter = new SmsAuthenticationFilter();
        //设置Manager
        smsAuthenticationFilter.setAuthenticationManager(
    		http.getSharedObject(AuthenticationManager.class)
        ); 
 
 //配置成功失败
   smsAuthenticationFilter.setAuthenticationSuccessHandler(authenticationSuccessHandler);    smsAuthenticationFilter.setAuthenticationFailureHandler(authenticationFailureHandler);

        
        //设置Provider  将自己的userDetailService 注入
        SmsAuthenticationProvider smsAuthenticationProvider = new SmsAuthenticationProvider();
        smsAuthenticationProvider.setUserDetailService(userDetailService);

        
        //指定provider为smsAuthenticationProvider
        http
            .authenticationProvider(smsAuthenticationProvider)
       //将SmsAuthenticationFilter过滤器添加至UsernamePasswordAuthenticationFilter后面
            .addFilterAfter(
            smsAuthenticationFilter, 
            UsernamePasswordAuthenticationFilter.class
        );
    }
}
```



- 配置至 `BrowserSecurityConfig`

  ```java
   .antMatchers(
       "/authentication/require",                            
       "/login.html",                            
       "/code/image","/code/sms")
                  
  .and()                                 
  .apply(smsAuthenticationConfig); // 将短信验证码认证配置加到 Spring Security 中
  ```

  

## 6、Session管理

- 设置session过期时间

  ```yml
  server:  
  	session:    
  		timeout: 3600
  ```

- session过期跳转页面`BrowserSecurityConfig`

  ```java
  .antMatchers("/session/invalid").permitAll() // 无需认证的请求路径
      
  .sessionManagement() // 添加 Session管理器      
  .invalidSessionUrl("/session/invalid") // Session失效后跳转到这个链接
  ```

- 接口

  ```java
  @GetMapping("session/invalid")
  @ResponseStatus(HttpStatus.UNAUTHORIZED)
  public String sessionInvalid(){    
      return "session已失效，请重新认证";
  }
  ```



### 6.1、  并发控制

Session并发控制可以控制一个账号同一时刻最多能登录多少个

```java
@Autowired
private MySessionExpiredStrategy sessionExpiredStrategy;

	.and()
    .sessionManagement() // 添加 Session管理器
    .invalidSessionUrl("/session/invalid") // Session失效后跳转到这个链接
    .maximumSessions(1)
    .expiredSessionStrategy(sessionExpiredStrategy)
```

`expiredSessionStrategy`配置了Session在并发下失效后的处理策略，

这里为我们自定义的策略`MySessionExpiredStrategy`。



- 踢出前登陆用户---自定义策略：`MySessionExpiredStrategy` 

  ```java
  @Component
  public class MySessionExpiredStrategy implements SessionInformationExpiredStrategy {    
      @Override    
      public void onExpiredSessionDetected(SessionInformationExpiredEvent event) throws IOException, ServletException {
          
          HttpServletResponse response = event.getResponse();        
          response.setStatus(HttpStatus.UNAUTHORIZED.value());        
          response.setContentType("application/json;charset=utf-8");        
          response.getWriter().write("您的账号已经在别的地方登录，当前登录已失效。如果密码遭到泄露，请立即修改密码！");    
          
      }
  }
  ```



- 用户登陆后限制其他用户登陆----`BrowserSecurityConfig`

  ```java
  .maxSessionsPreventsLogin(true)
  ```



- 支持

  ​		Session并发控制只对Spring Security默认的登录方式——账号密码登录有效，而像短信验证码登录，社交账号登录并不生效，解决方案可以参考我的开源项目https://github.com/wuyouzhuguli/FEBS-Security



### 6.2、  Session集群处理

存放session至第三方容器（如Redis集群），集群就可以通过第三方容器来共享Session。

- Redis和Spring Session依赖：

  ```xml
  <dependency>    
      <groupId>org.springframework.session</groupId>    
      <artifactId>spring-session</artifactId>
  </dependency>
  
  <dependency>    
      <groupId>org.springframework.boot</groupId>    
      <artifactId>spring-boot-starter-data-redis</artifactId>
  </dependency>
  ```

- 配置Session存储方式为Redis：

  ```yml
  spring:
    session:
      store-type: redis
  ```





## 7、退出登录

Spring Security默认的退出登录URL为`/logout`，退出登录后，Spring Security会做如下处理：

1. 使当前的Sesion失效；
2. 清除与当前用户关联的RememberMe记录；
3. 清空当前的SecurityContext；
4. 重定向到登录页。



### 7.1、  自定义退出登录行为

- 配置

  ```java
  .and()    
      .logout()    
      .logoutUrl("/signout")    
      .logoutSuccessUrl("/signout/success")    
      .deleteCookies("JSESSIONID")
  ```

退出登录的URL为`/signout`，

退出成功后跳转的URL为`/signout/success`

退出成功后删除名称为`JSESSIONID`的cookie。



- Controller

  ```java
  @GetMapping("/signout/success")
  public String signout() {    
      return "退出成功，请重新登录";
  }
  ```



- 或者自己实现退出成功逻辑

  - `logoutSuccessHandler`

    ```java
    @Autowired
    private MyLogOutSuccessHandler logOutSuccessHandler;
    
    .and()    
        .logout()    
        .logoutUrl("/signout")    
        //.logoutSuccessUrl("/signout/success")    
        //注入自定义逻辑
        .logoutSuccessHandler(logOutSuccessHandler)    
        .deleteCookies("JSESSIONID").and()
    ```

  - `MyLogOutSuccessHandler` 

    ```java
    @Component
    public class MyLogOutSuccessHandler implements LogoutSuccessHandler {    
        @Override    
        public void onLogoutSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException { 
            
            httpServletResponse.setStatus(HttpStatus.UNAUTHORIZED.value());        
            httpServletResponse.setContentType("application/json;charset=utf-8");     
            httpServletResponse.getWriter().write("退出成功，请重新登录");
            
        }
    }
    ```



## 8、权限控制

- admin权限才能访问

  ```java
  @GetMapping("/auth/admin")
  @PreAuthorize("hasAuthority('admin')")
  public String authenticationTest() {    
      return "您拥有admin权限，可以查看";
  }
  ```



- 自定义无权限访问

  - 自定义处理类

  ```java
  @Component
  public class MyAuthenticationAccessDeniedHandler implements AccessDeniedHandler {    
      @Override    
      public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException {        
          response.setStatus(HttpStatus.INTERNAL_SERVER_ERROR.value());        
          response.setContentType("application/json;charset=utf-8");        
          response.getWriter().write("很抱歉，您没有该访问权限");    
      }
  }
  ```
  - 配置

  ```java
  @Autowired
  private MyAuthenticationAccessDeniedHandler authenticationAccessDeniedHandler;
  
   @Override
  protected void configure(HttpSecurity http) throws Exception {
      http.exceptionHandling()
              .accessDeniedHandler(authenticationAccessDeniedHandler)
          .and()
      ......
  }
  ```

