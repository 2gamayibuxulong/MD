



# Cas5.3.2 单点登陆 

- 环境 tomcat8  overlays  java1.8

- 源码及说明
  - https://github.com/Clever-Wang/spring-boot-cas
  - https://blog.csdn.net/qq_34021712/article/details/80871015
  - Cas-server端采用overlays方式覆盖源码打包部署测试


## 1、  认证策略

#### 1.1、  密码认证

- JDBC认证(密码MD5和密码加盐)
  - 配置jdbc
  - 密码进行MD5加密
  - 密码加盐



- 自定义认证

  - 实现PasswordEncoder接口，重写encode和matches方法

  - 配置自己写的认证类

    

#### 1.2、自定义登录验证

除账号密码外 设置其余限制，例如权限

- 原理

  1. CAS服务器的org.apereo.cas.authentication.AuthenticationManager负责基于提供的凭证信息进行用户认证
  2. 认证委托给了一个或多个实现了org.apereo.cas.authentication.AuthenticationHandler接口的处理类
  3. cas的认证过程中逐个执行authenticationHandlers中配置的认证管理，直到有一个成功为止。
  4. CAS内置了一些AuthenticationHandler实现类，如下图所示，QueryDatabaseAuthenticationHandler中提供了基于jdbc的用户认证

  ![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20180721150827252?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0MDIxNzEy/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



- 实现自己的验证器

  1. 继承上图任意一个handler,例如AbstractUsernamePasswordAuthenticationHandler

  2. 实现authenticateUsernamePasswordInternal，完成相关验证逻辑，例如查询根据账号信息查询账号权限，如果不具备权限，则不允许登录

  3. 注册验证器

     
     
     

## 2、  鉴权

#### 2.1、 在服务端集成鉴权

- 使用shiro进行认证和鉴权

  - 依赖

    ```xml
    <dependency>
        <groupId>org.apereo.cas</groupId>
        <artifactId>cas-server-support-shiro-authentication</artifactId>
        <version>${cas.version}</version>
    </dependency>
    ```

    

  - 配置

    ```properties
    #整合shiro
    #允许登录的用户，必须要有以下角色，否则拒绝，多个逗号隔开
    cas.authn.shiro.requiredRoles=admin
    #允许登录的用户，必须要有以下权限，否则拒绝，多个逗号隔开
    cas.authn.shiro.requiredPermissions=userInfo:add,userInfo:view
    #shir配置文件位置
    cas.authn.shiro.location=classpath:shiro.ini
    #shiro name 唯一
    cas.authn.shiro.name=cas-shiro
    #与Query Authentication一致的加密策略
    cas.authn.shiro.passwordEncoder.type=DEFAULT
    cas.authn.shiro.passwordEncoder.characterEncoding=UTF-8
    cas.authn.shiro.passwordEncoder.encodingAlgorithm=MD5
    ```

    

  - shiro.ini

    ```ini
    [main]
    cacheManager = org.apache.shiro.cache.MemoryConstrainedCacheManager
    securityManager.cacheManager = $cacheManager
    
    [users]
    #密码123
    admin = e10adc3949ba59abbe56e057f20f883e, admin
    #不可登录，因为配置了需要角色admin
    #密码123456
    test = ed0290f05224a188160858124a5f5077, test
    
    [roles]
    admin = userInfo:*
    test = commit:*
    ```

    

- 自定义登录验证器集成shiro
  - 继承cas支持的验证器AbstractUsernamePasswordAuthenticationHandler
  
  - 配置shiro realm 指定认证授权逻辑
  
  - 注册验证器 添加shiro配置器
  
    



#### 2.2、  客户端集成cas:web.xml

- 服务端配置

  - 对所有https和http请求的service进行允许认证

    ```json
    {
        "@class" : "org.apereo.cas.services.RegexRegisteredService",
        "serviceId" : "^(https|imaps|http)://.*",
        "name" : "测试客户端",
        "id" : 10000001,
        "description" : "这是一个测试客户端的服务，所有的https或者http访问都允许通过",
        "evaluationOrder" : 10000
    }
    ```

    ● @class：必须为org.apereo.cas.services.RegisteredService的实现类
    ● serviceId：对服务进行描述的表达式，可用于匹配一个或多个 URL 地址
    ● name： 服务名称
    ● id：全局唯一标志
    ● description：服务描述，会显示在默认登录页
    ● evaluationOrder：定义多个服务的执行顺序

    

  - 告知 CAS 服务端从本地加载服务定义文件
  
    ```properties
    #注册客户端
    cas.serviceRegistry.initFromJson=true
    cas.serviceRegistry.watcherEnabled=true
    cas.serviceRegistry.schedule.repeatInterval=120000
    cas.serviceRegistry.schedule.startDelay=15000
    cas.serviceRegistry.managementType=DEFAULT
    cas.serviceRegistry.json.location=classpath:/services
    cas.logout.followServiceRedirects=true
    ```



- 客户端配置

  - web.xml

    ```xml
    <!-- ========================单点登录开始 ======================== -->
    
    <!-- 用于单点退出，该过滤器用于实现单点登出功能，可选配置 -->
    <listener>
    	<listener-class>
        	org.jasig.cas.client.session.SingleSignOutHttpSessionListener
        </listener-class>
    </listener>
    
    <!-- 该过滤器用于实现单点登出功能，可选配置。 -->
    <filter>
        <filter-name>CAS Single Sign Out Filter</filter-name>
        <filter-class>org.jasig.cas.client.session.SingleSignOutFilter</filter-class>
        <init-param>
            <param-name>casServerUrlPrefix</param-name>
            <param-value>https://server.cas.com:8443/cas</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CAS Single Sign Out Filter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    
    <!-- 该过滤器用于实现单点登录功能 -->
    <filter>
        <filter-name>CAS Filter</filter-name>
        <filter-class>
            org.jasig.cas.client.authentication.AuthenticationFilter
        </filter-class>
        
        <init-param>
            <param-name>casServerLoginUrl</param-name>
            <param-value>https://server.cas.com:8443/cas/login</param-value>
            <!-- 使用的CAS-Server的登录地址,一定是到登录的action -->
        </init-param>
        
        <init-param>
            <param-name>serverName</param-name>
            <param-value>http://app1.cas.com:8081</param-value>
            <!-- 当前Client系统的地址 -->
        </init-param> 
    </filter>
    <filter-mapping>
        <filter-name>CAS Filter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    
    <!-- 该过滤器负责对Ticket的校验工作 -->
    <filter>
        <filter-name>CAS Validation Filter</filter-name>
        <filter-class>
            org.jasig.cas.client.validation.Cas30ProxyReceivingTicketValidationFilter
        </filter-class>
        <init-param>
            <param-name>casServerUrlPrefix</param-name>
            <param-value>https://server.cas.com:8443/cas</param-value>
           <!-- 使用的CAS-Server的地址,一定是在浏览器输入该地址能正常打开CAS-Server的根地址 -->
        </init-param>
        <init-param>
            <param-name>serverName</param-name>
            <param-value>http://app1.cas.com:8081</param-value>
            <!-- 当前Client系统的地址 -->
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CAS Validation Filter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    
    <!-- 该过滤器负责实现HttpServletRequest请求的包裹， 比如允许开发者通过HttpServletRequest的getRemoteUser()方法获得SSO登录用户的登录名，可选配置。 -->
    <filter>
        <filter-name>CAS HttpServletRequest Wrapper Filter</filter-name>
        <filter-class>
                org.jasig.cas.client.util.HttpServletRequestWrapperFilter
        </filter-class>
    </filter>
    <filter-mapping>
        <filter-name>CAS HttpServletRequest Wrapper Filter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    
    <!--
       该过滤器使得开发者可以通过org.jasig.cas.client.util.AssertionHolder来获取用户的登录名。
             比如AssertionHolder.getAssertion().getPrincipal().getName()
             或者request.getUserPrincipal().getName()
    -->
    <filter>
        <filter-name>CAS Assertion Thread Local Filter</filter-name>
        <filter-class>
            org.jasig.cas.client.util.AssertionThreadLocalFilter
        </filter-class>
    </filter>
    <filter-mapping>
        <filter-name>CAS Assertion Thread Local Filter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
        <!-- ========================单点登录结束 ======================== -->
    ```



#### 2.3、  客户端集成cas:springBoot

- 集成

  - pom.xml

    ```xml
        <!--cas的客户端 -->
        <dependency>
            <groupId>net.unicon.cas</groupId>
            <artifactId>cas-client-autoconfig-support</artifactId>
            <version>1.4.0-GA</version>
            <exclusions>
                <exclusion>
                    <groupId>org.jasig.cas.client</groupId>
                    <artifactId>cas-client-core</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.jasig.cas.client</groupId>
            <artifactId>cas-client-core</artifactId>
            <version>${java.cas.client.version}</version>
        </dependency>
    ```

  - application.properties

    ```properties
    #cas配置
    #cas服务端前缀,不是登录地址
    cas.server-url-prefix=https://server.cas.com:8443/cas
    #cas的登录地址
    cas.server-login-url=https://server.cas.com:8443/cas/login
    #当前客户端的地址
    cas.client-host-url=http://app2.cas.com:8082
    #Ticket校验器使用Cas30ProxyReceivingTicketValidationFilter
    cas.validation-type=CAS3
    ```

  - 开启CAS Client支持

    ```java
    @EnableCasClient//开启cas
    public class Application {
        public static void main(String[] args) {
        	SpringApplication.run(Application.class, args);
       }
    }
    ```

  - 使用java -jar 或者 在命令窗口执行 mvn clean spring-boot:run; 否则将找不到jsp。

  

#### 2.4、  自定义鉴权路径

​	图片等资源不需要登录也能查看

- web.xml项目

  - 配置资源映射 spring-mvc.xml

    ```xml
    <mvc:resources mapping="/images/**" location="WEB-INF/images/" />
    ```

  - 给web.xml中的AuthenticationFilter添加参数

    ```xml
    <filter>
        <filter-name>CAS Filter</filter-name>
        <filter-class>
            org.jasig.cas.client.authentication.AuthenticationFilter</filter-class>
        <init-param>
            <param-name>casServerLoginUrl</param-name>
            <param-value>https://server.cas.com:8443/cas/login</param-value>
            <!-- 使用的CAS-Server的登录地址,一定是到登录的action -->
        </init-param>
        <init-param>
            <param-name>serverName</param-name>
            <param-value>http://app1.cas.com:8081</param-value>
            <!-- 当前Client系统的地址 -->
        </init-param>
        <init-param>
            <param-name>ignorePattern</param-name>
            <param-value>/js/*|/images/*|/view/*|/css/*</param-value>
        </init-param>
    </filter>
    ```

    

- SpringBoot项目

  - 配置资源映射

    ```java
    @Configuration
    public class SpringMVC implements WebMvcConfigurer {
    @Override
        public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry
            .addResourceHandler("/images/**")
            .addResourceLocations("classpath:/static/images/");
        }
    }
    ```

    

  - 配置授权过滤器

    ```java
    /**
     * 授权过滤器
     * @return
     */
    @Bean
    public FilterRegistrationBean filterAuthenticationRegistration() {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        registration.setFilter(new AuthenticationFilter());
        // 设定匹配的路径
        registration.addUrlPatterns("/*");
        Map<String,String>  initParameters = new HashMap<String, String>();
        initParameters.put("casServerLoginUrl", CAS_SERVER_LOGIN_URL);
        initParameters.put("serverName", SERVER_NAME);
        initParameters.put("ignorePattern", "/js/*|/images/*|/view/*|/css/*");
        registration.setInitParameters(initParameters);
        // 设定加载的顺序
        registration.setOrder(1);
        return registration;
    }
    ```

    

## 3、  单点登出

​		单点登出的原理是，服务端收到登出请求时（请求中肯定会带tgt cookie），会向通过此tgt登入成功的service url（也就是所有通过此tgt登录成功的客户端）发出登出请求，此请求会有特殊的标记被客户端的filter捕获到，也就登出了。

- 参数说明

  - 可设置的属性值

    ```properties
    #配置单点登出
    #配置允许登出后跳转到指定页面
    cas.logout.followServiceRedirects=false
    #跳转到指定页面需要的参数名为 service
    cas.logout.redirectParameter=service
    #登出后需要跳转到的地址,如果配置该参数,service将无效。
    cas.logout.redirectUrl=https://www.taobao.com
    #在退出时是否需要 确认退出提示   true弹出确认提示框  false直接退出
    cas.logout.confirmLogout=true
    #是否移除子系统的票据
    cas.logout.removeDescendantTickets=true
    #禁用单点登出,默认是false不禁止
    #cas.slo.disabled=true
    #默认异步通知客户端,清除session
    #cas.slo.asynchronous=true
    ```

  - 说明

    ​		cas 默认登出后默认会跳转到CASServer的登出页，若想跳转到其它资源，可在/logout的URL后面加上service=jumpurl，例如：https://server.cas.com:8443/cas/logout?service=https://www.github.com
    但默认servcie跳转不会生效，需要在 cas服务端的application.properties添加

    ```properties
    cas.logout.followServiceRedirects=true
    ```

    ​		这个参数也不一定非要叫 service, 可以通过`cas.logout.redirectParameter` 来修改它。
    另外,默认退出的时候没有任何提示,直接就退出了,若想要有弹出提示,需要添加

    ```
    as.logout.confirmLogout=true
    ```

    ​		再另外,有一个cas.logout.redirectUrl的属性,可以配置默认登出之后跳转到的连接,若 配置该属性,service参数将无效。就算传了service参数,也是走的该页面，所以我们不需要配置此参数。
    ​		如果配置了cas.slo.disabled=true 将禁用单点登出。调用登出将无效。

    

- 服务器配置

  - application.properties

    ```properties
    #配置允许登出后跳转到指定页面
    cas.logout.followServiceRedirects=true
    #跳转到指定页面需要的参数名为 service
    cas.logout.redirectParameter=service
    #在退出时是否需要 确认一下  true确认 false直接退出
    cas.logout.confirmLogout=true
    #是否移除子系统的票据
    cas.logout.removeDescendantTickets=true
    ```

- 客户端配置

  ```java
      /**
       * 跳转到默认页面
       * @param session
       * @return
       */
      @RequestMapping("/logout1")
      public String loginOut(HttpSession session){
          session.invalidate();
          //这个是直接退出，走的是默认退出方式
          return "redirect:https://server.cas.com:8443/cas/logout";
      }
  
      /**
       * 跳转到指定页面
       * @param session
       * @return
       */
      @RequestMapping("/logout2")
      public String loginOut2(HttpSession session){
          session.invalidate();
          //退出登录后，跳转到退成成功的页面，不走默认页面
          return "redirect:https://server.cas.com:8443/cas/logout?service=http://app1.cas.com:8081";
      }
  ```



## 4、  rest认证

rest认证：cas服务端通过调用其他服务接口,将用户名和密码传过去进行认证。

在不允许cas服务直接访问账号数据库的时候,这个时候就需要用到Rest认证。

- 流程

  1. cas 服务端会通过post请求，并且把用户信息以”用户名:密码”进行Base64编码放在authorization请求头中

  2. 200状态码：并且格式为

     ```json
     {“@class”:”org.apereo.cas.authentication.principal.SimplePrincipal”,”id”:”casuser”,”attributes”:{}}
     ```

     则成功认证

  3. 其余

     ● 403状态码：用户不可用
     ● 404状态码：账号不存在
     ● 423状态码：账户被锁定
     ● 428状态码：过期
     ● 其他登录失败



- 服务器配置

  - 依赖

    ```xml
    <dependency>
        <groupId>org.apereo.cas</groupId>
        <artifactId>cas-server-support-rest-authentication</artifactId>
        <version>${cas.version}</version>
    </dependency>
    ```

  - 添加配置

    ```properties
    cas.authn.rest.uri=http://rest.cas.com:8083/login
    #如果密码有加密,打开下面配置,我的是明文
    #cas.authn.rest.passwordEncoder.type=DEFAULT
    #cas.authn.rest.passwordEncoder.characterEncoding=UTF-8
    #cas.authn.rest.passwordEncoder.encodingAlgorithm=MD5
    ```

    

- rest-client认证服务端配置

  - SysUser.java(rest-client响应体)

    ```java
    @JsonProperty("id")
    @NotNull
    private String username;
    
    //需要返回实现org.apereo.cas.authentication.principal.Principal的类名接口
    
    @JsonProperty("@class")
    private String clazz = "org.apereo.cas.authentication.principal.SimplePrincipal";
    
    @JsonProperty("attributes")
    private Map<String, Object> attributes = new HashMap<>();
    
    @JsonIgnore
    @NotNull
    private String password;
    
    
    //用户状态,根据状态判断是否可用
    @JsonIgnore
    private String state;
    ```

  - 验证用户信息的接口

    ```java
      /**
         * 1. cas 服务端会通过post请求，并且把用户信息以"用户名:密码"进行Base64编码放在authorization请求头中
         * 2. 返回200状态码并且格式为{"@class":"org.apereo.cas.authentication.principal.SimplePrincipal","id":"casuser","attributes":{}} 是成功的
         * 2. 返回状态码403用户不可用；404账号不存在；423账户被锁定；428过期；其他登录失败
         * @param httpHeaders
         * @return
         */
        @RequestMapping("/login")
        public Object login(@RequestHeader HttpHeaders httpHeaders){
    
            logger.info("开始验证服务");
    
            SysUser user = null;
            try {
                UserTemp userTemp = obtainUserFormHeader(httpHeaders);
                //尝试查找用户库是否存在
                user = userService.selectUser(userTemp.username);
                if (user != null) {
                    if (!user.getPassword().equals(userTemp.password)) {
                        //密码不匹配
                        return new ResponseEntity(HttpStatus.BAD_REQUEST);
                    }
                    if (!"0".equals(user.getState())) {
                        //用户已锁定
                        return new ResponseEntity(HttpStatus.LOCKED);
                    }
                } else {
                    //不存在 404
                    return new ResponseEntity(HttpStatus.NOT_FOUND);
                }
            } catch (UnsupportedEncodingException e) {
                logger.error("用户认证错误", e);
                new ResponseEntity(HttpStatus.BAD_REQUEST);
            }
            //成功返回json
            return user;
        }
    
        /**
         * This allows the CAS server to reach to a remote REST endpoint via a POST for verification of credentials.
         * Credentials are passed via an Authorization header whose value is Basic XYZ where XYZ is a Base64 encoded version of the credentials.
         * @param httpHeaders
         * @return
         * @throws UnsupportedEncodingException
         */
        private UserTemp obtainUserFormHeader(HttpHeaders httpHeaders) throws UnsupportedEncodingException {
    
            //cas服务端会通过把用户信息放在请求头authorization中，并且通过Basic认证方式加密
            String authorization = httpHeaders.getFirst("authorization");
            if(StringUtils.isEmpty(authorization)){
                return null;
            }
    
            String baseCredentials = authorization.split(" ")[1];
            //用户名:密码
            String usernamePassword = new String(Base64Utils.decodeFromString(baseCredentials), "UTF-8");
            String[] credentials = usernamePassword.split(":");
    
            return new UserTemp(credentials[0], credentials[1]);
        }
    
        /**
         * 从请求头中获取用户名和密码
         */
        private class UserTemp {
            private String username;
            private String password;
    
            public UserTemp(String username, String password) {
                this.username = username;
                this.password = password;
            }
        }
    ```

    

## 5、  动态添加Service

​	以往整合客户端的时候,需要在cas服务端注册,使用的是json文件的方式,更简单的一点,直接配置为只要是http或者https的请求,都表示注册。现在需要动态注册客户端。

#### 5.1、  手动指定

#### 5.2、  服务管理cas-management部署



## 6、  自定义登录页面

主题规范：

1.   静态资源(js,css)存放目录为src/main/resources/static

2.  html资源存(thymeleaf)放目录为src/main/resources/templates

3. 主题配置文件存放在src/main/resources并且命名为[theme_name].properties

4. 主题页面html存放目录为src/main/resources/templates/

   

- 主题配置

  ```json
  {
      "@class" : "org.apereo.cas.services.RegexRegisteredService",
      "serviceId" : "^(https|imaps|http)://app1.cas.com.*",
      "name" : "测试客户端app1",
      "id" : 1000,
      "description" : "这是app1的客户端",
      "evaluationOrder" : 10,
      "theme" : "app1"
  }
  ```

  

- properties

  ```properties
  #原cas默认的css样式,如果更改了,某些页面样式将丢失
  cas.standard.css.file=/css/cas.css
  #自己的样式
  cas.myself.css=/themes/app1/css/cas.css
  cas.javascript.file=/themes/app1/js/jquery-1.4.2.min.js
  cas.page.title=app1的主题
  ```



- 引用

  ```html
  <link rel="stylesheet" th:href="@{${#themes.code('cas.myself.css')}}"/>
  <script th:src="@{${#themes.code('cas.javascript.file')}}"></script> 
  ```

  上面配置文件中有cas.standard.css.file属性,这个属性默认就是指向/css/cas.css也就是cas默认的css文件,这个我们要指明,否则你只是自定义了登录页面,其他页面的样式将丢失。我们在自定义的登录页面使用自己的css文件,不跟cas的默认css混淆。

  

- css

  ```css
  h2 {
      color: red;
  }
  ```

  

- 默认主题

  ```properties
  # 默认主题
  cas.theme.defaultThemeName=app2
  ```

  

- 登录页面

  ```html
  <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
  <html  xmlns:th="http://www.thymeleaf.org">
  <head>
      <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
      <title th:text="${#themes.code('cas.page.title')}"></title>
      <link rel="stylesheet" th:href="@{${#themes.code('cas.myself.css')}}"/>
      <script th:src="@{${#themes.code('cas.javascript.file')}}"></script>
  </head>
  
  <body>
  	<h2 th:text="${#themes.code('cas.page.title')}"></h2>
  	<div>
      <form method="post" th:object="${credential}">
          <div th:if="${#fields.hasErrors('*')}">
              <span th:each="err : ${#fields.errors('*')}" th:utext="${err}" style="color: red" />
          </div>
          <h4 th:utext="#{screen.welcome.instructions}"></h4>
          <section class="row">
              <label for="username" th:utext="#{screen.welcome.label.netid}" />
              <div th:unless="${openIdLocalId}">
                  <input class="required" id="username" size="25" tabindex="1" type="text"
                         th:disabled="${guaEnabled}"
                         th:field="*{username}"
                         th:accesskey="#{screen.welcome.label.netid.accesskey}"
                         autocomplete="off"
                         th:value="admin" />
              </div>
          </section>
          <section class="row">
              <label for="password" th:utext="#{screen.welcome.label.password}"/>
              <div>
                  <input class="required" type="password" id="password" size="25" tabindex="2"
                         th:accesskey="#{screen.welcome.label.password.accesskey}"
                         th:field="*{password}"
                         autocomplete="off"
                         th:value="123456" />
              </div>
          </section>
          <section>
              <input type="hidden" name="execution" th:value="${flowExecutionKey}" />
              <input type="hidden" name="_eventId" value="submit" />
              <input type="hidden" name="geolocation" />
              <input class="btn btn-submit btn-block" name="submit" accesskey="l" th:value="#{screen.welcome.button.login}" tabindex="6" type="submit" />
          </section>
      </form>
  </div>
  </body>
  </html>
  ```

  

## 7、  自定义确认登出页面

- casConfirmLogoutView.html

  ```html
  <!DOCTYPE html>
  <html xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout" layout:decorate="~{layout}">
  
  <head>
      <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
      <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no"/>
  
      <title th:text="#{screen.logout.confirm.header}">Confirm Logout View</title>
      <link href="../../static/css/cas.css" rel="stylesheet" th:remove="tag" />
  </head>
  
  <body>
  <main role="main" class="container mt-3 mb-3">
      <div layout:fragment="content">
          <div class="alert alert-info">
              <h2>Do you want to log out completely?</h2>
              <p>
              确认退出将结束你的会话,跳转到登录页面。
              </p>
              <p><br>你确认退出吗?</p>
  
              <div align="center">
                  <form method="post" id="fm1">
                      <input type="hidden" name="LogoutRequestConfirmed" value="true"/>
                      <input type="hidden" name="execution" th:value="${flowExecutionKey}"/>
                      <input type="hidden" name="_eventId" value="success"/>
                      <input class="btn btn-primary"
                             name="success"
                             accesskey="l"
                             th:value="#{screen.welcome.button.logout}"
                             value="Logout"
                             type="submit"/>
                  </form>
              </div>
          </div>
      </div>
  </main>
  </body>
  </html>
  ```

  

## 8、  记住我

- 配置

  ```properties
  #记住我
  cas.ticket.tgt.rememberMe.enabled=true
  cas.ticket.tgt.rememberMe.timeToKillInSeconds=3600
  ```

- 页面

   <p>
                  <input type="checkbox" name="rememberMe" id="rememberMe" value="true" tabindex="5"/>
                  <label for="rememberMe" th:text="#{screen.rememberme.checkbox.title}">Remember Me</label>
              </p>

​	



## 9、  验证码

1. 重写credential，继承RememberMeUsernamePasswordCredential，加上capcha属性

   ```java
   public class RememberMeUsernamePasswordCaptchaCredential extends RememberMeUsernamePasswordCredential {
       
       @Size(min = 5,max = 5, message = "require captcha")
       private String captcha;
       public String getCaptcha() {
       return captcha;
       }
       
       public void setCaptcha(String captcha) {
       this.captcha = captcha;
       }
       @Override
       public int hashCode() {
   	return new HashCodeBuilder()
                   .appendSuper(super.hashCode())
                   .append(this.captcha)
                   .toHashCode();
   	}
   }
   ```

   

2. 新建DefaultCaptchaWebflowConfigurer修改之前默认的Credential

   ```java
   /**
    * @author: wangsaichao
    * @date: 2018/8/31
    * @description: 重新定义 Credential model
    */
   public class DefaultCaptchaWebflowConfigurer extends DefaultLoginWebflowConfigurer {
       /**
        * Instantiates a new Default webflow configurer.
        *
        * @param flowBuilderServices    the flow builder services
        * @param flowDefinitionRegistry the flow definition registry
        * @param applicationContext     the application context
        * @param casProperties          the cas properties
        */
       public DefaultCaptchaWebflowConfigurer(FlowBuilderServices flowBuilderServices, FlowDefinitionRegistry flowDefinitionRegistry, ApplicationContext applicationContext, CasConfigurationProperties casProperties) {
           super(flowBuilderServices, flowDefinitionRegistry, applicationContext, casProperties);
       }
   
   
       /**
        * Create remember me authn webflow config.
        *
        * @param flow the flow
        */
       @Override
       protected void createRememberMeAuthnWebflowConfig(Flow flow) {
           if (casProperties.getTicket().getTgt().getRememberMe().isEnabled()) {
               createFlowVariable(flow, CasWebflowConstants.VAR_ID_CREDENTIAL, RememberMeUsernamePasswordCaptchaCredential.class);
               final ViewState state = getState(flow, CasWebflowConstants.STATE_ID_VIEW_LOGIN_FORM, ViewState.class);
               final BinderConfiguration cfg = getViewStateBinderConfiguration(state);
               cfg.addBinding(new BinderConfiguration.Binding("rememberMe", null, false));
               cfg.addBinding(new BinderConfiguration.Binding("captcha", null, true));
           } else {
               createFlowVariable(flow, CasWebflowConstants.VAR_ID_CREDENTIAL, UsernamePasswordCredential.class);
           }
       }
   }
   ```

   

3. 创建表单处理器

   ```java
   public class RememberMeUsernamePasswordCaptchaAuthenticationHandler extends AbstractPreAndPostProcessingAuthenticationHandler {
   
   
       private UserService userService;
   
       public UserService getUserService() {
           return userService;
       }
   
       public void setUserService(UserService userService) {
           this.userService = userService;
       }
   
       public RememberMeUsernamePasswordCaptchaAuthenticationHandler(String name, ServicesManager servicesManager, PrincipalFactory principalFactory, Integer order) {
           super(name, servicesManager, principalFactory, order);
       }
   
       @Override
       protected AuthenticationHandlerExecutionResult doAuthentication(Credential credential) throws GeneralSecurityException, PreventedException {
           RememberMeUsernamePasswordCaptchaCredential myCredential = (RememberMeUsernamePasswordCaptchaCredential) credential;
           String requestCaptcha = myCredential.getCaptcha();
           ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
           Object attribute = attributes.getRequest().getSession().getAttribute("captcha");
   
           String realCaptcha = attribute == null ? null : attribute.toString();
   
           if(StringUtils.isBlank(requestCaptcha) || !requestCaptcha.toUpperCase().equals(realCaptcha)){
               throw new FailedLoginException("验证码错误");
           }
   
           String username = myCredential.getUsername();
           Map<String, Object> user = userService.findByUserName(username);
   
           //可以在这里直接对用户名校验,或者调用 CredentialsMatcher 校验
           if (user == null || !user.get("password").equals(myCredential.getPassword())) {
               throw new UnknownAccountException("用户名或密码错误！");
           }
           //这里将 密码对比 注销掉,否则 无法锁定  要将密码对比 交给 密码比较器 在这里可以添加自己的密码比较器等
           //if (!password.equals(user.getPassword())) {
           //    throw new IncorrectCredentialsException("用户名或密码错误！");
           //}
           if ("1".equals(user.get("state"))) {
               throw new LockedAccountException("账号已被锁定,请联系管理员！");
           }
           return createHandlerResult(credential, this.principalFactory.createPrincipal(username));
       }
   
       @Override
       public boolean supports(Credential credential) {
           return credential instanceof RememberMeUsernamePasswordCaptchaCredential;
       }
   }
   ```

   

4. 新建配置类来 配置 DefaultCaptchaWebflowConfigurer 和 表单处理器

   ```java
   @Configuration("captchaWebflowConfiguration")
   @EnableConfigurationProperties(CasConfigurationProperties.class)
   @AutoConfigureBefore(value = CasWebflowContextConfiguration.class)
   public class CaptchaWebflowConfiguration {
   
       @Autowired
       @Qualifier("loginFlowRegistry")
       private FlowDefinitionRegistry loginFlowRegistry;
       @Autowired
       private ApplicationContext applicationContext;
       @Autowired
       private CasConfigurationProperties casProperties;
       @Autowired
       private FlowBuilderServices flowBuilderServices;
   
       @Bean("defaultLoginWebflowConfigurer")
       public CasWebflowConfigurer defaultLoginWebflowConfigurer() {
           DefaultCaptchaWebflowConfigurer c = new DefaultCaptchaWebflowConfigurer(flowBuilderServices, loginFlowRegistry, applicationContext, casProperties);
           c.initialize();
           return c;
       }
   }
   ```

   

5. 配置表单处理器

   ```java
   @Configuration("rememberMeConfiguration")
   @EnableConfigurationProperties(CasConfigurationProperties.class)
   public class RememberMeConfiguration implements AuthenticationEventExecutionPlanConfigurer {
   
       @Autowired
       private CasConfigurationProperties casProperties;
   
       @Autowired
       @Qualifier("servicesManager")
       private ServicesManager servicesManager;
   
       @Autowired
       private UserService userService;
   
   
       /**
        * 放到 shiro(order=10) 验证器的前面 先验证验证码
        * @return
        */
       @Bean
       public AuthenticationHandler rememberMeUsernamePasswordCaptchaAuthenticationHandler() {
           RememberMeUsernamePasswordCaptchaAuthenticationHandler handler = new RememberMeUsernamePasswordCaptchaAuthenticationHandler(
                   RememberMeUsernamePasswordCaptchaAuthenticationHandler.class.getSimpleName(),
                   servicesManager,
                   new DefaultPrincipalFactory(),
                   9);
           handler.setUserService(userService);
           return handler;
       }
   
       @Override
       public void configureAuthenticationExecutionPlan(AuthenticationEventExecutionPlan plan) {
           plan.registerAuthenticationHandler(
               rememberMeUsernamePasswordCaptchaAuthenticationHandler());
       }
   }
   ```

   

6. 绘制验证码 以图片形式返回

   ```java
   @Controller
   public class CaptchaController {
       public static final String KEY_CAPTCHA = "captcha";
       
       @RequestMapping("/captcha.jpg")
       public void getCaptcha(HttpServletRequest request, HttpServletResponse response)throws ServletException, IOException {
           // 设置相应类型,告诉浏览器输出的内容为图片
           response.setContentType("image/jpeg");
           // 不缓存此内容
           response.setHeader("Pragma", "No-cache");
           response.setHeader("Cache-Control", "no-cache");
           response.setDateHeader("Expire", 0);
           
           try {
               HttpSession session = request.getSession();
               CaptchaUtil tool = new CaptchaUtil();
               StringBuffer code = new StringBuffer();
               BufferedImage image = tool.genRandomCodeImage(code);
               session.removeAttribute(KEY_CAPTCHA);
               session.setAttribute(KEY_CAPTCHA, code.toString());
               // 将内存中的图片通过流动形式输出到客户端
               ImageIO.write(image, "JPEG", response.getOutputStream());
           } catch (Exception e) {
           	e.printStackTrace();
           }
           
       }
   }
   ```

   

7. 页面展示

   ```html
    <img th:src="@{/captcha.jpg}" id="captcha_img" onclick="javascript:refreshCaptcha()"/>
   <!--(看不清<a href="javascript:void(0)" οnclick="javascript:refreshCaptcha()">换一张</a>)-->
   
   <script type="text/javascript">
       function refreshCaptcha(){
           $("#captcha_img").attr("src","/cas/captcha.jpg?id=" + new Date() + Math.floor(Math.random()*24));
       }
   </script>
   ```





## 10、  自定义错误信息返回

**1.**创建 一个异常类继承javax.security.auth.login.AccountExpiredException
**2.**配置异常到application.properties中
**3.**在messages_zh_CN.properties 配置文件中，配置异常弹出的消息
使用maven的Overlay编译好之后,默认已经有`messages_zh_CN.properties`文件了,直接拷贝过来,在基础上进行修改。



## 11、  自定义返回信息给客户端

在注册Service的json中配置返回信息的规则

● Return All （所有配置返回的都返回）
● Deny All （配置拒绝的出现则报错）
● Return Allowed（只返回允许的主要属性）
● 自定义Filter（自定义过滤策略）



1. 配置注册service的json文件

   配置返回所有信息

   ```json
   {
     "@class" : "org.apereo.cas.services.RegexRegisteredService",
     "serviceId" : "^(https|imaps|http)://app1.cas.com.*",
     "name" : "测试客户端app1",
     "id" : 1000,
     "description" : "这是app1的客户端",
     "evaluationOrder" : 10,
     "theme" : "app1",
     "attributeReleasePolicy" : {
       "@class" : "org.apereo.cas.services.ReturnAllAttributeReleasePolicy"
     }
   }
   ```

   配置返回姓名和身份证号

   ```json
   {
     "@class" : "org.apereo.cas.services.RegexRegisteredService",
     "serviceId" : "^(https|imaps|http)://app2.cas.com.*",
     "name" : "测试客户端app2",
     "id" : 1001,
     "description" : "这是app2的客户端",
     "evaluationOrder" : 11,
     "theme" : "app2",
     "attributeReleasePolicy" : {
       "@class" : "org.apereo.cas.services.ReturnAllowedAttributeReleasePolicy"
       "allowedAttributes" : [ "java.util.ArrayList", [ "name", "id_card_num" ] ]
     }
   }
   ```

   

2. 修改表单处理器Handler中添加返回结果

   ```java
       @Override
       protected AuthenticationHandlerExecutionResult doAuthentication(Credential credential) throws GeneralSecurityException, PreventedException {
           RememberMeUsernamePasswordCaptchaCredential myCredential = (RememberMeUsernamePasswordCaptchaCredential) credential;
           String requestCaptcha = myCredential.getCaptcha();
           ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
           Object attribute = attributes.getRequest().getSession().getAttribute("captcha");
   
           String realCaptcha = attribute == null ? null : attribute.toString();
   
           if(StringUtils.isBlank(requestCaptcha) || !requestCaptcha.toUpperCase().equals(realCaptcha)){
               throw new CaptchaErrorException("验证码错误");
           }
   
           String username = myCredential.getUsername();
           Map<String, Object> user = userService.findByUserName(username);
   
           if(user == null){
               throw new MyAccountNotFoundException("用户不存在");
           }
   
           //可以在这里直接对用户名校验,或者调用 CredentialsMatcher 校验
           if (!user.get("password").equals(myCredential.getPassword())) {
               throw new CredentialExpiredException("用户名或密码错误！");
           }
           //这里将 密码对比 注销掉,否则 无法锁定  要将密码对比 交给 密码比较器 在这里可以添加自己的密码比较器等
           //if (!password.equals(user.getPassword())) {
           //    throw new IncorrectCredentialsException("用户名或密码错误！");
           //}
           if ("1".equals(user.get("state"))) {
               throw new AccountLockedException("账号已被锁定,请联系管理员！");
           }
           return createHandlerResult(credential, this.principalFactory.createPrincipal(username,user));
       }
   ```

   



## 12、  Cas Server开启Oauth2.0协议

- 依赖

  ```xml
  <dependency>
      <groupId>org.apereo.cas</groupId>
      <artifactId>cas-server-support-oauth-webflow</artifactId>
      <version>${cas.version}</version>
  </dependency>
  ```

  

- 配置

  ```properties
  cas.authn.oauth.refreshToken.timeToKillInSeconds=2592000
  cas.authn.oauth.code.timeToKillInSeconds=30
  cas.authn.oauth.code.numberOfUses=1
  cas.authn.oauth.accessToken.releaseProtocolAttributes=true
  cas.authn.oauth.accessToken.timeToKillInSeconds=7200
  cas.authn.oauth.accessToken.maxTimeToLiveInSeconds=28800
  cas.authn.oauth.grants.resourceOwner.requireServiceHeader=true
  cas.authn.oauth.userProfileViewType=NESTED
  ```

  

- 增加接入servcie的注册文件

  ```json
  {
      "@class" : "org.apereo.cas.support.oauth.services.OAuthRegisteredService",
      "clientId": "20180901",
      "clientSecret": "123456",
      "serviceId" : "^(https|http|imaps)://.*",
      "name" : "OAuthService",
      "id" : 1002
  }
  ```

  

- 端点介绍

  | 端点                                  | 描述                             | 方法类型 |
  | ------------------------------------- | -------------------------------- | -------- |
  | /oauth2.0/authorize                   | 获取authCode或者token            | GET      |
  | /oauth2.0/accessToken,/oauth2.0/token | 获取accessToken                  | POST     |
  | /oauth2.0/profile                     | 通过access_token参数获取用户信息 | GET      |



- /oauth2.0/authorize方法介绍

  | 端点                | 请求参数                                               | 响应内容                                         |
  | ------------------- | ------------------------------------------------------ | ------------------------------------------------ |
  | /oauth2.0/authorize | response_type=code&client_id=ID&redirect_uri=CALLBACK  | 会重定向到redirect_uri这个地址并携带authCode参数 |
  | /oauth2.0/authorize | response_type=token&client_id=ID&redirect_uri=CALLBACK | 会重定向到redirect_uri这个地址并携带token参数    |

  response_type参数有两种：一种是`code`，另一种是`token`
  `code模式：`response_type=code也就是授权码模式。返回authCode。
  `token模式：`另一种是 是获取token,该token类似于accessToken,可以根据这个token获取用户信息。H5页面或者app等应用可以使用token模式。



- /oauth2.0/accessToken方法介绍

  | 端点                  | 请求参数                                                     | 响应内容                            |
  | --------------------- | ------------------------------------------------------------ | ----------------------------------- |
  | /oauth2.0/accessToken | grant_type=authorization_code&client_id=ID&client_secret=SECRET &code=CODE&redirect_uri=CALLBACK | 传统模式根据authCode获取accesstoken |
  | /oauth2.0/accessToken | grant_type=password&client_id=ID&client_secret=SECRET &username=USERNAME&password=PASSWORD | 返回accessToken                     |
  | /oauth2.0/accessToken | grant_type=client_credentials&client_id=client&client_secret=secret | 返回accessToken                     |
  | /oauth2.0/accessToken | grant_type=refresh_token&client_id=ID&client_secret=SECRET &refresh_token=REFRESH_TOKEN | 刷新之后返回新的accessToken         |

  grant_type参数有以下几种类型：

  `authorization_code：`
  grant_type=authorization_code使用最多的一种,根据code 获取accessToken。常用于第三方登录,用户输入账号密码,同意并授权,第三方不知道用户的账号和密码,获取code之后,再根据code获取accessToken，使用accessToken获取用户同意获取的信息。

  `password：`
  grant_type=password 允许Oauth客户端直接发送用户的账号和密码到Oauth服务端,常用于web服务,和受信任的客户端(一般是内部服务)。

  `client_credentials：`
  grant_type=client_credentials采用Client Credentials方式，即应用公钥、密钥方式获取Access Token，适用于任何类型应用，但通过它所获取的Access Token只能用于访问与用户无关的Open API，并且需要开发者提前向开放平台申请，成功对接后方能使用。认证服务器不提供像用户数据这样的重要资源，仅仅是有限的只读资源或者一些开放的 API。比如获取App首页最新闻列表,由于这个数据与用户无关，所以不涉及用户登录与授权,但又不想任何人都可以调用这个WebAPI，这样场景就适用。

  `refresh_token：`
  grant_type=refresh_token 上一次的令牌过期时,根据上一次请求返回的refresh_token来刷新accessToken.
  该怎么选择类型呢？参考文档https://alexbilbie.com/guide-to-oauth-2-grants/





## 13、  集群搭建

- redis存入ticket

  - 依赖

    ```xml
    <dependency>
        <groupId>org.apereo.cas</groupId>
        <artifactId>cas-server-support-redis-ticket-registry</artifactId>
        <version>${cas.version}</version>
    </dependency>
    ```

  - 配置

    #配置redis存储ticket

    ```properties
    cas.ticket.registry.redis.host=127.0.0.1
    cas.ticket.registry.redis.database=0
    cas.ticket.registry.redis.port=6379
    cas.ticket.registry.redis.password=wsc123456
    cas.ticket.registry.redis.timeout=2000
    cas.ticket.registry.redis.useSsl=false
    cas.ticket.registry.redis.usePool=true
    cas.ticket.registry.redis.pool.max-active=20
    cas.ticket.registry.redis.pool.maxIdle=8
    cas.ticket.registry.redis.pool.minIdle=0
    cas.ticket.registry.redis.pool.maxActive=8
    cas.ticket.registry.redis.pool.maxWait=-1
    cas.ticket.registry.redis.pool.numTestsPerEvictionRun=0
    cas.ticket.registry.redis.pool.softMinEvictableIdleTimeMillis=0
    cas.ticket.registry.redis.pool.minEvictableIdleTimeMillis=0
    cas.ticket.registry.redis.pool.lifo=true
    cas.ticket.registry.redis.pool.fairness=false
    cas.ticket.registry.redis.pool.testOnCreate=false
    cas.ticket.registry.redis.pool.testOnBorrow=false
    cas.ticket.registry.redis.pool.testOnReturn=false
    cas.ticket.registry.redis.pool.testWhileIdle=false
    #cas.ticket.registry.redis.sentinel.master=mymaster
    #cas.ticket.registry.redis.sentinel.nodes[0]=localhost:26377
    #cas.ticket.registry.redis.sentinel.nodes[1]=localhost:26378
    #cas.ticket.registry.redis.sentinel.nodes[2]=localhost:26379
    ```

- session存入redis

  - 依赖

    ```xml
    <dependency>
        <groupId>org.apereo.cas</groupId>
        <artifactId>cas-server-webapp-session-redis</artifactId>
        <version>${cas.version}</version>
    </dependency>
    ```

    

  - 配置

    ```properties
    #配置redis存储session
    cas.webflow.autoconfigure=true
    cas.webflow.alwaysPauseRedirect=false
    cas.webflow.refresh=true
    cas.webflow.redirectSameState=false
    cas.webflow.session.lockTimeout=30
    cas.webflow.session.compress=false
    cas.webflow.session.maxConversations=5
    cas.webflow.session.storage=true
    spring.session.store-type=redis
    spring.redis.host=127.0.0.1
    spring.redis.password=wsc123456
    spring.redis.port=6379
    ```

    