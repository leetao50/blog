

# SecurityFilterChain配置

Spring Security框架看似比较复杂，但说到底，框架中的各种安全功能，基本上也就是一个个Filter(javax.servlet.Filter)组成的所谓“过滤器链”实现的，这些Filter以职责链的设计模式组织起来，环环相扣。6.x，WebSecurityConfigurerAdapter 被标记为过时,6.x使用SecurityFilterChain。

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
            .authorizeHttpRequests(authorize -> authorize
                    .anyRequest().authenticated()
            )
            .formLogin(withDefaults())
            .httpBasic(withDefaults());
    return http.build();
}
```
概况地说，HttpSecurity的配置过程，主要就是向这个SecurityFilterChian中添加不同功能的Filter对象，为了方便后文理解，首先来看一下其中涉及的几个重要的接口和类（关系如下图）

![图 0](../28f6f23b047fd6cde3c1c730d1736bacbdd95a253bf1eeb9bde7607ccc23b796.png)  

**接口**
+ SecurityBuilder：顶层接口，定义了抽象的泛型构造器方法——build()
+ SecurityConfigurer：顶层接口，用来定义配置类的通用方法，每个Filter都是由特定的SecurityConfigurer的实现类构建出来并添加到FilterChain中的
+ SecurityFilterChain：顶层接口，即过滤器链，定义了获取List<Filter>的方法，以及matches，用于判断某个请求是否满足进链的条件
+ HttpSecurityBuilder：继承SecurityBuilder，定义了构建SecurityFilterChain过程中的各种辅助方法，如添加Filter到SecurityFilterChain，获取SecurityConfigurer配置实现类等

**类**
+ AbstractSecurityBuilder：顶层的抽象父类，它没有实现build具体的逻辑，实际交由doBuild方法实现，只是用CAS对doBuild过程进行了并发控制
+ AbstractConfiguredSecurityBuilder：继承了AbstractSecurityBuilder，它内部维护了一个SecurityConfigurer的列表，实现了doBuild方法，确立了整个构建的流程
+ HttpSecurity 作为final实现类，它主要面向开发者，我们在开发过程中就是用它提供的一系列的配置入口，方便开发者对SecurityFilterChain中不同的Filter进行定制，包括添加自定义的Filter，关闭某些Filter，或扩展原来Filter的能力等等

## 最后做一个简单的总结
流程如图所示：
![图 1](../926bcaa6888a40623a8bf01f8fa76957ce5321b5d900685dc8e156b402dd09dd.png)  

1. 从HttpSercurityConfiguration定义HttpSecurity的Bean对象开始，便向HttpSecurity中添加了若干SecurityConfigurer对象，另外我们可以在自定义的配置类中对其进行一些定制调整
2. 然后当调用HttpSecurity#Build()方法时，就会将取得所有SecurityConfigurer进行遍历，依次调用对应的init和configurer方法，而在configurer方法中，创建出各种功能的Filter实例，并添加到`List<OrderedFilter>`列表中
3. 最后通过performBuild方法，将`List<OrderedFilter>`进行排序，并创建出DefaultSecurityFilterChian

至此整个过滤器链的构建就完成了。

# SecurityFilterChain的工作原理
过滤器Filter是Servlet的标准组件，自Servlet 2.3版本引入，主要作用是在Servlet实例接受到请求之前，以及返回响应之后，这两个方向上进行动态拦截，这样就可以与Servlet主业务逻辑解耦，从而实现灵活性和可扩展性，利用这个特性可以实现很多功能，例如身份认证，统一编码，数据加密解密，审计日志等等。

Filter接口定义了3个方法：doFilter，init和destory，其中doFilter就是请求进入过滤器时需要执行的逻辑，伪代码实现如下

```java
public class ExampleFilter implements Filter {
    …
    public void doFilter(ServletRequest request, ServletResponse response,
                            FilterChain chain) throws IOException, ServletException {
        doSomething();
        chain.doFilter(request,response);    
    }
    …
}
```
其中FilterChain中维护了一个所有已注册的过滤器数组，它组成了真正的“过滤器链”，下面是FilterChain的实现类ApplicationFilterChain的部分源码：当请求到达Servlet容器时，就会创建出一个FilterChain实例，然后调用FilterChain#doFilter方法，这时会从数组中取出下一个过滤器，并调用Filter#doFilter方法，在方法末尾又会将请求继续交由FilterChain处理，如此往复，从而实现职责链模式的调用方式。


