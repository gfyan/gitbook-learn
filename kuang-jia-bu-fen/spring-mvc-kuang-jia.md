# Spring MVC框架

### Spring MVC是什么？

Spring MVC是一个基于MVC架构的用来简化web应用程序开发的应用开发框架，它是Spring的一部分，它和Struts2一样都属于表现层的框架。

### Spring MVC都有哪些组件？各个组件都有什么作用？

Spring MVC一共有九大组件：

- MultipartResolver：主要是解析Content-Type为multipart/\*请求的解析器接口，比如文件上传请求，MultipartResolver会将HttpServletRequest封装成MultipartHttpServletRequest，后续流程从MultipartHttpServletRequest中获取上传的文件。
- LocaleResolver：本地化解析解析器，解析出请求中使用的语言，并且设置所要使用的语言。
- ThemeResolver：主题解析器，现在前后端分离了，这个功能用的不多。
- HandlerMapping：处理匹配的接口，根据handler获取所对应的处理器和拦截器数组。
- HandlerAdapter：适配器，需要一个调用者来实现handler是怎么被使用，怎么被执行，而HandlerAdapter的用途就在于此。
- HandlerExceptionResolver：异常解析器。
- RequestToViewNameTranslator：请求到视图名的转换器。
- ViewResolver：视图解析器。

### 描述一下Spring MVC的请求流程？

![avatar](https://blog-pictures.oss-cn-shanghai.aliyuncs.com/15300766829012.jpg)

Spring MVC请求核心流程是DispatcherServlet，DispatcherServlet是继承自FrameworkServlet，而FrameworkServlet继承自HttpServlet。

DispatcherServlet收到请求以后会进行一下的流程操作：

1. 接受请求。
2. DispatcherServlet根据Url调用HandlerMapping获得对应的处理handler，最后以HandlerExecutionChain形式返回，HandlerExecutionChain包含handler以及对应的拦截器。
3. 根据handler找到对应的适配器Adapter，通过适配器Adapter对相应的handle方法进行调用。
4. 如果存在拦截器，进行拦截器流程的PreHandle执行。
5. 通过适配器HandlerAdapter执行handle方法，该方法最终会下发到每个Controller类对应的RequestMapping方法上，其中参数的装载、数据的转换、数据校验都会在handle方法中进行。
6. Controller执行完毕后，继续执行拦截器PostHandle执行，通过后返回ModelView视图对象。
7. 解析对应的ModelView视图，选择一个合适的ViewResolver返回给DispatcherServlet。
8. 响应对应的视图以及数据给用户。

**现在很多都是前后端分离，所以上述所说的视图就不存在了，目前我们所用的ResponseBody注解，直接会在第3步走完以后直接判断方法是否有@ResponseBody注解，如果存在直接封装对应的数据返回给前端。**

### 介绍一下 WebApplicationContext？

WebApplicationContext 是实现ApplicationContext接口的子类，专门为 WEB 应用准备的。
- 它允许从相对于 Web 根目录的路径中加载配置文件，完成初始化 Spring MVC 组件的工作。
- 从 WebApplicationContext 中，可以获取 ServletContext 引用，整个 Web 应用上下文对象将作为属性放置在 ServletContext 中，以便 Web 应用环境可以访问 Spring 上下文。

### 说说Spring MVC的异常处理？

Spring MVC 提供了异常解析器HandlerExceptionResolver 接口，将处理器(handler)执行时发生的异常，解析成对应的 ModelAndView 结果，最后通过调用HandlerException流程进行异常处理机制。

### 详细介绍下 Spring MVC 拦截器？

HandlerInterceptor接口定义了三个方法，preHandle（调用Controller方法之前执行）、postHandle（调用Controller方法之后执行）、afterCompletion（处理完Controller返回结果后执行）

### Spring MVC的拦截器可以做哪些事情？

- 记录访问日志。
- 对请求进行token校验以及安全性校验等。
- 对接口请求进行统计。

### Spring MVC的拦截器和Filter过滤器有什么差别？

**功能相同：**其实拦截器和Filter的功能几乎一样，都是对于请求进行某种策略的校验过滤。
**容器不同：**但是他们属于不同的容器，Filter构建在Servlet容器上，但是拦截器是构建在Spring容器上的。
**使用不同：**拦截器可以在Controller执行之前、执行之后、处理完毕之后三个节点进行拦截，但是Filter只有一个只能对


### HttpMessageConverter了解吗，主要功能是什么？

HttpMessageConverter是一种策略接口，它指定了一个转换器，它可以转换HTTP请求和响应。Spring REST用这个接口转换HTTP 响应到多种格式，例如：JSON或XML或普通String。



