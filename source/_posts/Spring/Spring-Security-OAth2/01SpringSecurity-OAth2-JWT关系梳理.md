
# Security的职责

+ Spring Security是Spring提供的一个安全框架，提供认证和授权功能，最主要的是它提供了简单的使用方式，同时又有很高的灵活性，简单，灵活，强大。
+ Security本身是一套完整的认证和授权解决方案，只是我们在做系统的时候遵循了OAuth2.0规范，引入了其实现。现在流行的前后端分离，服务端不需要保持session管理，只需要验证token合法性。使用jwt生成token，是为了直接通过解密token直接获取用户信息，简化标准OAuth2.0的操作流程，也是当前服务端架构设计的实际需要。
+ SpringSecurity 采用的是责任链的设计模式，它有一条很长的过滤器链。在过滤器链中定义OAuth2.0的或者Jwt的具体实现。

## Spring Security 的核心功能主要包括
+ Authentication：用户认证，一般是通过用户名密码来确认用户是否为系统中合法主体。通过注解`@EnableWebSecurity`开启。认证可以单独使用，即不划分资源的级别，所有人只要登录都可以查看。

+ Authorization：用户授权，一般是给系统中合法主体授予相关资源访问权限；授权的目的是可以把资源进行划分，例如公司有不同的资料，有普通级别和机密级别，只有公司高层才能看到机密级别的子类，而普通级别的资料大家都可以看到！那么授权就是允许你查看某个资源，当然，如果你没有权限，就拒绝你查看！

授权有个前提就是先认证才能授权，例如你是A公司的人，有A公司的卡牌，才能进入A公司资料库。如果是B公司的人，大门都不给你进。

## Spring Security 的基本原理
Spring Security的核心就是一组过滤器链，项目启动后将会自动配置。最核心的就是 Basic Authentication Filter 用来认证用户的身份，一个在Spring Security中一种过滤器处理一种认证方式。如下图所示，这是一组链式处理器类，请求从做往右依次经过多个过滤器类处理：
![图 0](../f5c75221d723306391d70ec29f05cbe87d3b0088b5632dd043b32c046cb6767b.png)  

比如，对于Username Password Authentication Filter过滤器来说：

1. 会检查是否是一个登录请求,是否包含username 和 password;如果不满足则放行给下一个。
2. 下一个按照自身职责判定是否是自身需要的信息，Basic的特征就是在请求头中有 Authorization:Basic eHh4Onh4 的信息
3. 中间可能还有更多的认证过滤器。
4. 最后一环是 FilterSecurityInterceptor，这里会判定该请求是否能进行访问Rest API服务，判断的依据是 BrowserSecurityConfig中的配置，如果被拒绝了就会抛出不同的异常（根据具体的原因）。
5. Exception Translation Filter 会捕获抛出的错误，然后根据不同的认证方式进行信息的返回提示。

> 注意：绿色的过滤器可以配置是否生效，其他的都不能控制。

# 什么是 OAuth2
OAuth2是一种协议，核心是用Token令牌替换直接输入用户名和密码的方式，通过该协议，可以实现跨服务至之间的授权功能。一般分为2个模块，认证授权中心和资源中心，资源中心可以有多个。光有协议也没用，你的有具体的实现，那么Spring Security OAuth2是什么东西呢？答案就是 Spring Security框架内置了OAuth2的API，可以直接使用。当然，其他框架也有OAuth2的实现，可以根据需求选择。

+ 授权中心：负责颁发Token，配置`@EnableAuthorizationServer`注解，表示开启授权中心。

+ 资源中心：负责检查Token(可以自己检查，例如jwt本地检查 或委托授权中心检查)，检查通过后发放资源。配置`@EnableResourceServer`注解，表示开启资源中心功能。

## 基础原理是这样的：

1. 认证授权中心颁发Token，就是让用户输入用户名和密码，检查下是否正确，然后返回一个Token，说白了就是一个字符串，并存在授权中心的服务器上。就像你买的饭票一样，你用现金买了饭票，将来凭借饭票就可以领取一份盒饭！不需要再付钱了

2. 向资源中心发起一个资源的查询，资源中心检查Token，检查通过后，发放资源。

对于大家而言，我们在互联网应用中最常见的 OAuth2 应该就是各种第三方登录了。利用OAuth2协议，我们在注册csdn账户的时候，可以直接使用微信或QQ账户进行注册，这样，仅需登陆微信或QQ，即可让csdn获取到用户的昵称、爱好、邮箱等基础信息。

关键点在于微信颁发的Token，可以给csdn，csdn带着Token去微信获取资源，这个步骤是在csdn后台完成的，对用户来说是隐藏的。也就是说csdn拿了饭票，去微信领盒饭了，不需要付钱(输入密码)，爽！

## endpoint概念
授权中心内置很多url资源，表示用于token特定的功能，这些url是免密使用的，例如/oauth/token。
![图 2](../e96065909c03ca5ee9df313790016c0c20c4c4bd3b7b4c5d892a3b77233e6eec.png)  


## 那么如何实现检查Token
答案在于`@EnableResourceServer`注解，表示开启资源中心功能，会有代理类负责登陆和授权的检查.

如果某个服务的入口类被 `@EnableResourceServer`注解，说明现在是基于oauth2协议的架构，那么原来的Spring Security配置会被覆盖：
![图 1](../90ab9ad8a5dd1266c1e27180df4e7794a92a4872dd679f6b5b5053b903f261b5.png)  

一般的Spring Security要求所有的请求url都要先判断是否登录，如果没有登录，就跳转至登陆页，然后检查用户名和密码是否正确，但是`@EnableResourceServer`资源中心注解会内置有更高优先级的拦截器，会修改这个默认的逻辑，不是通过用户名和密码来检查是否正确，而是通过检查消息头中的Authorization:Bearer xxx参数。

开启资源中心，所有资源优先用Token方式进行检查，即检查消息头中是否含有 Authorization:Bearer xxx 这样格式的；
如果没有token，直接判定失败；

如果有了token，那么如何验证？可以本地验证，或转发token至授权中心进行判断，授权中心颁发token后，会把token存储在内存中，这样当ABC服务获得token后，转发至授权中心，和内存中存储的原始值进行比较就行了。



# 是什么 Jwt
+ JSON Web Token // 是一种具体的Token实现框架
+ 是基于token的认证协议的实现
+ 主要用来生成token，验证token合法性，是否过期，获取用户信息
+ token中包含用户信息

# 总结
+ security是基础，OAuth和Jwt是具体实现，在security上生效
+ OAuth2.0是规范
+ jwt是token的实现
+ 如果我们的系统要给第三方做授权，就实现OAuth2.0
+ 如果我们要做前后端分离，就实现token就可以了，jwt仅仅是token的一种实现方式

https://blog.csdn.net/u014494148/article/details/109260840