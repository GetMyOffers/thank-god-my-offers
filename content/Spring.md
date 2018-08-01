### 1. Spring IOC

 

要了解**控制反转( Inversion of Control )**, 有必要先了解软件设计的一个重要思想：**依赖倒置原则（Dependency Inversion Principle ）**。

**依赖倒置**：从高层谈需求，一层层往下实现。在代码上的体现为把底层类作为参数传入上层类，实现上层类对下层类的控制。

这样做的好处就是如果我们想要改变底层类的定义的时候，上层类并不会受到影响，因为上层类不关心下层是怎么实现的，只要它能够拿到下层类的实例化对象即可。

但是想想看，如果我想要一个车子，那么我得先要实例化一个车身类，要得到车身又得要一个底盘类，依次下去，轮子类，螺丝类，多么繁琐啊，并且依赖关系多了我们根本记不住依赖关系啊，所以 Spring 帮助我们做了这一步，我们要做的只是在配置文件中提前写好依赖关系即可，Spring 会返回一个我们想要的汽车实例。

那么，Spring 怎么做到的呢？我做了一个图，如下：

![](https://github.com/Lisanaaa/Nebulas-Learn/blob/master/image/Spring-Bean-Create.png)

#### 总结

Spring IOC 容器主要有继承体系底层的 BeanFactory、高层的 ApplicationContext 和 WebApplicationContext

Bean 有自己的生命周期

**容器启动原理: **

Spring 应用的 IOC 容器通过 tomcat 的 Servlet 或 Listener 监听启动加载；Spring MVC 的容器由 DispatchServlet 作为入口加载；Spring 容器是 Spring MVC 容器的父容器

**容器加载 Bean 原理：**

- BeanDefinitionReader 读取 Resource 所指向的配置文件资源，然后解析配置文件。配置文件中每一个解析成一个 BeanDefinition 对象，并保存到 BeanDefinitionRegistry 中

- 容器扫描 BeanDefinitionRegistry 中的 BeanDefinition；调用 InstantiationStrategy 进行Bean 实例化的工作；使用 BeanWrapper 完成 Bean 属性的设置工作；

单例 Bean 缓存池：Spring 在 DefaultSingletonBeanRegistry 类中提供了一个用于缓存单实例 Bean 的缓存器，它是一个用 HashMap 实现的缓存器，单实例的 Bean 以 beanName 为键保存在这个 HashMap 中。


### 2. Spring AOP


AOP为Aspect Oriented Programming的缩写，意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是Spring框架中的一个重要内容，它通过对既有程序定义一个切入点，然后在其前后切入不同的执行内容，比如常见的有：打开数据库连接/关闭数据库连接、打开事务/关闭事务、记录日志等。基于AOP不会破坏原来程序逻辑，因此它可以很好的对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

下面主要讲两个内容，一个是如何在Spring Boot中引入Aop功能，二是如何使用Aop做切面去统一处理Web请求的日志。

####实现原理

SpringAOP 的实现分为两种

* 基于 jdk 的动态代理,`java.lang.reflect.Proxy`,需要被代理类实现接口
  * 最终生成一个实现被代理类接口的类.同时持有被代理对象
* 基于 cglib 的字节码技术
  * 最终生成一个被代理类的子类.

#### 准备工作

因为需要对web请求做切面来记录日志，所以先引入web模块，并创建一个简单的hello请求的处理。

- `pom.xml`中引入web模块

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

- 实现一个简单请求处理：通过传入name参数，返回“hello xxx”的功能。

```java
@RestController
public class HelloController {

    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    @ResponseBody
    public String hello(@RequestParam String name) {
        return "Hello " + name;
    }

}
```

下面，我们可以对上面的/hello请求，进行切面日志记录。

#### 引入AOP依赖

在Spring Boot中引入AOP就跟引入其他模块一样，非常简单，只需要在`pom.xml`中加入如下依赖：

```java

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

在完成了引入AOP依赖包后，一般来说并不需要去做其他配置。也许在Spring中使用过注解配置方式的人会问是否需要在程序主类中增加`@EnableAspectJAutoProxy`来启用，实际并不需要。

可以看下面关于AOP的默认配置属性，其中`spring.aop.auto`属性默认是开启的，也就是说只要引入了AOP依赖后，默认已经增加了`@EnableAspectJAutoProxy`。

```java
# AOP
spring.aop.auto=true # Add @EnableAspectJAutoProxy.
spring.aop.proxy-target-class=false # Whether subclass-based (CGLIB) proxies are to be created (true) as
opposed to standard Java interface-based proxies (false).

```

而当我们需要使用CGLIB来实现AOP的时候，需要配置`spring.aop.proxy-target-class=true`，不然默认使用的是标准Java的实现。

#### 实现Web层的日志切面

实现AOP的切面主要有以下几个要素：

- 使用`@Aspect`注解将一个java类定义为切面类

- 使用`@Pointcut`定义一个切入点，可以是一个规则表达式，比如下例中某个package下的所有函数，也可以是一个注解等。

- 根据需要在切入点不同位置的切入内容

- 使用`@After`在切入点结尾处切入内容

- 使用`@Before`在切入点开始处切入内容

- 使用`@After`在切入点结尾处切入内容

- 使用`@AfterReturning`在切入点return内容之后切入内容（可以用来对处理返回值做一些加工处理）

- 使用`@Around`在切入点前后切入内容，并自己控制何时执行切入点自身的内容

- 使用`@AfterThrowing`用来处理当切入内容部分抛出异常之后的处理逻辑


```java
@Aspect
@Component
public class WebLogAspect {

  private Logger logger = Logger.getLogger(getClass());

  @Pointcut("execution(public * com.didispace.web..*.*(..))")
  public void webLog(){}

  @Before("webLog()")
  public void doBefore(JoinPoint joinPoint) throws Throwable {
      // 接收到请求，记录请求内容
      ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
      HttpServletRequest request = attributes.getRequest();

      // 记录下请求内容
      logger.info("URL : " + request.getRequestURL().toString());
      logger.info("HTTP_METHOD : " + request.getMethod());
      logger.info("IP : " + request.getRemoteAddr());
      logger.info("CLASS_METHOD : " + joinPoint.getSignature().getDeclaringTypeName() + "." + joinPoint.getSignature().getName());
      logger.info("ARGS : " + Arrays.toString(joinPoint.getArgs()));

  }

  @AfterReturning(returning = "ret", pointcut = "webLog()")
  public void doAfterReturning(Object ret) throws Throwable {
      // 处理完请求，返回内容
      logger.info("RESPONSE : " + ret);
  }

}
```

可以看上面的例子，通过`@Pointcut`定义的切入点为`com.didispace.web`包下的所有函数（对web层所有请求处理做切入点），然后通过`@Before`实现，对请求内容的日志记录（本文只是说明过程，可以根据需要调整内容），最后通过`@AfterReturning`记录请求返回的对象。

通过运行程序并访问：`http://localhost:8080/hello?name=didi`，可以获得日志输出

#### 优化：AOP切面中的同步问题

在WebLogAspect切面中，分别通过doBefore和doAfterReturning两个独立函数实现了切点头部和切点返回后执行的内容，若我们想统计请求的处理时间，就需要在doBefore处记录时间，并在doAfterReturning处通过当前时间与开始处记录的时间计算得到请求处理的消耗时间。

那么我们是否可以在WebLogAspect切面中定义一个成员变量来给doBefore和doAfterReturning一起访问呢？是否会有同步问题呢？

的确，直接在这里定义基本类型会有同步问题，所以我们可以引入ThreadLocal对象，像下面这样进行记录：

```java
@Aspect
@Component
public class WebLogAspect {

  private Logger logger = Logger.getLogger(getClass());

  ThreadLocal<Long> startTime = new ThreadLocal<>();

  @Pointcut("execution(public * com.didispace.web..*.*(..))")
  public void webLog(){}

  @Before("webLog()")
  public void doBefore(JoinPoint joinPoint) throws Throwable {
      startTime.set(System.currentTimeMillis());

      // 省略日志记录内容
  }

  @AfterReturning(returning = "ret", pointcut = "webLog()")
  public void doAfterReturning(Object ret) throws Throwable {
      // 处理完请求，返回内容
      logger.info("RESPONSE : " + ret);
      logger.info("SPEND TIME : " + (System.currentTimeMillis() - startTime.get()));
  }


}
```

#### 优化：AOP切面的优先级

由于通过AOP实现，程序得到了很好的解耦，但是也会带来一些问题，比如：我们可能会对Web层做多个切面，校验用户，校验头信息等等，这个时候经常会碰到切面的处理顺序问题。

所以，我们需要定义每个切面的优先级，我们需要`@Order(i)`注解来标识切面的优先级。**i的值越小，优先级越高**。假设我们还有一个切面是`CheckNameAspect`用来校验name必须为didi，我们为其设置`@Order(10)`，而上文中WebLogAspect设置为`@Order(5)`，所以WebLogAspect有更高的优先级，这个时候执行顺序是这样的：

- 在`@Before`中优先执行`@Order(5)`的内容，再执行`@Order(10)`的内容
- 在`@After`和`@AfterReturning`中优先执行`@Order(10)`的内容，再执行`@Order(5)`的内容

所以我们可以这样子总结：

- 在切入点前的操作，按order的值由小到大执行
- 在切入点后的操作，按order的值由大到小执行

   
### 3. Spring Filter

它使用户可以改变一个 request 和修改一个 response. Filter 不是一个 servlet，它不能产生一个 response，它能够在一个 request 到达 servlet 之前预处理 request，也可以在离开 servlet时处理 response.换种说法，filter 其实是一个 ”servlet chaining” (servlet 链).
一个Filter包括：
1）在servlet被调用之前截获;
2）在servlet被调用之前检查servlet request;
3）根据需要修改request头和request数据;
4）根据需要修改response头和response数据;
5）在servlet被调用之后截获.

#### 定义自己的过滤器

新增HTTPBasicAuthorizeAttribute.java

如果请求的Header中存在Authorization: Basic 头信息，且用户名密码正确，则继续原来的请求，否则返回没有权限的错误信息

```java
package com.xiaofangtech.sunt.filter;

import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.xiaofangtech.sunt.utils.ResultMsg;
import com.xiaofangtech.sunt.utils.ResultStatusCode;
import sun.misc.BASE64Decoder;

@SuppressWarnings("restriction")
public class HTTPBasicAuthorizeAttribute implements Filter{
	
	private static String Name = "test";
	private static String Password = "test";

	@Override
	public void destroy() {
		// TODO Auto-generated method stub
		
	}

	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		// TODO Auto-generated method stub
		
		ResultStatusCode resultStatusCode = checkHTTPBasicAuthorize(request);
		if (resultStatusCode != ResultStatusCode.OK)
		{
			HttpServletResponse httpResponse = (HttpServletResponse) response;
			httpResponse.setCharacterEncoding("UTF-8");  
			httpResponse.setContentType("application/json; charset=utf-8"); 
			httpResponse.setStatus(HttpServletResponse.SC_UNAUTHORIZED);

			ObjectMapper mapper = new ObjectMapper();
			
			ResultMsg resultMsg = new ResultMsg(ResultStatusCode.PERMISSION_DENIED.getErrcode(), ResultStatusCode.PERMISSION_DENIED.getErrmsg(), null);
			httpResponse.getWriter().write(mapper.writeValueAsString(resultMsg));
			return;
		}
		else
		{
			chain.doFilter(request, response);
		}
	}

	@Override
	public void init(FilterConfig arg0) throws ServletException {
		// TODO Auto-generated method stub
		
	}
	
	private ResultStatusCode checkHTTPBasicAuthorize(ServletRequest request)
	{
		try
		{
			HttpServletRequest httpRequest = (HttpServletRequest)request;
			String auth = httpRequest.getHeader("Authorization");
			if ((auth != null) && (auth.length() > 6))
			{
				String HeadStr = auth.substring(0, 5).toLowerCase();
				if (HeadStr.compareTo("basic") == 0)
				{
					auth = auth.substring(6, auth.length());  
		            String decodedAuth = getFromBASE64(auth);
		            if (decodedAuth != null)
		            {
		            	String[] UserArray = decodedAuth.split(":");
		            	
		            	if (UserArray != null && UserArray.length == 2)
		            	{
		            		if (UserArray[0].compareTo(Name) == 0
			            			&& UserArray[1].compareTo(Password) == 0)
		            		{
		            			return ResultStatusCode.OK;
		            		}
		            	}
		            }
				}
			}
			return ResultStatusCode.PERMISSION_DENIED;
		}
		catch(Exception ex)
		{
			return ResultStatusCode.PERMISSION_DENIED;
		}
		
	}
	
	private String getFromBASE64(String s) {  
        if (s == null)  
            return null;  
        BASE64Decoder decoder = new BASE64Decoder();  
        try {  
            byte[] b = decoder.decodeBuffer(s);  
            return new String(b);  
        } catch (Exception e) {  
            return null;  
        }  
    }

}

```

#### 在SpringRestApplication类中注册过滤器，给user/*都加上http basic认证过滤器

```java
package com.xiaofangtech.sunt;

import java.util.ArrayList;
import java.util.List;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.embedded.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;

import com.xiaofangtech.sunt.filter.HTTPBasicAuthorizeAttribute;

@SpringBootApplication
public class SpringRestApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringRestApplication.class, args);
	}
	
	@Bean
    public FilterRegistrationBean  filterRegistrationBean() {
		FilterRegistrationBean registrationBean = new FilterRegistrationBean();
		HTTPBasicAuthorizeAttribute httpBasicFilter = new HTTPBasicAuthorizeAttribute();
		registrationBean.setFilter(httpBasicFilter);
		List<String> urlPatterns = new ArrayList<String>();
	    urlPatterns.add("/user/*");
	    registrationBean.setUrlPatterns(urlPatterns);
	    return registrationBean;
    }
}

```



### 4. Spring Interceptor

Web开发中，我们除了使用 Filter 来过滤请web求外，还可以使用Spring提供的HandlerInterceptor（拦截器）。

HandlerInterceptor 的功能跟过滤器类似，但是提供更精细的的控制能力：在request被响应之前、request被响应之后、视图渲染之前以及request全部结束之后。我们不能通过拦截器修改request内容，但是可以通过抛出异常（或者返回false）来暂停request的执行。

实现 UserRoleAuthorizationInterceptor 的拦截器有： 
- ConversionServiceExposingInterceptor 
- CorsInterceptor 
- LocaleChangeInterceptor 
- PathExposingHandlerInterceptor 
- ResourceUrlProviderExposingInterceptor 
- ThemeChangeInterceptor 
- UriTemplateVariablesHandlerInterceptor 
- UserRoleAuthorizationInterceptor

其中 LocaleChangeInterceptor 和 ThemeChangeInterceptor 比较常用。

配置拦截器也很简单，Spring 为什么提供了基础类 WebMvcConfigurerAdapter ，我们只需要重写 addInterceptors 方法添加注册拦截器。

实现自定义拦截器只需要3步： 
1、创建我们自己的拦截器类并实现 HandlerInterceptor 接口。 
2、创建一个Java类继承WebMvcConfigurerAdapter，并重写 addInterceptors 方法。 
2、实例化我们自定义的拦截器，然后将对像手动添加到拦截器链中（在addInterceptors方法中添加）。 

代码示例：

MyInterceptor1.java
```java
package org.springboot.sample.interceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

/**
 * 自定义拦截器1
 *
 * @author   单红宇(365384722)
 * @myblog  http://blog.csdn.net/catoop/
 * @create    2016年1月7日
 */
public class MyInterceptor1 implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {
        System.out.println(">>>MyInterceptor1>>>>>>>在请求处理之前进行调用（Controller方法调用之前）");

        return true;// 只有返回true才会继续向下执行，返回false取消当前请求
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
            ModelAndView modelAndView) throws Exception {
        System.out.println(">>>MyInterceptor1>>>>>>>请求处理之后进行调用，但是在视图被渲染之前（Controller方法调用之后）");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
            throws Exception {
        System.out.println(">>>MyInterceptor1>>>>>>>在整个请求结束之后被调用，也就是在DispatcherServlet 渲染了对应的视图之后执行（主要是用于进行资源清理工作）");
    }

}
```

MyInterceptor2.java
```java
package org.springboot.sample.interceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

/**
 * 自定义拦截器2
 *
 * @author   单红宇(365384722)
 * @myblog  http://blog.csdn.net/catoop/
 * @create    2016年1月7日
 */
public class MyInterceptor2 implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {
        System.out.println(">>>MyInterceptor2>>>>>>>在请求处理之前进行调用（Controller方法调用之前）");

        return true;// 只有返回true才会继续向下执行，返回false取消当前请求
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
            ModelAndView modelAndView) throws Exception {
        System.out.println(">>>MyInterceptor2>>>>>>>请求处理之后进行调用，但是在视图被渲染之前（Controller方法调用之后）");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
            throws Exception {
        System.out.println(">>>MyInterceptor2>>>>>>>在整个请求结束之后被调用，也就是在DispatcherServlet 渲染了对应的视图之后执行（主要是用于进行资源清理工作）");
    }

}
```

MyWebAppConfigurer.java
```java
package org.springboot.sample.config;

import org.springboot.sample.interceptor.MyInterceptor1;
import org.springboot.sample.interceptor.MyInterceptor2;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

@Configuration
public class MyWebAppConfigurer 
        extends WebMvcConfigurerAdapter {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 多个拦截器组成一个拦截器链
        // addPathPatterns 用于添加拦截规则
        // excludePathPatterns 用户排除拦截
        registry.addInterceptor(new MyInterceptor1()).addPathPatterns("/**");
        registry.addInterceptor(new MyInterceptor2()).addPathPatterns("/**");
        super.addInterceptors(registry);
    }

}
```

然后在浏览器输入地址： http://localhost:8080/index 后，控制台的输出为：
```java
>>>MyInterceptor1>>>>>>>在请求处理之前进行调用（Controller方法调用之前）
>>>MyInterceptor2>>>>>>>在请求处理之前进行调用（Controller方法调用之前）
>>>MyInterceptor2>>>>>>>请求处理之后进行调用，但是在视图被渲染之前（Controller方法调用之后）
>>>MyInterceptor1>>>>>>>请求处理之后进行调用，但是在视图被渲染之前（Controller方法调用之后）
>>>MyInterceptor2>>>>>>>在整个请求结束之后被调用，也就是在DispatcherServlet 渲染了对应的视图之后执行（主要是用于进行资源清理工作）
>>>MyInterceptor1>>>>>>>在整个请求结束之后被调用，也就是在DispatcherServlet 渲染了对应的视图之后执行（主要是用于进行资源清理工作）
```

根据输出可以了解拦截器链的执行顺序（具体原理介绍，大家找度娘一问便知）

最后强调一点：只有经过DispatcherServlet 的请求，才会走拦截器链，我们自定义的Servlet 请求是不会被拦截的，
比如我们自定义的Servlet地址 http://localhost:8080/xs/myservlet 是不会被拦截器拦截的。并且不管是属于哪个Servlet 只要复合过滤器的过滤规则，过滤器都会拦截。





## References

1. [Spring IOC原理总结](https://www.jianshu.com/p/9fe5a3c25ab6)
2. [Spring IoC有什么好处呢？](https://www.zhihu.com/question/23277575)
3. [Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/articles/injection.html)
4. [Dependency Injection](https://en.wikipedia.org/wiki/Dependency_injection)
5. [Spring Boot中使用AOP统一处理Web请求日志](http://blog.didispace.com/springbootaoplog/)
6. [Spring Boot实战之Filter实现简单的Http Basic认证](https://blog.csdn.net/sun_t89/article/details/51916834)
7. [Spring Boot 拦截器](https://blog.csdn.net/catoop/article/details/50501696)

 

 

 

