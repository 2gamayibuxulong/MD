# Spring Security OAuth2

## 1、四种授权模式

在浏览器上使用QQ账号登录虎牙直播

1. **Third-party application** 第三方应用程序，比如这里的虎牙直播；
2. **HTTP service** HTTP服务提供商，比如这里的QQ（腾讯）;
3. **Resource Owner** 资源所有者，就是QQ的所有人，你；
4. **User Agent** 用户代理，这里指浏览器；
5. **Authorization server** 认证服务器，这里指QQ提供的第三方登录服务；
6. **Resource server** 资源服务器，这里指虎牙直播提供的服务，比如高清直播，弹幕发送等（需要认证后才能使用）。



​		认证服务器和资源服务器可以在同一台服务器上，比如前后端分离的服务后台，它即供认证服务（认证服务器，提供令牌），客户端通过令牌来从后台获取服务（资源服务器）；它们也可以不在同一台服务器上，比如上面第三方登录的例子。



### 1.1、  授权码模式

![QQæªå¾20190624150726.png](https://mrbird.cc/img/QQ%E6%88%AA%E5%9B%BE20190624150726.png)

- 步骤

  A. 客户端将用户导向认证服务器；

  B. 用户决定是否给客户端授权；

  C. 同意授权后，认证服务器将用户导向客户端提供的URL，并附上授权码；

  D. 客户端通过重定向URL和授权码到认证服务器换取令牌；

  E. 校验无误后发放令牌。

   

- A步骤细节

  A步骤，客户端申请认证的URI，包含以下参数：

  1. response_type：表示授权类型，必选项，此处的值固定为”code”，标识授权码模式
  2. client_id：表示客户端的ID，必选项
  3. redirect_uri：表示重定向URI，可选项
  4. scope：表示申请的权限范围，可选项
  5. state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值。



- D步骤细节

  D步骤中，客户端向认证服务器申请令牌的HTTP请求，包含以下参数：

  1. grant_type：表示使用的授权模式，必选项，此处的值固定为”authorization_code”。
  2. code：表示上一步获得的授权码，必选项。
  3. redirect_uri：表示重定向URI，必选项，且必须与A步骤中的该参数值保持一致。
  4. client_id：表示客户端ID，必选项。



### 1.2、  密码模式

在密码模式中，用户像客户端提供用户名和密码，客户端通过用户名和密码到认证服务器获取令牌。

![QQæªå¾20190624150632.png](https://mrbird.cc/img/QQ%E6%88%AA%E5%9B%BE20190624150632.png)

- 步骤

  A. 用户向客户端提供用户名和密码；

  B. 客户端向认证服务器换取令牌；

  C. 发放令牌。



- B步骤细节

  B步骤中，客户端发出的HTTP请求，包含以下参数：

  1. grant_type：表示授权类型，此处的值固定为”password”，必选项。
  2. username：表示用户名，必选项。
  3. password：表示用户的密码，必选项。
  4. scope：表示权限范围，可选项。



### 1.3、  其余：https://oauth.net/2/



## 2、  Spring Security OAuth2

Spring 框架对OAuth2协议进行了实现

Spring Security OAuth2主要包含认证服务器和资源服务器这两大块的实现：

![QQæªå¾20190624155418.png](https://mrbird.cc/img/QQ%E6%88%AA%E5%9B%BE20190624155418.png)

### 2.1、  认证服务器

- 认证

  - 新建User
  - 实现自己的UserDetailsService 
  - 指定认证逻辑：loadUserByUsername

  

- 配置认证服务器

  ```java
  @Configuration
  @EnableAuthorizationServer
  public class AuthorizationServerConfig extends WebSecurityConfigurerAdapter {    
      
      @Bean    
      public PasswordEncoder passwordEncoder() {        
          return new BCryptPasswordEncoder();    
      }
  }
  ```



- 授权码模式获取令牌

  - 请求授权码

    http://localhost:8080/oauth/authorize?response_type=code&client_id=test&redirect_uri=http://mrbird.cc&scope=all&state=hello

    1. response_type:code 授权码模式
    2. client_id: 配置的
    3. redirect_uri：重定向获取授权码
    4. scope:all 表示所有权限

  - 同意授权  scope:all  Approve

  - 获得授权码

    https://mrbird.cc/?code=uYj0Wy&state=hello

    1. code:授权码  uYj0Wy

  - 获取令牌

    1. 请求授权服务器地址 [localhost:8080/oauth/token](localhost:8080/oauth/token)

    2. 参数

       - Param:

         1. grant_type: authorization_code 授权码模式

         2. code:请求授权码时候取得的授权码

         3. client_id:请求授权码时候指定的client_id

         4. redirect_uri:请求授权码时候指定的redirect_uri

         5. scope:all

            

       - Headers:
         
       1. Authentication:`Basic`加上`client_id:client_secret`经过base64加密后的值
    
  3. 发送请求获取令牌
    
       ```json
       {    
           "access_token": "950018df-0199-4936-aa80-a3a66183f634",    
           "refresh_token": "cc22e8b2-e069-459d-8c24-cfda0bc72128",    
           "expires_in": 42827,    
           "scope": "all"
       }
       ```

  

- 密码模式获取令牌

  - 获取令牌

    1. 指定授权服务器地址

       [localhost:8080/oauth/token](localhost:8080/oauth/token)

    2. 指定参数

       - params:

         grant_type：passowrd 密码模式

         username:用户名

         password:密码

         scope:all

       - Headers:

         同授权码模式

    3. 获得令牌

       ```json
       {    
           "access_token": "d612cf50-6499-4a0c-9cd4-9c756839aa12",    
           "token_type": "bearer",    
           "refresh_token": "fdc6c77f-b910-46dc-a349-835dc0587919",    
           "expires_in": 43090,    
           "scope": "all"
       }
       ```



### 2.2、  资源服务器

​			拿到令牌之后仍然无法访问静态资源

- 配置

  ```java
  @Configuration
  @EnableResourceServer
  public class ResourceServerConfig  {
      
  }
  ```



- tips

  ​		在同时定义了认证服务器和资源服务器后，再去使用授权码模式获取令牌可能会遇到 `Full authentication is required to access this resource` 的问题，这时候只要确保认证服务器先于资源服务器配置即可，比如在认证服务器的配置类上使用`@Order(1)`标注，在资源服务器的配置类上使用`@Order(2)`标注。





## 3、  Spring Security OAuth2自定义Token获取方式

### 3.1、  自定义用户名密码方式获取令牌

- 原理

  ![QQæªå¾20190624223930.png](https://mrbird.cc/img/QQ%E6%88%AA%E5%9B%BE20190624223930.png)

- 登陆成功返回令牌

  ```java
  @Component
  public class MyAuthenticationSucessHandler implements AuthenticationSuccessHandler {
  
      private Logger log = LoggerFactory.getLogger(this.getClass());
  
      @Autowired
      private ClientDetailsService clientDetailsService;
      
      
      @Autowired
      private AuthorizationServerTokenServices authorizationServerTokenServices;
  
      @Override
      public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException {
          
          // 1. 从请求头中获取 ClientId
          String header = request.getHeader("Authorization");
          if (header == null || !header.startsWith("Basic ")) {
              throw new UnapprovedClientAuthenticationException("请求头中无client信息");
          }
  
          //解密Headers里面经过Base64加密的东西
          String[] tokens = this.extractAndDecodeHeader(header, request);
          String clientId = tokens[0];
          String clientSecret = tokens[1];
  
          TokenRequest tokenRequest = null;
  
          // 2. 通过 ClientDetailsService 获取 ClientDetails
          ClientDetails clientDetails = clientDetailsService.loadClientByClientId(clientId);
  
          // 3. 校验 ClientId和 ClientSecret的正确性
          if (clientDetails == null) {
              throw new UnapprovedClientAuthenticationException("clientId:" + clientId + "对应的信息不存在");
          } else if (!StringUtils.equals(clientDetails.getClientSecret(), clientSecret)) {
              throw new UnapprovedClientAuthenticationException("clientSecret不正确");
          } else {
              // 4. 通过 TokenRequest构造器生成 TokenRequest
              tokenRequest = new TokenRequest(new HashMap<>(), clientId, clientDetails.getScope(), "custom");
          }
  
          // 5. 通过 TokenRequest的 createOAuth2Request方法获取 OAuth2Request
          OAuth2Request oAuth2Request = tokenRequest.createOAuth2Request(clientDetails);
          // 6. 通过 Authentication和 OAuth2Request构造出 OAuth2Authentication
          OAuth2Authentication auth2Authentication = new OAuth2Authentication(oAuth2Request, authentication);
  
          // 7. 通过 AuthorizationServerTokenServices 生成 OAuth2AccessToken
          OAuth2AccessToken token = authorizationServerTokenServices.createAccessToken(auth2Authentication);
  
          // 8. 返回 Token
          log.info("登录成功");
          response.setContentType("application/json;charset=UTF-8");
          response.getWriter().write(new ObjectMapper().writeValueAsString(token));
      }
  
      private String[] extractAndDecodeHeader(String header, HttpServletRequest request) {
          byte[] base64Token = header.substring(6).getBytes(StandardCharsets.UTF_8);
  
          byte[] decoded;
          try {
              decoded = Base64.getDecoder().decode(base64Token);
          } catch (IllegalArgumentException var7) {
              throw new BadCredentialsException("Failed to decode basic authentication token");
          }
  
          String token = new String(decoded, StandardCharsets.UTF_8);
          int delim = token.indexOf(":");
          if (delim == -1) {
              throw new BadCredentialsException("Invalid basic authentication token");
          } else {
              return new String[]{token.substring(0, delim), token.substring(delim + 1)};
          }
      }
  }
  
  ```

- 获得令牌

  - params

    1. username:123

    2. password:123456

       

  - Headers

    1. Authorization: Basic dGVzdDp0ZXN0MTIzNA==

    

  - 获得令牌

    ```json
    {    
        "access_token": "88a3dd6c-ab27-41af-95ee-5cd406fe5ab1",   
        "token_type": "bearer",    
        "refresh_token": "b316177d-68e9-4fc9-9f4a-804a7367ebc9",    
        "expires_in": 43199
    }
    ```

    

  - 带令牌访问资源服务器

    1. 参数

       Authorization:bearer   88a3dd6c-ab27-41af-95ee-5cd406fe5ab1



### 3.2、  短信验证码获取令牌

​		之前的例子验证码存储在Session中，现在使用令牌的方式和系统交互后Session已经不适用了，我们可以使用第三方存储来保存我们的验证码（无论是短信验证码还是图形验证码都是一个道理），比如Redis等。

- 获取验证码

  ```java
  @Service
  public class RedisCodeService {    
      private final static String SMS_CODE_PREFIX = "SMS_CODE:";    
      private final static Integer TIME_OUT = 300;    
      
      @Autowired    
      private StringRedisTemplate redisTemplate;       
      
      //保存验证码
      public void save(SmsCode smsCode, ServletWebRequest request, String mobile) throws Exception {        
          
          redisTemplate.opsForValue().set(
              key(request, mobile), 
              smsCode.getCode(), TIME_OUT, 
              TimeUnit.SECONDS
          );
          
      }    
      //获取验证码    
      public String get(ServletWebRequest request, String mobile) throws Exception {     
          return redisTemplate.opsForValue().get(key(request, mobile));    
      }    
      //移除验证码
      public void remove(ServletWebRequest request, String mobile) throws Exception {
          redisTemplate.delete(key(request, mobile));    
      }    
  
      private String key(ServletWebRequest request, String mobile) throws Exception {        
          String deviceId = request.getHeader("deviceId");        
          if (StringUtils.isBlank(deviceId)) {            
              throw new Exception("请在请求头中设置deviceId");        
          }        
          return SMS_CODE_PREFIX + deviceId + ":" + mobile;    
      }
  }
  ```



- 设置请求头信息deviceId

  

- 根据验证码 获取令牌

  - params

    1. mobile:号码
    2. smscode:验证码

  - Headers

    1. Authorization: Basic dGVzdDp0ZXN0MTIzNA==（经过base64加密的`client_id:client_secret`：）
    2. deviceId

  - 获得令牌

    ```json
    {    
        "access_token": "7fe22e67-1a11-4708-8707-0100555a9d1a",    
        "token_type": "bearer",    
        "refresh_token": "7c7a814f-2ace-4171-9748-56cb1994b04b",    
        "expires_in": 41982
    }
    ```



## 4、  Spring Security OAuth2自定义令牌配置

​		前面几节中，我们获取到的令牌都是基于Spring Security OAuth2默认配置生成的，Spring Security允许我们自定义令牌配置，比如不同的client_id对应不同的令牌，令牌的有效时间，令牌的存储策略等；我们也可以使用JWT来替换默认的令牌。



### 4.1、  自定义令牌配置

1. 认证服务器`AuthorizationServerConfig`继承`AuthorizationServerConfigurerAdapter`，并重写它的`configure(ClientDetailsServiceConfigurer clients)`方法，

   configure方法内需指定AuthenticationManager和UserDetailService。

   

   ```java
   @Configuration
   @EnableAuthorizationServer
   public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {    
       ......
           
       //需指定AuthenticationManager    
       @Autowired    
       private AuthenticationManager authenticationManager;    
       @Autowired    
       private UserDetailService userDetailService;    
       
       @Override    
       public void configure(AuthorizationServerEndpointsConfigurer endpoints) {       
           endpoints.authenticationManager(authenticationManager)                
               .userDetailsService(userDetailService);    
       }    
       
       @Override    
       public void configure(ClientDetailsServiceConfigurer clients) throws Exception{     
           
           clients.inMemory() 
               
               .withClient("test1")//client_id为 test1 
               //client_secret 为test1111
               .secret(new BCryptPasswordEncoder().encode("test1111"))                
               .accessTokenValiditySeconds(3600)//test1令牌有效时间3600秒               
               .refreshTokenValiditySeconds(864000)//获取新令牌时间                
               .scopes("all", "a", "b", "c")//指定范围                
               .authorizedGrantTypes("password")//指定模式  
               
               
               .and()                
               .withClient("test2")                
               .secret("test2222")                
               .accessTokenValiditySeconds(7200);    
       }
   }
   ```

   - 定义两个client_id，及客户端可以通过不同的client_id来获取不同的令牌；
   - client_id为test1的令牌有效时间为3600秒，client_id为test2的令牌有效时间为7200秒；
   - client_id为test1的refresh_token有效时间为864000秒，即10天，也就是说在这10天内都可以通过refresh_token来换取新的令牌；
   - 在获取client_id为test1的令牌的时候，scope只能指定为all，a，b或c中的某个值，否则将获取失败
   - 只能通过密码模式(password)来获取client_id为test1的令牌，而test2则无限制。

   ​                 

2. 创建一个新的配置类`SecurityConfig`，在里面注册我们需要的`AuthenticationManager`Bean：

   ```java
   @Configuration
   public class SecurityConfig extends WebSecurityConfigurerAdapter {
       
       @Bean(name = BeanIds.AUTHENTICATION_MANAGER)    
       @Override    
       public AuthenticationManager authenticationManagerBean() throws Exception {        return super.authenticationManagerBean();
       }
       
   }
   ```

    

   

3. 存储令牌至第三方存储Redis

   ```java
   @Configuration
   public class TokenStoreConfig {    
       @Autowired    
       private RedisConnectionFactory redisConnectionFactory;
       
       @Bean    
       public TokenStore redisTokenStore (){        
           return new RedisTokenStore(redisConnectionFactory);    
       }
   }
   ```

   

4. 配置令牌存储方式

   ```java
   @Configuration
   @EnableAuthorizationServer
   public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter { 
       
       @Autowired    
       private TokenStore redisTokenStore;    
       
       //重写configure方法
       @Override   
       public void configure(AuthorizationServerEndpointsConfigurer endpoints) {        
           endpoints
               .authenticationManager(authenticationManager)            
               .tokenStore(redisTokenStore);//指定存储策略    
       }    ......
   
   }
   ```

   

### 4.2、  使用JWT替换默认令牌

使用JWT替换默认的令牌（默认令牌使用UUID生成）只需要指定TokenStore为JwtTokenStore即可。

- 配置类JWTokenConfig

  ```java
  @Configuration
  public class JWTokenConfig {
      
      @Bean    
      public TokenStore jwtTokenStore() {        
          return new JwtTokenStore(jwtAccessTokenConverter());    
      }    
      
      @Bean    
      public JwtAccessTokenConverter jwtAccessTokenConverter() {        
          JwtAccessTokenConverter accessTokenConverter = new JwtAccessTokenConverter();         accessTokenConverter.setSigningKey("test_key"); // 签名密钥        
          return accessTokenConverter;    
      }
  }
  ```

- 在认证服务器里指定

  ```java
  @Configuration
  @EnableAuthorizationServer
  public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {    
      @Autowired    
      private TokenStore jwtTokenStore;    
      //JWT的
      @Autowired 
      private JwtAccessTokenConverter jwtAccessTokenConverter;    
      
      @Override    
      public void configure(AuthorizationServerEndpointsConfigurer endpoints) {        
          endpoints.authenticationManager(authenticationManager)                
              .tokenStore(jwtTokenStore)                
              .accessTokenConverter(jwtAccessTokenConverter);    
      }    
      ......
  }
  ```

- 解析返回的access_token

  https://jwt.io/



### 4.3、  拓展JWT

- 解析JWT

  ```json
  `{  "exp": 1561532501,  "user_name": "mrbird",  "authorities": [    "admin"  ],  "jti": "2df680ca-af7d-4e86-997a-b5fedc41ff0e",  "client_id": "test1",  "scope": ["all"]}`
  ```



- 在JWT中添加额外信息，需要实现`TokenEnhancer`（Token增强器）：

  ```java
  public class JWTokenEnhancer implements TokenEnhancer {    
      @Override    
      public OAuth2AccessToken enhance(OAuth2AccessToken oAuth2AccessToken, OAuth2Authentication oAuth2Authentication) {        
          
          Map<String, Object> info = new HashMap<>();        
          info.put("message", "hello world");        
          ((DefaultOAuth2AccessToken) oAuth2AccessToken).setAdditionalInformation(info);         return oAuth2AccessToken; 
          
      }
  }
  ```



- Token中添加了`message: hello world`信息。然后在`JWTokenConfig`里注册该Bean：

  ```java
  @Configuration
  public class JWTokenConfig {    
      ......    
      @Bean    
      public TokenEnhancer tokenEnhancer() {        
          return new JWTokenEnhancer();    
      }
  }
  ```



- 认证服务器里配置该增强器：

  ```java
  @Configuration
  @EnableAuthorizationServer
  public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {    
      @Autowired    
      private TokenStore jwtTokenStore;    
      //JWT
      @Autowired    
      private JwtAccessTokenConverter jwtAccessTokenConverter;    
      
      //增加JWT返回
      @Autowired    
      private TokenEnhancer tokenEnhancer;    
      
      @Override    
      public void configure(AuthorizationServerEndpointsConfigurer endpoints) {        
          
          TokenEnhancerChain enhancerChain = new TokenEnhancerChain();        
          List<TokenEnhancer> enhancers = new ArrayList<>();
          
          //添加额外JWT信息
          enhancers.add(tokenEnhancer);
          //JWT对象
          enhancers.add(jwtAccessTokenConverter); 
   		//设置Chain
          enhancerChain.setTokenEnhancers(enhancers); 
          
          
          endpoints
              .tokenStore(jwtTokenStore)                
              .accessTokenConverter(jwtAccessTokenConverter)                
              .tokenEnhancer(enhancerChain);    
      }   
      ......
  }
  ```



### 4.4、  Java中解析JWT

- 依赖

  ```xml
  <dependency>    
      <groupId>io.jsonwebtoken</groupId>    
      <artifactId>jjwt</artifactId>    
      <version>0.9.1</version>
  </dependency>
  ```



- 修改index

  ```java
  @GetMapping("index")
  public Object index(@AuthenticationPrincipal Authentication authentication, HttpServletRequest request) {
      
      String header = request.getHeader("Authorization");    
      String token = StringUtils.substringAfter(header, "bearer ");    
      return Jwts.parser()
          .setSigningKey("test_key".getBytes(StandardCharsets.UTF_8))
          .parseClaimsJws(token)
          .getBody();}
  ```





- 拿到令牌之后 访问index

  ```java
  `{    "exp": 1561557893,    "user_name": "mrbird",    "authorities": [        "admin"    ],    "jti": "3c29f89a-1344-40d8-bcfd-1b9c45fb8b89",    "client_id": "test1",    "scope": [        "all"    ]}`
  ```



### 4.5、  刷新令牌

​		令牌过期后我们可以使用refresh_token来从系统中换取一个新的可用令牌。但是从前面的例子可以看到，在认证成功后返回的JSON信息里并没有包含refresh_token，要让系统返回refresh_token，需要在认证服务器自定义配置里添加如下配置：

- 配置

  ```java
  @Configuration
  @EnableAuthorizationServer
  public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {	......    
      @Override    
      public void configure(ClientDetailsServiceConfigurer clients) throws Exception {        
      	clients.inMemory()                
              .withClient("test1")                
              .secret(new BCryptPasswordEncoder().encode("test1111"))                
              .authorizedGrantTypes("password", "refresh_token")                
              .accessTokenValiditySeconds(3600)                
              .refreshTokenValiditySeconds(864000)                
              .scopes("all", "a", "b", "c")            
          .and()                
              .withClient("test2")                
              .secret(new BCryptPasswordEncoder().encode("test2222"))                
              .accessTokenValiditySeconds(7200);    
     }
                                                                                      }
  ```

  ​	    授权方式需要加上`refresh_token`，除了四种标准的OAuth2获取令牌方式外，Spring Security OAuth2内部把`refresh_token`当作一种拓展的获取令牌方式

  ​         通过上面的配置，使用test1这个client_id获取令牌时将返回refresh_token，refresh_token的有效期为10天，即10天之内都可以用它换取新的可用令牌。

- 重新获取令牌

  - 获得令牌

    ```json
    {    
        "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1NjE1NTgwOTcsInVzZXJfbmFtZSI6Im1yYmlyZCIsImF1dGhvcml0aWVzIjpbImFkbWluIl0sImp0aSI6Ijg2NTdhMDBlLTFiM2MtNDA5NS1iMjNmLTJlMjUxOWExZmUwMiIsImNsaWVudF9pZCI6InRlc3QxIiwic2NvcGUiOlsiYWxsIl19.hrxKOz3NKY6Eq8k5QeOqKhXUQ4aAbicrb6J5y-LBRA0",    
        "token_type": "bearer",    
        "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJtcmJpcmQiLCJzY29wZSI6WyJhbGwiXSwiYXRpIjoiODY1N2EwMGUtMWIzYy00MDk1LWIyM2YtMmUyNTE5YTFmZTAyIiwiZXhwIjoxNTYyNDE4NDk3LCJhdXRob3JpdGllcyI6WyJhZG1pbiJdLCJqdGkiOiI2MTNjMDVlNS1hNzUzLTRmM2UtOWViOC1hZGE4MTJmY2IyYWQiLCJjbGllbnRfaWQiOiJ0ZXN0MSJ9.efw9OePFUN9X6UGMF3h9BF_KO3zqyIfpvfmE8XklBDs",    
        "expires_in": 3599,   
        "scope": "all",    
        "jti": "8657a00e-1b3c-4095-b23f-2e2519a1fe02"
    }
    ```

  - 发送重新获得令牌请求：

    - grant_type:refresh_token

    - refresh_token:access_token

      

## 5、  单点登录 Spring Security OAuth2 SSO

- 准备

  artifactId为sso-server（作为认证服务器）

  rtifactId为sso-application-one（作为客户端一）

  另外一个客户端和sso-application-one一致，只不过artifactId为sso-application-two。



### 5.1、  认证服务器配置

​		作为统一令牌发放并校验的地方，所以我们先要编写一些基本的Spring Security 安全配置的代码指定如何进行用户认证。

1. 配置  

   ```java
   @Configuration
   public class UserDetailService implements UserDetailsService {
   
       @Autowired
       private PasswordEncoder passwordEncoder;
   
       @Override
       public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
           MyUser user = new MyUser();
           user.setUserName(username);
           user.setPassword(this.passwordEncoder.encode("123456"));
   
           return new User(
               username,
               user.getPassword(),
               user.isEnabled(),
               user.isAccountNonExpired(), 
               user.isCredentialsNonExpired(),
               user.isAccountNonLocked(), 
               AuthorityUtils.commaSeparatedStringToAuthorityList("user:add"));
       }
   }
   ```
   基本逻辑是用户名随便写，密码为123456，并且拥有`user:add`权限

2. 认证服务器配置 SsoAuthorizationServerConfig

   ```java
   @Configuration
   @EnableAuthorizationServer
   public class SsoAuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {    
       @Autowired    
       private PasswordEncoder passwordEncoder;    
       
       @Autowired    
       private UserDetailService userDetailService;    
       
       @Bean    
       public TokenStore jwtTokenStore() {        
           return new JwtTokenStore(jwtAccessTokenConverter());    
       }    
       
       @Bean    
       public JwtAccessTokenConverter jwtAccessTokenConverter() {        
           JwtAccessTokenConverter accessTokenConverter = new JwtAccessTokenConverter();         accessTokenConverter.setSigningKey("test_key");        
           return accessTokenConverter;    
       }    
       
       @Override    
       public void configure(ClientDetailsServiceConfigurer clients) throws Exception {            
        }   
       
       @Override    
       public void configure(AuthorizationServerEndpointsConfigurer endpoints) {        
           endpoints
               .tokenStore(jwtTokenStore())                
               .accessTokenConverter(jwtAccessTokenConverter())                
               .userDetailsService(userDetailService);    
       }    
       
       @Override    
       public void configure(AuthorizationServerSecurityConfigurer security) {
           security.tokenKeyAccess("isAuthenticated()"); // 获取密钥需要身份认证    
       }
   }
   ```

   

### 5.2、  客户端配置

1. 开启SSO

   ```java
   @EnableOAuth2Sso
   @SpringBootApplication
   public class SsoApplicaitonOne {    
       public static void main(String[] args) {        
           new SpringApplicationBuilder(SsoApplicaitonOne.class).run(args);    
       }
   }
   ```

2. 项目配置文件

   ```yml
   security:  
   	oauth2:    
   		client:      
   			client-id: app-a      
   			client-secret: app-a-1234     
               user-authorization-uri: http://127.0.0.1:8080/server/oauth/authorize      
               access-token-uri: http://127.0.0.1:8080/server/oauth/token    
               resource:      
               	jwt:        
               		key-uri: http://127.0.0.1:8080/server/oauth/token_key
   server:  
   	port: 9090  
   	servlet:    
           context-path: /app1
   ```

   `		security.oauth2.client.client-id`和`security.oauth2.client.client-secret`指定了客户端id和密码，这里和认证服务器里配置的client一致（另外一个客户端为app-b）；

   `user-authorization-uri`指定为认证服务器的`/oauth/authorize`地址，`access-token-uri`指定为认证服务器的`/oauth/token`地址，`jwt.key-uri`指定为认证服务器的`/oauth/token_key`地址。

### 5.3、  测试

1. 在resources/static下新增一个index.html页面

   ```html
   <!DOCTYPE html>
   <html lang="en">
       
       <head>    
           <meta charset="UTF-8">    
           <title>管理系统一</title>
       </head>
       
       <body>    
       	<h1>管理系统一</h1>    
       	<a href="http://127.0.0.1:9091/app2/index.html">跳转到管理系统二</a>
       </body>
   </html>
   ```

2. 新增一个控制器

   ```java
   @RestController
   public class UserController {    
       @GetMapping("user")    
       public Principal user(Principal principal) {        
           return principal;    
       }
   }
   ```

3. 测试访问http://127.0.0.1:9090/app1/index.html：

   结果：

   ​		非法请求，至少需要一个重定向URL被注册到client。从URL中可以看出，redirect_uri为http://127.0.0.1:9090/app1/login，所以我们修改认证服务器client相关配置如下：

   ```java
   @Override
   public void configure(ClientDetailsServiceConfigurer clients) throws Exception {    
       clients.inMemory()            
           .withClient("app-a")            
           .secret(passwordEncoder.encode("app-a-1234"))            
           .authorizedGrantTypes("refresh_token","authorization_code")            
           .accessTokenValiditySeconds(3600)            
           .scopes("all")
           //新增
           .redirectUris("http://127.0.0.1:9090/app1/login")        
     .and()            
           .withClient("app-b")            
           .secret(passwordEncoder.encode("app-b-1234"))            
           .authorizedGrantTypes("refresh_token","authorization_code")            
           .accessTokenValiditySeconds(7200)            
           .scopes("all")
           //新增
           .redirectUris("http://127.0.0.1:9091/app2/login");
   }
   ```

4. 自动授权

   跳转会提示用户是否授权

   ```java
   @Override
   public void configure(ClientDetailsServiceConfigurer clients) throws Exception {    
       clients.inMemory()            
           .withClient("app-a")            
           .secret(passwordEncoder.encode("app-a-1234"))            
           .authorizedGrantTypes("refresh_token","authorization_code")            
           .accessTokenValiditySeconds(3600)            
           .scopes("all")
           //自动授权
           .autoApprove(true)            
           .redirectUris("http://127.0.0.1:9090/app1/login")       
      .and()           
           .withClient("app-b")            
           .secret(passwordEncoder.encode("app-b-1234"))            
           .authorizedGrantTypes("refresh_token","authorization_code")            
           .accessTokenValiditySeconds(7200)            
           .scopes("all")
           //自动授权
           .autoApprove(true)            
           .redirectUris("http://127.0.0.1:9091/app2/login");}
   ```



### 5.3、  权限校验

- 控制器

  ```java
  @GetMapping("auth/test1")
  @PreAuthorize("hasAuthority('user:add')")
  public String authTest1(){    
      return "您拥有'user:add'权限";
  }
  
  @GetMapping("auth/test2")
  @PreAuthorize("hasAuthority('user:update')")
  public String authTest2(){    
      return "您拥有'user:update'权限";
  }
  ```

- 客户端新增Spring Security配置类:

  ```java
  @Configuration
  @EnableGlobalMethodSecurity(prePostEnabled = true)
  public class WebSecurityConfigurer extends WebSecurityConfigurerAdapter {
      
  }
  ```



- 客户端Spring Security配置和配置服务器冲突

  更改order使优先级小于认证服务器的Spring Security配置。

  ```java
  @Order(101)
  @Configuration
  @EnableGlobalMethodSecurity(prePostEnabled = true)
  public class WebSecurityConfigurer extends WebSecurityConfigurerAdapter {
  
  }
  ```



- 测试