# Spring-boot框架

## Spring Boot

Spring Boot是spring开源组织下的一个开源框架，是Spring组件一站式解决方案，简化了Spring的配置复杂度，提供了各种start组件便于开发人员 快速开发上手。

**都集成了哪些组件** 
1. spring-boot-starter-web 快速配置spring mvc 
2. mybatis-spring-boot-starter 快速配置mybatis 
3. spring-boot-starter-data-redis 快速配置redis 
4. druid-spring-boot-starter 快速配置druid jdbc线程池

。。。。。。等等

## Spring Boot、Spring MVC和Spring区别。

Spring Boot是建立在Spring之上的，而Spring Foramwork又包含Spring IOC、Spring MVC、Spring AOP等模块。

## Spring Boot是怎么实现自动装配starter组件的。

Spring Boot实现自动装配通过@EnableAutoConfiguration注解开启自动配置，加载Spring.factories中注册的各种AutoConfiguration类，当某个AutoConfiguration类满足其@Conditional指定的生成条件时就会自动实例化该bean，并注入到Bean容器中，从而达到自动装配各个组件。

**@EnableAutoConfiguration注解**

@EnableAutoConfiguration注解表示开启自动配置的注解，这个注解有两个参数需要注意，一个是exclude class数组类型，一个是excludeName string数组，这两个表示自动注解需要移除自动装载功能的类，@EnableAutoConfiguration注解核心功能是通过@Import注解来实现的。

**@Import注解**

@EnableAutoConfiguration注解使用的是AutoConfigurationImportSelector注解，AutoConfigurationImportSelector继承Import，重写了selectImport，该方法会去加载META-INF目录下的配置文件，配置文件中配置了需要自动加载的类。



