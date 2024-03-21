---
title: SpringBean注解
date: 2024-03-20 18:16:11
tags:
---

# @Bean

Bean可以说是Spring当中最为重要的概念之一了。简单来说，Bean就是一个对象，只不过这个对象是由Spring容器来初始化，装配，管理的，因此也可以叫做Spring Bean。
```java

@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Bean {
    @AliasFor("name")
    String[] value() default {};

    @AliasFor("value")
    String[] name() default {};

    /** @deprecated */
    @Deprecated
    Autowire autowire() default Autowire.NO;

    boolean autowireCandidate() default true;

    String initMethod() default "";

    String destroyMethod() default "(inferred)";
}
```
> 注：当注解中只传入value一个属性的时候，value属性的名称可以省略；即：@Bean(value="myBean")等同于@Bean("myBean")；Alias是别名的意思，因此使用@Bean(name="myBean")也是一样的效果。


```java
@Configuration
public class AppConfig {
    @Bean
    public FooService fooService() {
        return new FooService(fooRepository());
    }

    @Bean
    public FooRepository fooRepository() {
        return new JdbcFooRepository(datasource());
    }
}
```
在上面的代码中，我们向Spring容器中注入了两个Bean，当没有显式命名时，自动注册的名称为方法名。

使用name属性显式命名如下：

```java
@Bean({"b1", "b2"}) // Bean可以用'b1'或者'b2'取到，而非'myBean'
public MyBean myBean() {
    // 此处初始化和配置MyBean对象
    return obj;
}  
```

# @Configuration

@Bean注解往往和@Configuration配合，前者标注在方法上，后者标注在类上；两者搭配，将Bean注册到容器中。
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";

    boolean proxyBeanMethods() default true;
}
```
> 我们可以从源码中可以看到，它实质上也是一个@Component注解，所以该注解标记的类本身也会被注册成一个Bean
```java
@Configuration
public class AppConfig {
    @Bean
    public MyBean myBean() {
        // instance, configure and return bean
    }
}
```
它指示类生成一个或者多个@Bean方法，可以由Spring容器处理，在运行时为这些Bean生成BeanDefination和服务请求。

# @Component
从前面我们知道，所谓的Bean其实就是一个个对象；但是@Bean注解是标注在方法上的，意思就是通过方法返回一个对象，那么有没有直接通过类获取对象的呢？当然有，那就是@Component，被该注解标注的类会被注册到当前容器，bean的id就是类名转换为小驼峰。
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Indexed
public @interface Component {
    String value() default "";
}
```
直接标注在类上，Spring扫描时会将其加入容器。
```java
@Component // or @Component("myBean")
public class MyClass {
    // write your bean here...
}
```
## Spring常用注册Bean注解：

+ @Component, @Service, @Repository, @Controller四个注解作用于类上，实质上是一样的，注册类到当前容器，value属性就是BeanName
+ @Configuration注解也作用于类上，该注解通常与@Bean配合使用
+ @Bean用于方法上，该方法需要在@Configutation标注的类里面，且方法必须为public
+ @Component注解的效果与@Bean的效果类似，也是单例模式。可以搭配的注解也类似，例如@Scope, @Profile, @Primary等等。

# @Autowired
我们使用@Bean(或者@Component)注解将Bean注册到了Spring容器；我们创建这些Bean的目的，最终还是为了使用，@Autowired注解可以将bean自动注入到类的属性中。@Autowired注解可以直接标注在属性上，也可以标注在构造器，方法，甚至是传入参数上，其实质都是调用了setter方法注入。
```java
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired {
    boolean required() default true;
}
```
> 当自动注入的Bean不存在时，Spring会报错；如果希望Bean存在时才注入，可以使用@Autowired(required=false)。

## 使用
```java
...
@Autowired //标注在属性上
private MyBean myBean;
...

@Autowired //标注在方法上
public void setMyBean(MyBean myBean) {
    this.myBean = myBean;
}
...

@Autowired//标注在构造函数上
public MyClass(MyBean myBean) {
    this.myBean = myBean;
}
...
//标注在方法参数上
public void setMyBean(@Autowired MyBean myBean) {
    this.myBean = myBean;
}
...
```

+ @Autowired和@Resource注解都是作为bean对象注入时使用的，@Autowired是Spring提供的注解，而@Resource是J2EE本身提供的。
+ @Autowired首先根据类型去寻找注入的对象，如果有多个再根据名字匹配。
+ 当名字也无法区分时可以通过@Qulifier显式指定，如：
```java
@Autowired
@Qualifier("userServiceImpl1")
private UserService userService;
```

# @Qualifier
当同一个类型的Bean创建了多个时，我们可以通过搭配@Autowired和@Qualifier来确定需要注入的Bean解决混淆。除此以外，@Qualifier还可以标注在Bean上实现逻辑分组。
```java
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Qualifier {
    String value() default "";
}
```

标注在单个属性上，是根据name去获取Bean:
```java
@Autowired
@Qualifier("bean2")
private MyBean mybean;

@Bean
public MyBean bean1() {
    return new MyBean();
}

@Bean
public MyBean bean2() {
    return new MyBean();
}
```
标注在集合上，则可以筛选出被@Qualifier标注的Bean
```java
@Autowired
private List<User> users; // user1, user2, user3

@Autowired
@Qualifier
private List<User> usersQualifier; // user2, user3

@Bean
public User user1() {
    return new User();
}

@Bean 
@Qualifier
public User user2() {
    return new User();
}

@Bean
@Qualifier
public User user3() {
    return new User();
}
```

# @Primary
当一个接口有多个实现时，我们可以通过给@Autowired注解搭配@Qualifier来注入我们想要的Bean。这里还有另一种情况：Bean之前分优先级顺序，一般情况下我们只会注入默认实现；这个时候可以采用@Primary注解，该注解标注于Bean上，指示了优先注入的类。
```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Primary {
}
```
使用时直接标注在返回Bean的方法上
```java
...
@Autowired
private MyBean myBean(); // 注入myBean1

@Primary
@Bean
public MyBean myBean1() {
    return new MyBean();
}

@Bean
public MyBean myBean2() {
    return new MyBean();
}
...
```
也可以标注在@Component的类上(@Controller, @Service, @Repository也是一样的)
```java
@Primary
@Component
public class MyBean {
    //...
}
```
# @Scope
Spring中的Bean默认是单例模式，在代码各处注入的一个Bean都是同一个实例；我们将在Spring IoC创建的Bean对象的请求可见范围称为作用域。注解@Scope可用于更改Bean的作用域。
```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Scope {
    @AliasFor("scopeName")
    String value() default "";

    @AliasFor("value")
    String scopeName() default "";

    ScopedProxyMode proxyMode() default ScopedProxyMode.DEFAULT;
}
```
> 传入作用域类型时直接传入字符串，例如：@Scope("prototype")，但这种方法字符串拼写错误不容易发现；Spring提供了默认的参数：ConfigurableBeanFactory.SCOPE_PROTOTYPE, ConfigurableBeanFactory.SCOPE_SINGLETON, WebApplicationContext.SCOPE_REQUEST, WebApplicationContext.SCOPE_SESSION
proxyMode也提供了参数：ScopedProxyMode.INTERFACES, ScopedProxyMode.TARGET_CLASS

## 作用域分为：
+ 基本作用域(singleton, prototype)
+ Web作用域(request, session, globalssion)
  由于默认为单例模式，所以需要标注时一般都是使用prototype
```java
@Bean
@Scope("prototype")  // @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public MyBean myBean() {
    return new MyBean();
}
```
prototype是原型模式，具体可以看设计模式中的"原型模式"，每次通过容器的getBean方法获取Bean时，都将产生一个新的Bean实例。

## 常见使用误区，单例调用多例：
```java
@Component
public class SingletonBean {
    @Autowired
    private PrototypeBean bean;
    
    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    
}
```

上面的代码中，外面的SingletonBean默认单例模式，里面的PrototypeBean设置成原型模式。
这里并不能达到我们每次创建返回新实例的效果。
因为@Autowired只会在单例SingletonBean初始化的时候注入一次，再次调用SingletonBean的时候，PrototypeBean不会再创建了(注入时创建，而注入只有一次)。
解决方法1：不使用@Autowired，每次调用多例的时候，直接调用Bean;
解决方法2：Spring的解决办法：设置proxyMode, 每次请求的时候实例化。

> 两种代理模式的区别：
ScopedProxyMode.INTERFACES: 创建一个JDK代理模式
ScopedProxyMode.TARGET_CLASS: 基于类的代理模式
前者只能将其注入一个接口，后者可以将其注入类。

# @Lazy
Spring Boot中Bean默认的作用域为单例模式；而Spring IoC容器会在启动的时候实例化所有单例Bean。熟悉单例模式的朋友也许立刻想到了饿汉模式和懒汉模式。在Spring Boot中，@Lazy用于指定Bean是否取消预初始化，该注解可以用于直接或者间接用了@Component注解的类，或使用了@Bean的方法。
```java

@Target({ElementType.TYPE, ElementType.METHOD, ElementType.CONSTRUCTOR, ElementType.PARAMETER, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Lazy {
    boolean value() default true;
}
```
@Lazy注解使用在@Bean方法上：
```java
@Bean
@Lazy
public MyBean myBean() {
    return new MyBean();
}
```
也可以在@Comfiguration类上使用，该类中所有的@Bean方法都会延迟初始化
```java
@Configuration
@Lazy
public class MyConfig() {
    @Bean
    public MyBean myBean1() {
        return new MyBean();
    }
    @Bean
    public MyBean myBean2() {
        return new MyBean();
    }
}
```

在标注了@Lazy的@Configuration类内，在某个@Bean方法上标注@Lazy(false)，表明这个bean立即初始化。
```java
@Configuration
@Lazy
public class MyConfig() {
    @Bean
    @Lazy(false)
    public MyBean myBean1() {
        return new MyBean();
    }
}
```
# @Profile
@Profile注解在前面的的文章中提到过，这是一个直接用英文翻译比较难理解的词；profile本身的意思是"轮廓"，"侧写"，在编程的其他领域，这个词有针对每个用户的数据存储的意思。而在Spring Boot当中，这个词用于区分不同的环境，如开发环境，测试环境，生产环境。
```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional({ProfileCondition.class})
public @interface Profile {
    String[] value();
}
```
该注解可以传入多个字符串，@Profile("prod"), @Profile({"test","master"})都是合法的使用，如果有多个条件，需要同时满足

一般在@Configuration下使用，标注在能返回Bean的类或者方法上，标注的时候填入一个字符串，作为一个场景或者一个区分。
```java
@Configuration
@Profile("dev")
public class MyConfig {
    // your beans here
}
```
当Profile为dev时，上面的代码才会在Spring中注册MyConfig这个类。

在Profile的名字前加"!"，即Profile不激活才注册相应的Bean。
```java
...
@Bean
@Profile("!test") // test不激活才创建该Bean
public MyBean myBean() {
    return new MyBean();
}
...
```
## 激活环境
最推荐的方式是在计算机中配置环境变量：export SPRING_PROFILES_ACTIVE=prod，这样注册的环境就和计算机绑定在一起。
也可以设置启动参数-Dspring.profiles.active=test
允许同时激活多个环境，如：-Dspring.profiles.active=test,master
也可以在application.yml中配置，或者使用@ActiveProfiles注解，但是它们直接写在了源代码中，失去了灵活性，这里就不展开说了。
> prod, dev, test等都是常用的缩写, 属于自己定义的内容而非Spring提供的定义

# @ComponentScan
标注了@Component等注解的类会被注册为Bean。那么@ComponentScan自然就是用于开启包扫描，将Bean注册到Spring IoC环境中的注解。该注解标注在启动类上，它定义了扫描路径，从中找到标识了需要装配的类，自动将其装配到环境中。

> 在Spring Boot中，我们常常见到的是复合注解@SpringBootApplication，这个注解的功能之一等效于使用了@ComponentScan，因此我们前面没有显式提到要使用@ComponentScan

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Repeatable(ComponentScans.class)
public @interface ComponentScan {
    @AliasFor("basePackages")
    String[] value() default {};

    @AliasFor("value")
    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};

    Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;

    Class<? extends ScopeMetadataResolver> scopeResolver() default AnnotationScopeMetadataResolver.class;

    ScopedProxyMode scopedProxy() default ScopedProxyMode.DEFAULT;

    String resourcePattern() default "**/*.class";

    boolean useDefaultFilters() default true;

    ComponentScan.Filter[] includeFilters() default {};

    ComponentScan.Filter[] excludeFilters() default {};

    boolean lazyInit() default false;

    @Retention(RetentionPolicy.RUNTIME)
    @Target({})
    public @interface Filter {
        FilterType type() default FilterType.ANNOTATION;

        @AliasFor("classes")
        Class<?>[] value() default {};

        @AliasFor("value")
        Class<?>[] classes() default {};

        String[] pattern() default {};
    }
}
```
@ComponentScan标注在启动类上，如果不传入basePackages属性，那么扫描的路径默认为当前包以及当前路径下的子包，这也是启动类为什么一般放在源代码最外层的原因。
```java
@ComponentScan
public class MyApplication {
    // 启动类
}
```
includeFilters和excludeFilters两个属性的行为比较理解，就是包含或者排除哪些特殊情况，但Filter类值得讲一讲。
FilterType有五种类型：注解类型FilterType.ANNOTATION, 指定固定类FilterType.ASSIGNABLE_TYPE, 切入点类型FilterType.ASPECTJ, 正则表达式FilterType.REGEX, 自定义类型FilterType.CUSTOM
首先说注解类型，比如我们知道@Controller会被注册为Bean，我们可以将其排除。
```java
@ComponentScan(
    excludeFilters={@Filter(type=FilterType.ANNOTATION,classes={Controller.class})}
)
public class MyApplication {
    // 启动类
}
```
使用includeFilters将未标注的自定义类注册进容器：
```java
@ComponentScan(
    includeFilters={@Filter(type=FilterType.ASSIGNABLE_TYPE, classes={MyClass.class})}
)
public class MyApplication {
    // 启动类
}
```

# 总结

我们对Spring注解的学习整理也完成了第一阶段。别的文章往往一开始都会介绍@SpringBootApplication，而我们仅仅只让它在第十篇文章内露了一下面。看似我们的文章组织的没有章法，实际上也是无可奈何，平级的知识之间的关系就如同网状，起步阶段总是感觉备受困扰，我已经按照我认为最容易理解的顺序组织内容。如果看的不太懂，不妨先去找找DEMO，上手敲一敲。
回到我们的文章上来，第一阶段的主题是Bean装配，想要将Bean注册到Spring IoC容器。一般有两种思路：
①使用@Configuration标注在类上, 正如其名字所表达的那样，在这个配置类中，我们可以在返回Bean的方法上标注@Bean；
②直接在类上标注@Component，那么这个类就会被注册到容器中。
Bean被注册到了容器中，但我们并不能直接使用，否则那岂不是成了全局变量，开了历史的倒车？
@Autowired能够将Bean注入到成员变量中，它实际上调用了setter方法，所以需要注入的字段，需要声明public的setter方法。
自动注入时类型必须匹配，必须是对应的类或者子类。当对应的类只有一个时自然不必烦恼。当有多个实例时则根据名字匹配。但为了自动匹配上而强迫自己按照Bean的名字命名字段对象无异于削足适履，还好，@Qualifier和@Primary能解决我们的问题。Spring会根据@Qualifier内传入的名字去匹配相应的Bean；而@Primary更加适用于Bean之间有优先级顺序的情况，像主从数据库就是一个很好的例子。
之后，我们需要对Bean有着更加细粒度的控制。比如@Scope来决定Bean的作用域。@Lazy来决定Bean是否懒加载。@Profile让我们根据条件决定Bean是否装配。
最后，是十分重要常常被提到，却往往沦为@SpringBootApplication的背景板的@ComponentScan，它开启了包扫描，是我们Bean被装配到容器中的最大功臣。

# @ConfigurationProperties

即便现在简化了配置，但是一个独立的配置文件总是易于理解而且使人安心的。Spring在构建完项目后，会默认在resources文件夹下创建一个application.properties文件，application.yml也是一样的效果。@ConfigurationProperties可以获取配置文件中的数据，将其注入类。
```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ConfigurationProperties {
    @AliasFor("prefix")
    String value() default "";

    @AliasFor("value")
    String prefix() default "";

    boolean ignoreInvalidFields() default false;

    boolean ignoreUnknownFields() default true;
}
```

向注解中传入配置文件中的前缀名，如果配置文件如下：

```yml
myConfigs:
  config1:
    field1: f1
    field2: f2
    field3: f3
```
那么代码中的配置类应该这样写：
```java
@Component
@ConfigurationProperties("myConfigs.config1")
public class MyConfig1 {
    String field1;
    String field2;
    String field3;
}
```

如上所示，field1, field2, field3三个属性就被绑定到了对象上。

ignoreInvalidFields默认为false，不合法的属性的属性会默认抛出异常；
ignoreUnknownFields默认为true, 未能识别的属性会被忽略(所以打错了名字就会被忽略了)
```java
@ConfigurationProperties(prefix="config.prefix", ignoreInvalidFields=true, ignoreUnknownFields=false)
public class MyConfig {
    // fields
}
```
Spring Boot的绑定规则相当宽松，myField, my-field, my_field等都能识别绑定到myField上。

可以给字段设定默认值，这样配置中没有传入时会使用默认值。

```java
@ConfigurationProperties("your.prefix")
public class YourConfig {
    private String field = "Default"
    // setter
}
```
类的字段必须要有public访问权限的setter方法。

# @EnableConfigurationProperties
我们从前面大概知道，在Spring中，想让一个类被扫描进入IoC容器中，一般有两种方法：一是在这个类上添加@Component；二是在配置类上指定类。第二类的方法对应到实际中就是大多以Enable开头，例如@EnableConfigurationProperties就是和@ConfigurationProperties搭配使用。
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({EnableConfigurationPropertiesRegistrar.class})
public @interface EnableConfigurationProperties {
    String VALIDATOR_BEAN_NAME = "configurationPropertiesValidator";

    Class<?>[] value() default {};

```

使用使用时要加载配置类上，启动类也是一种配置类。
```java
@Configuration
@EnableConfigurationProperties(YouConfigurationProperties.class)
public class MyConfig {
    // ...
}
```
 # @Value
 使用@ConfigurationProperties能够将配置注入到整个类中，而@Value注解能够将配置注入到字段中，进行更为细粒度的控制。
```java
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Value {
    String value();
}
```

> 注意：注解修饰的字段不能为static或者final。

使用上有两种形式：
@Value("${}")，用来加载外部文件中的值；
@Value("#{}")，用于执行SpEl表达式，并将内容赋值给属性。
我们可以获取Spring Boot配置文件(.yml或.properties)中的属性值并将其赋值给指定变量。
```java
@Value("${my.config.field}")
private String value;
...
```

SpEL是一种表达式语言，能够动态的运行语句。
如果有使用Thymeleaf，会感到一种熟悉感。

```java
...
@Value("#{1 + 1}")
private Integer value; // 2
...

```