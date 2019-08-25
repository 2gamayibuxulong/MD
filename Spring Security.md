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
  @Configurationpublic 
  class UserDetailService implements UserDetailsService {    
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
              username, user.getPassword(), 
              user.isEnabled(),                
              user.isAccountNonExpired(), 
              user.isCredentialsNonExpired(),               		                       user.isAccountNonLocked(),             
              AuthorityUtils.
              commaSeparatedStringToAuthorityList("admin")//admin角色
          );    
      }
  }
  ```

- 替换登录页

  在src/main/resources/resources目录下定义一个login.html

  ```java
  @Override
  protected void configure(HttpSecurity http) throws Exception {    http.formLogin() // 表单登录            
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
        .antMatchers("/authentication/require", "/login.html").permitAll() // 登录跳转 URL 无需认证            
        .anyRequest()  // 所有请求            
        .authenticated() // 都需要认证            
        .and().csrf().disable();
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
            if (StringUtils.endsWithIgnoreCase(targetUrl, ".html")){                               	//则重定向至login.html	
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
           public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response,                                        Authentication authentication) throws IOException, ServletException {        		response.setContentType("application/json;charset=utf-8");        		  response.getWriter().write(mapper.writeValueAsString(authentication));    }
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
       @Componentpublic class MyAuthenticationSucessHandler implements AuthenticationSuccessHandler {    
           private RedirectStrategy redirectStrategy = new DefaultRedirectStrategy();    
           @Override    
           public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response,                                        Authentication authentication) throws IOException {
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
       public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response,                                        AuthenticationException exception) throws IOException {  
           
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
       public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response,                                        AuthenticationException exception) throws IOException {
           //定义状态码HttpStatus.INTERNAL_SERVER_ERROR.value()=500 
           response.setStatus(HttpStatus.INTERNAL_SERVER_ERROR.value());     
           //设置格式
           response.setContentType("application/json;charset=utf-8"); 
           //写入
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

   