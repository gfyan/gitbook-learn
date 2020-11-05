# Spring框架

### 什么是Spring？
Spring是一个轻量级的开发框架，主要是为了降低程序开发的复杂度，它具有轻量级、松耦合的特性，特具有多个组件，开发人员可以根据不同的
需求选择对应的组件进行开发，同时Spring还集成了其他的框架，例如Mybatis、Spring MVC、Hibernate等框架，所以它也被称为脚手架。

### Spring Framework中有多少个模块，它们分别都有什么作用？
Spring Framework主要的模块有Spring核心模块、测试模块、数据模块、Web模块。其中他们每个模块分别包括：

**核心层**

核心层包括Spring Bean容器、SpringIOC功能的实现主要是通过DI（依赖注入）的方式，Spring AOP功能的实现以及SpEL spring表达式语言。
>我们编程代码中常见的@Value加载配置文件中的，其中Value里面的写法就是SpEL表达式。

**测试层**

测试层就提供了一些Junit等一些测试的支持。

**数据层包**

数据层包括JDBC（提供了对数据库的访问）、ORM（对象关系映射框架，集成了Hibernate和JPA）、OXM、JMS、Transactions（事务管理）。
> JPA（Java Persistence API）java持久层API

**Web层**

Web层包括WebSocket（Spring4.0版本添加的支持，在一个Web应用中实现高效、双向通讯），WebMVC（构建 Web 应用程序的 MVC 实现），WebFlux（包含了对反应式 HTTP、服务器推送事件和 WebSocket 的客户端和服务器端的支持）。
>WebFlux是重点

### 使用Spring Framework的好处？

- DI依赖注入，使得构造器和JavaBean、properties文件中的依赖关系一目了然。
- 轻量级。
- 面向切面编程(AOP)： Spring 支持面向切面编程，同时把应用的业务逻辑与系统的服务分离开来。
- 集成主流框架，Spring集成了很多主流框架，例如Mybatis、Hibernate、Quartz（定时任务）等。
- Web框架支持。
- 事务管理。

### Spring 框架中都用到了哪些设计模式？

- 代理模式 AOP中使用到。
- 单例模式 Spring Bean默认就是单例。
- 模板方法 RestTemplate、JmsTemplate、JdbcTemplate。
- 工厂模式 BeanFactory 用来创建对象的实例。
- 适配器模式 在Spring MVC中，DispatcherServlet 根据请求信息调用 HandlerMapping，解析请求对应的 Handler。解析到对应的 Handler（也就是我们平常说的 Controller 控制器）后，开始由HandlerAdapter 适配器处理。HandlerAdapter 作为期望接口，具体的适配器实现类用于对目标类进行适配，Controller 作为需要适配的类。
- 装饰者模式，Spring中用到的包装器模式在类名上含有Wrapper或者 Decorator。这些类基本上都是动态地给一个对象添加一些额外的职责。
- 观察者模式 Spring 事件驱动模型就是观察者模式。


### Spring框架中有哪些类型的事件？

1. 上下文更新事件（ContextRefreshedEvent）：在调用ConfigurableApplicationContext 接口中的refresh()方法时被触发。
2. 上下文开始事件（ContextStartedEvent）：当容器调用ConfigurableApplicationContext的Start()方法开始/重新开始容器时触发该事件。
3. 上下文停止事件（ContextStoppedEvent）：当容器调用ConfigurableApplicationContext的Stop()方法停止容器时触发该事件。
4. 上下文关闭事件（ContextClosedEvent）：当ApplicationContext被关闭时触发该事件。容器被关闭时，其管理的所有单例Bean都被销毁。
5. 请求处理事件（RequestHandledEvent）：在Web应用中，当一个http请求（request）结束触发该事件。如果一个bean实现了ApplicationListener接口，当一个ApplicationEvent 被发布以后，bean会自动被通知。

### 什么是Spring IOC 容器？

控制反转即IOC (Inversion of Control)，它把传统上由程序代码直接操控的对象的调用权交给容器，通过容器来实现对象组件的装配和管理。所谓的“控制反转”概念就是对组件对象控制权的转移，从程序代码本身转移到了外部容器。Spring IOC负责创建对象、管理对象，通过DI装配对象、配置对象，并且管理对象的声明周期。

### 什么是依赖注入（DI）？

依赖注入就是你无需手动创建实例对象所依赖的对象，你只需要在实例对象中对依赖的对象进行描述即可，实例的创建以及赋值交给Spring框架或者其他框架来实现，这就是依赖注入。

**依赖注入分别有哪些方式**
1. 构造器注入。好处是保证依赖不可变（final关键字）、保证依赖不为空（省去了我们对其检查）、保证返回客户端（调用）的代码的时候是完全初始化的状态、提升了代码的可复用性。缺点是如果需要注入的对象多构造器的入参会越来越复杂，特别是某些参数可选的情况下，多参数构造器显得更加笨重，存在循环依赖死循环问题。
2. setter注入。好处是与传统JavaBean写法更为相似，程序开发人员更容易理解，不存在循环依赖死循环问题。
3. 接口注入。

其中1、2是现在常用的方式，3基本上已经不用，

### 控制反转(IOC)有什么作用？
- 托管对象的创建和维护依赖关系。
- 解耦，由容器维护具体的对象，业务代码无需维护。

### IOC的优点是什么？
- IOC降低了业务代码的复杂度。
- 将业务松散耦合便于各个单元进行模块测试。
- IOC支持懒汉式加载、饿汉式加载。

### Spring IOC 的实现机制？
IOC的实现机制就是工厂模式+反射，但是Spring IOC过程更为复杂，在实现IOC的过程会增加许多额外的功能。

### BeanFactory 和 ApplicationContext有什么区别？
BeanFactory和ApplicationContext是Spring的两大核心接口，都可以当做Spring的容器。其中ApplicationContext是BeanFactory的子接口。

BeanFactory：是Spring里面最底层的接口，包含了各种Bean的定义，读取bean配置文档，管理bean的加载、实例化，控制bean的生命周期，维护bean之间的依赖关系。

- ApplicationContext接口作为BeanFactory的派生，除了提供BeanFactory所具有的功能外，还提供了更完整的框架功能：
- 继承MessageSource，因此支持国际化。
- 统一的资源文件访问方式。
- 提供在监听器中注册bean的事件。
- 同时加载多个配置文件。
- 载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层。

### ApplicationContext都有哪些实现？
- **FileSystemXmlApplicationContext ：**此容器从一个XML文件中加载beans的定义，XML Bean 配置文件的全路径名必须提供给它的构造函数。

- **ClassPathXmlApplicationContext ：**此容器从一个XML文件中加载beans的定义，XML Bean 配置文件的全路径名必须提供给它的构造函数。

- **WebXmlApplicationContext ：**此容器从一个XML文件中加载beans的定义，XML Bean 配置文件的全路径名必须提供给它的构造函数。


### 什么是Spring beans？
Spring beans 是那些形成Spring应用的主干的java对象。它们被Spring IOC容器初始化，装配，和管理。这些beans通过容器中配置的元数据创建。比如以XML文件中的形式定义，注解Bean定义的Java POJO。

### Spring支持的几种bean的作用域

- **singleton :** bean在每个Spring ioc 容器中只有一个实例。
- **prototype：**一个bean的定义可以有多个实例。
- **request：**每次http请求都会创建一个bean，该作用域仅在基于web的Spring ApplicationContext情形下有效。
- **session：**在一个HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。
- **global-session：**在一个全局的HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。


### 使用@Autowired注解自动装配的过程是怎样的？
在启动spring IOC时，容器自动装载了一个AutowiredAnnotationBeanPostProcessor后置处理器，当容器扫描到@Autowied、@Resource或@Inject时，就会在IOC容器自动查找需要的bean，并装配给该对象的属性。在使用@Autowired时，首先在容器中查询对应类型的bean：
- 如果查询结果刚好为一个，就将该bean装配给@Autowired指定的数据；
- 如果查询的结果不止一个，那么@Autowired会根据名称来查找；
- 如果上述查找的结果为空，那么会抛出异常。解决方法时，使用required=false。

### @Component, @Controller, @Repository, @Service 有何区别？

- @Component：这将java类标记为bean。它是任何Spring管理组件的通用构造型。spring的组件扫描机制现在可以将其拾取并将其拉入应用程序环境中。
- @Controller：这将一个类标记为 Spring Web MVC 控制器。标有它的 Bean 会自动导入到 IOC 容器中。
- @Service：此注解是组件注解的特化。它不会对@Component注解提供任何其他行为。您可以在服务层类中使用@Service而不是 @Component，因为它以更好的方式指定了意图。
- @Repository：这个注解是具有类似用途和功能的 @Component 注解的特化。它为 DAO 提供了额外的好处。它将 DAO 导入 IOC 容器，并使未经检查的异常有资格转换为 Spring DataAccessException。

### @Autowired 注解有什么作用

@Autowired默认是按照类型装配注入的，默认情况下它要求依赖对象必须存在（可以设置它required属性为false）。@Autowired 注解提供了更细粒度的控制，包括在何处以及如何完成自动装配。它的用法和@Required一样，修饰setter方法、构造器、属性或者具有任意名称和/或多个参数的PN方法。

```
public class Person {
    private String name;
    @Autowired
    public void setName(String name) {
        this.name=name;
    }
    public string getName(){
        return name;
    }
}

public class Person {
	@Autowired
    private String name;
}

public class Person {
    private String name;

    @Autowired
    public Person(name n){
    	this.name = n;
    }
}
```
### @Autowired和@Resource之间的区别？

**相同之处：**
- @Autowired和@Resource可用于：构造函数、成员变量、Setter方法。
- 如果装配的时候出现多个符合条件的实例都会报错。

**区别之处：**
- @Autowired默认是按照类型装配注入的，默认情况下它要求依赖对象必须存在（可以设置它required属性为false）。
- @Resource默认是按照名称来装配注入的，只有当找不到与名称匹配的bean才会按照类型来装配注入。

### @Qualifier注解有什么作用？
当使用@Autowired注解进行装配属性的时候，出现多个实例对象的时候可以采用@Qualifier注解进行特定Bean的指定，消除多个Bean的歧义。

### @RequestMapping 注解有什么用？
@RequestMapping 注解用于将特定 HTTP 请求方法映射到将处理相应请求的控制器中的特定类/方法。此注释可应用于两个级别：
- 类级别：映射请求的URL
- 方法级别：映射URL以及HTTP请求方法

### 事务的特性有哪些？

- 原子性：一个事务（transaction）中的所有操作，或者全部完成，或者全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被恢复（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。即，事务不可分割、不可约简。
- 隔离性：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。
- 一致性：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设约束、触发器)、级联回滚等。
- 持久性：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

### Spring支持的事务管理类型，Spring事务实现方式有哪些？
- **编程式事务管理：**这意味你通过编程的方式管理事务，给你带来极大的灵活性，但是难维护。
- **声明式事务管理：**这意味着你可以将业务代码和事务管理分离，你只需用注解和XML配置来管理事务。

### Spring 事务如何和不同的数据持久层框架做集成？
>久层框架，指的是Spring JDBC、Hibernate、Spring JPA、MyBatis等等。

Spring事务的管理，是通过 org.springframework.transaction.PlatformTransactionManager进行管理的，不同的数据持久层框架，会有其对应的 PlatformTransactionManager实现类，所有的实现类，都基于AbstractPlatformTransactionManager这个骨架类，然后不同的实现类与不同的持久层进行集成。
- HibernateTransactionManager，和Hibernate5的事务管理做集成。
- DataSourceTransactionManager，和JDBC的事务管理做集成，所以它也适用于 MyBatis、Spring JDBC等等。
- JpaTransactionManager，和JPA的事务管理做集成。

### Spring 事务隔离级别有哪几个？
- DEFAULT 默认隔离级别，采用的是当前jdbc的隔离级别。
- READ_UNCOMMITTED 读未提交，能读取到事务未提交的数据。
- READ_COMMITTED 读提交，只能读取到已经提交的事务数据，可以防止脏读，但是幻读和不可重复读依然存在。
- REPEATABLE_READ 可重复读，保证当前事务下多次读取结果都是一致的，除非数据本身修改，可以防止脏读和不可重复读，但存在幻读。
- SERIALIZABLE 串行化，最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。

### Spring 事务传播级别有哪几个？
Spring事务传播级别有七个。

支持当前事务：1、当前存在事务则使用该事务。2、当前不存在事务则以非事务的方式运行。3、当前不存在事务则直接抛出异常。

不支持当前事务：1、当前存在事务则挂起当前事务，并创建一个新的事务进行。2、当前存在事务则挂起当前事务，并以非事务方式运行。3、当前存在事务，则抛出异常。

其他情况：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行，当前没有事务则创建一个新的事务进行运行。


### Spring AOP实现原理？

### Spring 事务实现原理？

