---
title: @EnableMethodSecurity注解
date: 2024-05-07 18:06:09
tags:
---

Spring Security支持方法级别的权限控制。我们可以在任意层的任意方法上加入权限注解，加入注解的方法将自动被Spring Security保护起来，仅仅允许特定的用户访问，从而还到权限控制的目的，当然如果现有的权限注解不满足我们也可以自定义。

Spring Security默认是禁用注解的，要想开启注解，要在继承WebSecurityConfigurerAdapter的类加@EnableMethodSecurity注解，并在该类中将AuthenticationManager定义为Bean。

```java
@EnableWebSecurity
@Configuration
@EnableGlobalMethodSecurity(
  prePostEnabled = true, 
  securedEnabled = true, 
  jsr250Enabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
}
```
我们看到@EnableGlobalMethodSecurity 分别有prePostEnabled 、securedEnabled、jsr250Enabled三个字段，其中每个字段代码一种注解支持，默认为false，true为开启。那么我们就一一来说一下这三总注解支持。

+ prePostEnabled = true 的作用的是启用Spring Security的@PreAuthorize 以及@PostAuthorize 注解。

+ securedEnabled = true 的作用是启用Spring Security的@Secured 注解。

+ jsr250Enabled = true 的作用是启用@RoleAllowed 注解
  
# 在方法上设置权限认证

## @RolesAllowed

```java
@RolesAllowed("ROLE_VIEWER")
public String getUsername2() {
    //...
}
     
@RolesAllowed({ "USER", "ADMIN" })
public boolean isValidUsername2(String username) {
    //...
}
```

代表标注的方法只要具有USER, ADMIN任意一种权限就可以访问。这里可以省略前缀ROLE_，实际的权限可能是ROLE_ADMIN

在功能及使用方法上与 @Secured 完全相同

## securedEnabled注解

```java
@Secured("ROLE_VIEWER")
public String getUsername() {
    SecurityContext securityContext = SecurityContextHolder.getContext();
    return securityContext.getAuthentication().getName();
}

@Secured({ "ROLE_DBA", "ROLE_ADMIN" })
public String getUsername2() {
    //...
}
```
@Secured 规定了访问访方法的角色列表，在列表中最少指定一种角色。@Secured在方法上指定安全性，要求 角色/权限等 只有对应 角色/权限 的用户才可以调用这些方法。

> 还有一点就是@Secured,不支持Spring EL表达式

## prePostEnabled注解

> 这个开启后支持Spring EL表达式。如果没有访问方法的权限，会抛出AccessDeniedException。

@PreAuthorize注解：
进入方法之前验证授权。

```java
@PreAuthorize("hasRole('ROLE_VIEWER')")
public String getUsernameInUpperCase() {
    return getUsername().toUpperCase();
}
```
@PreAuthorize("hasRole('ROLE_VIEWER')") 相当于@Secured(“ROLE_VIEWER”)。

同样的@Secured({“ROLE_VIEWER”,”ROLE_EDITOR”}) 也可以替换为：@PreAuthorize(“hasRole(‘ROLE_VIEWER') or hasRole(‘ROLE_EDITOR')”)。

在方法执行之前执行，这里可以调用方法的参数，也可以得到参数值，这里利用JAVA8的参数名反射特性，如果没有JAVA8，那么也可以利用Spring Secuirty的@P标注参数，或利用Spring Data的@Param标注参数。

```java
//无java8
@PreAuthorize("#userId == authentication.principal.userId or hasAuthority(‘ADMIN’)")
void changePassword(@P("userId") long userId ){}
//有java8
@PreAuthorize("#userId == authentication.principal.userId or hasAuthority(‘ADMIN’)")
void changePassword(long userId ){}

```
这里表示在changePassword方法执行之前，判断方法参数userId的值是否等于principal中保存的当前用户的userId，或者当前用户是否具有ROLE_ADMIN权限，两种符合其一，就可以访问该 方法。

@PostAuthorize注解：
在方法执行之后执行,以获取到方法的返回值，并且可以根据该方法来决定最终的授权结果（是允许访问还是不允许访问)

```java
@PostAuthorize
  ("returnObject.username == authentication.principal.nickName")
public CustomUser loadUserDetail(String username) {
    return userRoleRepository.loadUserByUserName(username);
}
```
上述代码中，仅当loadUserDetail方法的返回值中的username与当前登录用户的username相同时才被允许访问

> 注意如果EL为false，那么该方法也已经执行完了，可能会回滚。EL变量returnObject表示返回的对象。

@PreFilter注解：
在方法执行之前执行，而且这里可以调用方法的参数，然后对参数值进行过滤或处理或修改。
```java
@PreFilter("filterObject != authentication.principal.username")
public String joinUsernames(List<String> usernames) {
    return usernames.stream().collect(Collectors.joining(";"));
}
```

当usernames中的子项与当前登录用户的用户名不同时，则保留；当usernames中的子项与当前登录用户的用户名相同时，则移除。比如当前使用用户的用户名为zhangsan，此时usernames的值为{"zhangsan", "lisi", "wangwu"}，则经@PreFilter过滤后，实际传入的usernames的值为{"lisi", "wangwu"}

如果执行方法中包含有多个类型为Collection的参数，filterObject 就不太清楚是对哪个Collection参数进行过滤了。此时，便需要加入 filterTarget 属性来指定具体的参数名称：

```java
@PreFilter(value = "filterObject != authentication.principal.username",
  filterTarget = "usernames")
public String joinUsernamesAndRoles(List<String> usernames, List<String> roles) {
  
    return usernames.stream().collect(Collectors.joining(";")) 
      + ":" + roles.stream().collect(Collectors.joining(";"));
}
```

@PostFilter：
在方法执行之后执行，而且这里可以调用方法的返回值，然后对返回值进行过滤或处理或修改并返回

```java
@PostFilter("filterObject != authentication.principal.username")
public List<String> getAllUsernamesExceptCurrent() {
    return userRoleRepository.getAllUsernames();
}
```
此时 filterObject 代表返回值。如果按照上述代码则实现了：移除掉返回值中与当前登录用户的用户名相同的子项。

# 自定义元注解
如果我们需要在多个方法中使用相同的安全注解，则可以通过创建元注解的方式来提升项目的可维护性。

比如创建以下元注解：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@PreAuthorize("hasRole('ROLE_VIEWER')")
public @interface IsViewer {
}
```
然后可以直接将该注解添加到对应的方法上：

```java
@IsViewer
public String getUsername4() {
    //...
}
```
在生产项目中，由于元注解分离了业务逻辑与安全框架，所以使用元注解是一个非常不错的选择。

# 类上使用安全注解
如果一个类中的所有的方法我们全部都是应用的同一个安全注解，那么此时则应该把安全注解提升到类的级别上：

```java
@Service
@PreAuthorize("hasRole('ROLE_ADMIN')")
public class SystemService {
  
    public String getSystemYear(){
        //...
    }
  
    public String getSystemDate(){
        //...
    }
}
```

上述代码实现了：访问getSystemYear 以及getSystemDate 方法均需要ROLE_ADMIN权限。

# 总结
默认情况下，在方法中使用安全注解是由Spring AOP代理实现的，这意味着：如果我们在方法1中去调用同类中的使用安全注解的方法2，则方法2上的安全注解将失效。
Spring Security上下文是线程绑定的，这意味着：安全上下文将不会传递给子线程。

```java
public boolean isValidUsername4(String username) {
    // 以下的方法将会跳过安全认证
    this.getUsername();
    return true;
}
```
