---
title: SpringSecurity+OAth2+JWT
date: 2024-05-06 16:00:17
tags:
---

微服务中使用Spring Security + OAuth 2.0 + JWT 搭建认证授权服务

OAuth 是一种用来规范令牌（Token）发放的授权机制，主要包含了四种授权模式：
1. 授权码模式
2. 简化模式
3. 密码模式
4. 客户端模式

# 引入依赖

spring-cloud-starter-oauth2 已经包含了 spring-cloud-starter-security、spring-security-oauth2、spring-security-jwt 这3个依赖，只需引入 spring-cloud-starter-oauth2 即可。

# 准备工作

1. 新建 UserDTO 类，实现 org.springframework.security.core.userdetails.UserDetails 接口

nec-common:Account

```java
@Data
public class UserDTO implements Serializable, UserDetails {
    private static final long serialVersionUID = 5538522337801286424L;

    private String userName;
    private String password;
    private Set<SimpleGrantedAuthority> authorities;

    public Collection<? extends GrantedAuthority> getAuthorities() {
        return this.authorities;
    }

    public String getPassword() {
        return this.password;
    }

    public String getUsername() {
        return this.userName;
    }

    public boolean isAccountNonExpired() {
        return true;
    }

    public boolean isAccountNonLocked() {
        return true;
    }

    public boolean isCredentialsNonExpired() {
        return true;
    }

    public boolean isEnabled() {
        return true;
    }
}
```

2. 新建类 UserDetailsServiceImpl，实现 org.springframework.security.core.userdetails.UserDetailsService 接口，用于校验用户凭据。

nec-auth:NecUserDetailService


```java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    private PasswordEncoder passwordEncoder;

    @Autowired
    public void setPasswordEncoder(PasswordEncoder passwordEncoder) {
        this.passwordEncoder = passwordEncoder;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // TODO 实际开发中，这里请修改从数据库中查询...
        UserDTO user = new UserDTO();
        user.setUserName(username);
        // 密码为 123456 ，且加密
        user.setPassword(passwordEncoder.encode("123456"));
        return user;
    }
}
```

# 配置认证授权服务器

1. 新建类 Oauth2ServerConfig，继承 org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter 类；在 Oauth2ServerConfig 类上 添加注解 @EnableAuthorizationServer 。

nec-auth:AuthorizationServer

框架提供了几个默认的端点：

+ /oauth/authorize：授权端点
+ /oauth/token：获取令牌端点
+ /oauth/confirm_access：用户确认授权端点
+ /oauth/check_token：校验令牌端点
+ /oauth/error：用于在授权服务器中呈现错误
+ /oauth/token_key：获取 jwt 公钥端点

2. 继承 AuthorizationServerConfigurerAdapter 类后，我们需要重写以下三个方法扩展实现我们的需求。

+ configure(ClientDetailsServiceConfigurer clients) ：用于定义、初始化客户端信息
+ configure(AuthorizationServerEndpointsConfigurer endpoints)：用于定义授权令牌端点及服务
+ configure(AuthorizationServerSecurityConfigurer security)：用于定义令牌端点的安全约束

## 配置客户端详细信息 ClientDetailsServiceConfigurer

用于定义 内存 中或 基于JDBC存储实现 的客户端，其重要的几个属性有：

1. clientId：客户端id，必填；
2. clientSecret：客户端密钥；
3. authorizedGrantTypes：客户端授权类型，有 5 种模式： authorization_code、password、client_credentials、implicit、refresh_token；
4. scope：授权范围；
5. accessTokenValiditySeconds：access_token 有效时间，单位为秒，默认为 12 小时；
6. refreshTokenValiditySeconds：refresh_token 有效时间，单位为秒，默认为 30 天；

客户端信息一般保存在 Redis 或 数据库中

1. 使用以下 SQL（适用于MySQL） 来建表：
```SQL
CREATE TABLE `oauth_client_details`  (
  `client_id` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `resource_ids` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `client_secret` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `scope` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `authorized_grant_types` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `web_server_redirect_uri` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `authorities` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `access_token_validity` int(11) NULL DEFAULT NULL,
  `refresh_token_validity` int(11) NULL DEFAULT NULL,
  `additional_information` varchar(4096) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `autoapprove` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  PRIMARY KEY (`client_id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```

2. 添加一条客户端信息用于测试：
```SQL
INSERT INTO `oauth_client_details` VALUES ('auth-server', NULL, '$2a$10$mcEwJ8qqhk2DYIle6VfhEOZHRdDbCSizAQbIwBR7tTuv9Q7Fca9Gi', 'all', 'password,refresh_token', '', NULL, NULL, NULL, NULL, NULL);

```
其中密码 123456 使用 BCryptPasswordEncoder 加密，加密后字符为 $2a$10$mcEwJ8qqhk2DYIle6VfhEOZHRdDbCSizAQbIwBR7tTuv9Q7Fca9Gi。

3. 配置 ClientDetailsServiceConfigurer ，指定客户端信息：

```java
@Configuration
@EnableAuthorizationServer
public class Oauth2ServerConfig extends AuthorizationServerConfigurerAdapter {

    private final DataSource dataSource;
    
    private final PasswordEncoder passwordEncoder;

    @Autowired
    public Oauth2ServerConfig(DataSource dataSource, PasswordEncoder passwordEncoder) {
        this.dataSource = dataSource;
        this.passwordEncoder = passwordEncoder;
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        // 使用基于 JDBC 存储模式
        JdbcClientDetailsService clientDetailsService = new JdbcClientDetailsService(dataSource);
        // client_secret 加密
        clientDetailsService.setPasswordEncoder(passwordEncoder);
        clients.withClientDetails(clientDetailsService);
    }
}
```

## 配置授权令牌端点及服务 AuthorizationServerEndpointsConfigurer
需要指定 AuthenticationManager 及 UserDetailService，尤其是使用密码模式时，必须指定 AuthenticationManager，否则会报 Unsupported grant type: password 错误。

新建 WebSecurityConfig 类，继承 org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter 类，重写 authenticationManagerBean() 方法，并定义需要用到的 PasswordEncoder；

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                // 支持跨域请求
                .cors()

                .and()
                // 禁用 CSRF
                .csrf().disable()

                .formLogin().disable()
                .httpBasic().disable()
                .logout().disable()

                .authorizeRequests()
                .antMatchers("/oauth/token").permitAll();

                .anyRequest().authenticated();
    }

    /**
     * 重写 authenticationManagerBean()
     * @return
     * @throws Exception
     */
    @Override
    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

2. 配置 AuthorizationServerEndpointsConfigurer：

```java
@Configuration
@EnableAuthorizationServer
public class Oauth2ServerConfig extends AuthorizationServerConfigurerAdapter {

    private final UserDetailsServiceImpl userDetailsService;

    /**
     * 密码模式 grant_type:password 需指定 AuthenticationManager
     */
    private final AuthenticationManager authenticationManager;


    @Autowired
    public Oauth2ServerConfig(UserDetailsServiceImpl userDetailsService,
                              AuthenticationManager authenticationManager) {
        this.userDetailsService = userDetailsService;
        this.authenticationManager = authenticationManager;
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints
                // 开启密码模式授权
                .authenticationManager(authenticationManager)
                .userDetailsService(userDetailsService);
    }
}
```

使用 JWT 作为令牌格式
生成 JWT 密钥对
使用 JDK 的 keytool 工具生成 JKS 密钥对 jwt.jks，并将 jwt.jks 放到 resources 目录下。

定位至 JDK 目录下的 bin 目录，执行以下命令生成密钥对，记住口令密钥，代码中需要用到密钥来读取密钥对，以下命令以 123456 为例：

keytool -genkey -alias weihong -keyalg RSA -keypass 123456 -keystore jwt.jks -storepass 123456

```console
-genkey 生成密钥

-alias 别名

-keyalg 密钥算法

-keypass 密钥口令

-keystore 生成密钥对的存储路径和名称

-storepass 密钥对口令
```
定义 token 转换器
在 Oauth2ServerConfig 类中定义 accessTokenConverter() 及 keyPair()：
指定令牌存储策略为 JWT
配置 AuthorizationServerEndpointsConfigurer 的令牌存储策略为 JWT，指定 accessTokenConverter 为我们定义好的 accessTokenConverter()：

```java
@Configuration
@EnableAuthorizationServer
public class Oauth2ServerConfig extends AuthorizationServerConfigurerAdapter {

    private final UserDetailsServiceImpl userDetailsService;

    /**
     * 密码模式 grant_type:password 需指定 AuthenticationManager
     */
    private final AuthenticationManager authenticationManager;

    @Autowired
    public Oauth2ServerConfig(UserDetailsServiceImpl userDetailsService,
                              AuthenticationManager authenticationManager) {
        this.userDetailsService = userDetailsService;
        this.authenticationManager = authenticationManager;
    }


    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
        endpoints
                // 开启密码模式授权
                .authenticationManager(authenticationManager)
                .userDetailsService(userDetailsService)
                // 指定令牌存储策略
                .accessTokenConverter(accessTokenConverter());
    }

    /**
     * token 转换器
     * 默认是 uuid 格式，我们在这里指定 token 格式为 jwt
     * @return
     */
    @Bean
    public JwtAccessTokenConverter accessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        // 使用非对称加密算法对 token 签名
        converter.setKeyPair(keyPair());
        return converter;
    }

    @Bean
    public KeyPair keyPair() {
        // 从 classpath 目录下的证书 jwt.jks 中获取秘钥对
        KeyStoreKeyFactory keyStoreKeyFactory = new KeyStoreKeyFactory(new ClassPathResource("jwt.jks"), "123456".toCharArray());
        return keyStoreKeyFactory.getKeyPair("weihong", "123456".toCharArray());
    }

}    
```

扩展 JWT 存储内容

有时候我们需要扩展 JWT 存储的内容，比如存储一些用户数据、权限信息等。我们可以定义 TokenEnhancer 或继承 TokenEnhancer 来实现 JWT 内容增强器：

```java
@Configuration
@EnableAuthorizationServer
public class Oauth2ServerConfig extends AuthorizationServerConfigurerAdapter {

    private final UserDetailsServiceImpl userDetailsService;

    private final AuthenticationManager authenticationManager;

    @Autowired
    public Oauth2ServerConfig(UserDetailsServiceImpl userDetailsService,
                              AuthenticationManager authenticationManager) {
        this.userDetailsService = userDetailsService;
        this.authenticationManager = authenticationManager;
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        TokenEnhancerChain enhancerChain = new TokenEnhancerChain();
        List<TokenEnhancer> delegates = new ArrayList<>();

        delegates.add(tokenEnhancer());
        delegates.add(accessTokenConverter());

        // 配置 JWT 内容增强
        enhancerChain.setTokenEnhancers(delegates);

        endpoints
                // 开启密码模式授权
                .authenticationManager(authenticationManager)
                .userDetailsService(userDetailsService)
                .accessTokenConverter(accessTokenConverter())
                .tokenEnhancer(enhancerChain);
    }
    
    /**
     * token 转换器
     * 默认是 uuid 格式，我们在这里指定 token 格式为 jwt
     * @return
     */
    @Bean
    public JwtAccessTokenConverter accessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        // 使用非对称加密算法对 token 签名
        converter.setKeyPair(keyPair());
        return converter;
    }

    @Bean
    public KeyPair keyPair() {
        KeyStoreKeyFactory keyStoreKeyFactory = new KeyStoreKeyFactory(new ClassPathResource("jwt.jks"), "123456".toCharArray());
        return keyStoreKeyFactory.getKeyPair("weihong", "123456".toCharArray());
    }
    
    /**
     * JWT 内容增强器，用于扩展 JWT 内容，可以保存用户数据
     * @return
     */
    @Bean
    public TokenEnhancer tokenEnhancer() {
        return (oAuth2AccessToken, oAuth2Authentication) -> {
            Map<String, Object> map = new HashMap<>(1);
            UserDTO userDTO = (UserDTO) oAuth2Authentication.getPrincipal();
            map.put("userName", userDTO.getUsername());
            // TODO 其他信息可以自行添加
            ((DefaultOAuth2AccessToken) oAuth2AccessToken).setAdditionalInformation(map);
            return oAuth2AccessToken;
        };
    }
}
```

使用 Redis 存储 token

添加 token 保存至 redis 的配置:

```java
@Configuration
public class RedisTokenStoreConfig {

    @Resource
    private RedisConnectionFactory connectionFactory;

    @Bean
    public TokenStore redisTokenStore() {
        return new RedisTokenStore(connectionFactory);
    }
}
```
在认证服务配置中指定 token 存储方式：
```java
@Configuration
@EnableAuthorizationServer
public class Oauth2ServerConfig extends AuthorizationServerConfigurerAdapter {

    private final UserDetailsServiceImpl userDetailsService;

    /**
     * 密码模式 grant_type:password 需指定 AuthenticationManager
     */
    private final AuthenticationManager authenticationManager;
    
    private final TokenStore tokenStore;

    @Autowired
    public Oauth2ServerConfig(UserDetailsServiceImpl userDetailsService,
                              AuthenticationManager authenticationManager,
                              @Qualifier("redisTokenStore") TokenStore tokenStore) {
        this.userDetailsService = userDetailsService;
        this.authenticationManager = authenticationManager;
        this.tokenStore = tokenStore;
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints
                // 开启密码模式授权
                .authenticationManager(authenticationManager)
                .userDetailsService(userDetailsService)
                // 设置 token 存储方式
                .tokenStore(tokenStore);
    }
}
```

## 配置授权令牌安全约束 AuthorizationServerSecurityConfigurer

```java
@Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security
                // 允许表单认证
                .allowFormAuthenticationForClients()
                // 开放 /oauth/token_key 获取 token 加密公钥
                .tokenKeyAccess("permitAll()")
                // 开放 /oauth/check_token
                .checkTokenAccess("permitAll()");
    }
```
