---
title: Lombok注解
date: 2024-05-06 11:20:13
tags:
---

# @AllArgsConstructor

生成包含所有字段的构造器，标记为的字段@NonNull将对这些参数进行空检查。

## 参数 
1. staticName : 不为空的话，生成一个静态方法返回实例，并把构造器设置为private

```java
@AllArgsConstructor(staticName = "create")
public class Example {

    private int foo;
    private final String bar;
}
```
生成：

```java
public class Example {
    private int foo;
    private final String bar;

    private Example(int foo, String bar) {
        this.foo = foo;
        this.bar = bar;
    }

    public static Example create(int foo, String bar) {
        return new Example(foo, bar);
    }
}
```

1. access : 这个选项可以用来改变生成的构造方法的访问级别，默认public

# @RequiredArgsConstructor

生成必须初始化字段的构造器，比如带final、@NonNull，对于标有@NonNull注解的字段，还将生成一个显式的null检查。

```java
@RequiredArgsConstructor
public class Example {

    @NonNull
    private Integer foo;
    private final String bar;
}
```

生成后：

```java
public class Example {
    @NonNull
    private Integer foo;
    private final String bar;

    public Example(@NonNull Integer foo, String bar) {
        if (foo == null) {
            throw new NullPointerException("foo is marked @NonNull but is null");
        } else {
            this.foo = foo;
            this.bar = bar;
        }
    }
}
```

# @NoArgsConstructor

生成无参数构造器

## 参数
1. access：访问权限修饰符
2. force：为true时，当有 final 字段没有被初始化时，这个选项可以强制 Lombok 生成一个无参数的构造方法，并将所有 final 字段初始化为其默认值（0、false、null等）
3. onConstructor：添加注解，参考@Getter#onMethod

# @NoArgsConstructor、@AllArgsConstructor和@RequiredArgsConstructor区别

1. @NoArgsConstructor：这个注解用于生成一个无参数的构造函数。当你在一个类上使用这个注解时，Lombok会自动为该类生成一个无参数的构造函数。这个注解在Spring Boot中常常被用在不需要参数就能创建对象的地方，例如单例模式或者作为其他构造函数的依赖注入。
2. @AllArgsConstructor：这个注解用于生成一个包含所有参数的构造函数。当你在一个类上使用这个注解时，Lombok会自动为该类的所有字段生成一个带有参数的构造函数。这个注解在Spring Boot中常常被用在需要使用所有字段来创建对象的地方，例如DTO（Data Transfer Object）对象的创建。
3. @RequiredArgsConstructor：这个注解用于自动生成带有final修饰符的成员变量的构造函数。当一个类中存在多个final修饰符的成员变量时，使用这个注解可以避免手动编写重复的构造函数代码。这个注解在Spring Boot中常常被用在需要将final字段绑定到具体实现的地方，例如Spring的Bean配置。

> 总结一下，@NoArgsConstructor、@AllArgsConstructor和@RequiredArgsConstructor这三个注解在Spring Boot项目中的常见应用场景包括：
> 在单例模式中，使用@NoArgsConstructor来创建一个无参数的构造函数，以便在不需要任何参数的情况下创建对象。
> 在DTO对象的创建中，使用@AllArgsConstructor来创建一个包含所有字段的构造函数，以便将所有字段的值传递给对象。
> 在Spring的Bean配置中，使用@RequiredArgsConstructor来自动生成包含final字段的构造函数的代码，以便将final字段绑定到具体实现。

# @Getter
生成getter、写在类上会生成该类下所有字段的getter。写在某个字段上就作用与该字段

## 参数
1. onMethod：把需要添加的注解写在这
```java
public class Example {

    @Getter(onMethod_={@Deprecated}) // JDK7写法 @Getter(onMethod=@__({@Deprecated}))
    private int foo;
    private final String bar  = "";
}
```

生成：
```java
public class Example {
    private int foo;
    private final String bar = "";

    public Example() {
    }

    /** @deprecated */
    @Deprecated
    public int getFoo() {
        return this.foo;
    }
}
```

2. value：访问权限修饰符

# @Setter
生成Setter

1. onMethod：在方法上添加中注解，见@Getter#onMethod
2. onParam：在方法的参数上添加注解，见@Getter#onMethod
3. value：访问权限修饰符


# @NonNull

空检查
```java
public class Example {

    @NonNull
    @Getter
    @Setter
    private Integer foo;
}
```

生成后：

```java
public class Example {
    @NonNull
    private Integer foo;

    public Example() {
    }

    @NonNull
    public Integer getFoo() {
        return this.foo;
    }

    public void setFoo(@NonNull Integer foo) {
        if (foo == null) {
            throw new NullPointerException("foo is marked @NonNull but is null");
        } else {
            this.foo = foo;
        }
    }
}

```