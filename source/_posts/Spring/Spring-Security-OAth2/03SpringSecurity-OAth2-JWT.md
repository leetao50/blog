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

# 客户端模式（Client Credentials）
这是最简单的一种模式，我们可以直接向验证服务器请求一个Token（这里可能有些小伙伴对Token的概念不是很熟悉，Token相当于是一个令牌，我们需要在验证服务器**（User Account And Authentication）**服务拿到令牌之后，才能去访问资源，比如用户信息、借阅信息等，这样资源服务器才能知道我们是谁以及是否成功登录了）
当然，这里的前端页面只是一个例子，它还可以是其他任何类型的客户端，比如App、小程序甚至是第三方应用的服务。
![图 0](../b61bffdb1c0b1422e59512f1c92e0f1bc22e022a1901ed69dd166ed415c76230.png)  

虽然这种模式比较简便，但是已经失去了用户验证的意义，压根就不是给用户校验准备的，而是更适用于服务内部调用的场景。

# 密码模式（Resource Owner Password Credentials）
密码模式相比客户端模式，就多了用户名和密码的信息，用户需要提供对应账号的用户名和密码，才能获取到Token。
![图 1](../005775b0d9deff09cb2190fcb38d1ffbb9ae8e2d2d9c1b2ae0e35a8bed7be7c1.png)  

虽然这样看起来比较合理，但是会直接将账号和密码泄露给客户端，需要后台完全信任客户端不会拿账号密码去干其他坏事，所以这也不是我们常见的。

# 隐式授权模式（Implicit Grant）
首先用户访问页面时，会重定向到认证服务器，接着认证服务器给用户一个认证页面，等待用户授权，用户填写信息完成授权后，认证服务器返回Token。
![图 2](../87d53a374d9b1107f93e6224102d600fa559dd89849715ef4f25f5d3b02e9e2d.png)  

它适用于没有服务端的第三方应用页面，并且相比前面一种形式，验证都是在验证服务器进行的，敏感信息不会轻易泄露，但是Token依然存在泄露的风险。

# 授权码模式（Authrization Code）
这种模式是最安全的一种模式，也是推荐使用的一种，比如我们手机上的很多App都是使用的这种模式。
相比隐式授权模式，它并不会直接返回Token，而是返回授权码，真正的Token是通过应用服务器访问验证服务器获得的。在一开始的时候，应用服务器（客户端通过访问自己的应用服务器来进而访问其他服务）和验证服务器之间会共享一个secret，这个东西没有其他人知道，而验证服务器在用户验证完成之后，会返回一个授权码，应用服务器最后将授权码和secret一起交给验证服务器进行验证，并且Token也是在服务端之间传递，不会直接给到客户端。
![图 3](../acc2bb2eab659990644c0feabc559523a2ddaf2068f36aef1a594eb15209108c.png)  

这样就算有人中途窃取了授权码，也毫无意义，因为，Token的获取必须同时携带授权码和secret，但是secret第三方是无法得知的，并且Token不会直接丢给客户端，大大减少了泄露的风险。


# 搭建验证服务器
第一步就是最重要的，我们需要搭建一个验证服务器，它是我们进行权限校验的核心，验证服务器有很多的第三方实现也有Spring官方提供的实现，这里我们使用Spring官方提供的验证服务器。

## 引入依赖
spring-cloud-starter-oauth2 已经包含了 spring-cloud-starter-security、spring-security-oauth2、spring-security-jwt 这3个依赖，只需引入 spring-cloud-starter-oauth2 即可。

## 修改一下配置文件
```yml
server:
  port: 8500
  servlet:
  	#为了防止一会在服务之间跳转导致Cookie打架（因为所有服务地址都是localhost，都会存JSESSIONID）
  	#这里修改一下context-path，这样保存的Cookie会使用指定的路径，就不会和其他服务打架了
  	#但是注意之后的请求都得在最前面加上这个路径
    context-path: /sso
```
## 编写配置类
接着我们需要编写一下配置类，这里需要两个配置类，一个是OAuth2的配置类，还有一个是SpringSecurity的配置类：

SecurityConfig 对象完整内容
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
 
    @Autowired
    private UserDetailsService userDetailsService;
 
    @Autowired
    private InvalidAuthenticationEntryPoint invalidAuthenticationEntryPoint;
 
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
 
    @Bean
    public JwtTokenOncePerRequestFilter authenticationJwtTokenFilter() {
        return new JwtTokenOncePerRequestFilter();
    }
 
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
                // 禁用basic明文验证
                .httpBasic().disable()
                // 前后端分离架构不需要csrf保护
                .csrf().disable()
                // 禁用默认登录页
                .formLogin().disable()
                // 禁用默认登出页
                .logout().disable()
                // 设置异常的EntryPoint，如果不设置，默认使用Http403ForbiddenEntryPoint
                .exceptionHandling(exceptions -> exceptions.authenticationEntryPoint(invalidAuthenticationEntryPoint))
                // 前后端分离是无状态的，不需要session了，直接禁用。
                .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                .authorizeHttpRequests(authorizeHttpRequests -> authorizeHttpRequests
                        // 允许所有OPTIONS请求
                        .requestMatchers(HttpMethod.OPTIONS, "/**").permitAll()
                        // 允许直接访问授权登录接口
                        .requestMatchers(HttpMethod.POST, "/web/authenticate").permitAll()
                        // 允许 SpringMVC 的默认错误地址匿名访问
                        .requestMatchers("/error").permitAll()
                        // 其他所有接口必须有Authority信息，Authority在登录成功后的UserDetailsImpl对象中默认设置“ROLE_USER”
                        //.requestMatchers("/**").hasAnyAuthority("ROLE_USER")
                        // 允许任意请求被已登录用户访问，不检查Authority
                        .anyRequest().authenticated())
                .authenticationProvider(authenticationProvider())
                // 加我们自定义的过滤器，替代UsernamePasswordAuthenticationFilter
                .addFilterBefore(authenticationJwtTokenFilter(), UsernamePasswordAuthenticationFilter.class);
 
        return http.build();
    }
 
    @Bean
    public UserDetailsService userDetailsService() {
        // 调用 JwtUserDetailService实例执行实际校验
        return username -> userDetailsService.loadUserByUsername(username);
    }
 
    /**
     * 调用loadUserByUsername获得UserDetail信息，在AbstractUserDetailsAuthenticationProvider里执行用户状态检查
     *
     * @return
     */
    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider authProvider = new DaoAuthenticationProvider();
        // DaoAuthenticationProvider 从自定义的 userDetailsService.loadUserByUsername 方法获取UserDetails
        authProvider.setUserDetailsService(userDetailsService());
        // 设置密码编辑器
        authProvider.setPasswordEncoder(passwordEncoder());
        return authProvider;
    }
 
    /**
     * 登录时需要调用AuthenticationManager.authenticate执行一次校验
     *
     * @param config
     * @return
     * @throws Exception
     */
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }
}
```
OAuth2Configuration 对象完整内容

```java
@EnableAuthorizationServer   //开启验证服务器
@Configuration
public class OAuth2Configuration extends AuthorizationServerConfigurerAdapter {

    @Resource
    private AuthenticationManager manager;

    private final BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();

    /**
     * 这个方法是对客户端进行配置，一个验证服务器可以预设很多个客户端，
     * 之后这些指定的客户端就可以按照下面指定的方式进行验证
     * @param clients 客户端配置工具
     */
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients
                .inMemory()   //这里我们直接硬编码创建，当然也可以像Security那样自定义或是使用JDBC从数据库读取
                .withClient("web")   //客户端名称，随便起就行
                .secret(encoder.encode("654321"))      //只与客户端分享的secret，随便写，但是注意要加密
                .autoApprove(false)    //自动审批，这里关闭，要的就是一会体验那种感觉
                .scopes("book", "user", "borrow")     //授权范围，这里我们使用全部all
                .authorizedGrantTypes("client_credentials", "password", "implicit", "authorization_code", "refresh_token");
                //授权模式，一共支持5种，除了之前我们介绍的四种之外，还有一个刷新Token的模式
                //这里我们直接把五种都写上，方便一会实验，当然各位也可以单独只写一种一个一个进行测试
                //现在我们指定的客户端就支持这五种类型的授权方式了
    }

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) {
        security
                .passwordEncoder(encoder)    //编码器设定为BCryptPasswordEncoder
                .allowFormAuthenticationForClients()  //允许客户端使用表单验证，一会我们POST请求中会携带表单信息
                .checkTokenAccess("permitAll()");     //允许所有的Token查询请求
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
        endpoints
                .authenticationManager(manager);
        //由于SpringSecurity新版本的一些底层改动，这里需要配置一下authenticationManager，才能正常使用password模式
    }
}
```


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
