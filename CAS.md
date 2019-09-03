# Cas Demo学习

环境 tomcat8  overlays

将cas-server-webapp代码通过maven overlays插件进行修改打包部署至tomcat

https://github.com/Clever-Wang/spring-boot-cas

使用jdk自带的keytool 生成密钥仓 生成用户 设置密码

keytool -genkey -alias tomcat -keyalg RSA -validity 365 -keystore C:\Users\NPL\Desktop\tomcat.keystore



- cas5.3.2单点登录-骨架搭建(一)
  - 指定ssl信息 使用cas进行指定用户名密码登录认证
  - cas.authn.accept.users=casuser::Mellon



- cas5.3.2单点登录-JDBC认证(密码MD5和密码加盐)(二)
  - 添加jdbc认证
  - 设置加密策略
  - 设置盐值



- cas5.3.2单点登录-自定义密码认证(三)
  - 配置自己的加密 账户密码匹配策略
  - 实现自己的PasswordEncoder
  - 在配置文件中指定为自己的PasswordEncoder



- cas5.3.2单点登录-自定义登录验证(四)
  - 实现自己的注册验证器
  - 实现自定义验证器
  - 指定指定用户名通过认证



- cas5.3.2单点登录-服务端集成shiro权限认证(自定义方式)(五)
  - 集成shiro 权限管理