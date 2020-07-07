### Spring Boot

#### 简介

Spring Boot是Spring组件一站式解决方案，主要是简化了Spring的难度，简化配置（可以不用XML），提供各种启动器，提高开发效率。

#### 核心注解是哪个？

启动类上的@SpringBootApplication，这一个注解就包含了三个注解：@SpringBootConfiguration 实现配置文件的功能；@EnableAutoConfiguration 打开自动配置的功能，也可以关闭某个自动配置的选项；@ComponentScan Spring组件扫描

#### 什么是JavaConfig

JavaConfig提供了配置Spring IOC容器的纯Java方法，有助于避免XML的使用：

1. 面向对象的配置，配置类可以继承默认的配置类，重写它的@Bean方法
2. 减少SML配置，基于依赖注入原则的外化配置具有一定优势
3. 类型安全和重构友好，现在可以按照类型来检测Bean而不是名称，不需要再转化为基于字符串的查找

#### 自动配置原理？Springboot配置的加载顺序？

@EnableAutoConfiguration：这个注解的源码中写了`@Import(AutoConfigurationImportSelector.class)` 这个注解，`自动配置导入选择器？`  的作用就是往 Spring 容器中导入组件。将类路径下 								 `META-INF/spring.factories  `里面配置的所有 EnableAutoConfiguration 的值加入到了容器中。所有在配置文件中能配置的属性都是在 xxxxProperties 类中封装的

@Configuration

@ConditionalOnClass 某个class位于类路径上，才会实例化一个Bean

可以通过一下形式进行配置：

+ Properties文件；
+ YAML文件；
+ 系统环境变量；
+ 命令行参数；--spring.config.location=D:/application.properties  --server.port=8087 

`SpringApplication` loads properties from `application.properties` files in the following locations and adds them to the Spring `Environment`:

1. A `/config` subdirectory of the current directory 当前文件的config子文件
2. The current directory 当前文件
3. A classpath `/config` package 类路径下的config文件夹
4. The classpath root 类路径

#### 核心配置文件是哪个？

bootstrap (. yml 或者 . properties)：boostrap 由父 ApplicationContext 加载的，比 applicaton 优先加载，配置在应用程序上下文的引导阶段生效。一般来说我们在 Spring Cloud Config 或者 Nacos 中会用到它。且 boostrap 里面的属性不能被覆盖；
**application (. yml 或者 . properties)： 由ApplicatonContext 加载。**

#### 什么是 Spring Profiles？

主要用在项目**多环境运行**的情况下，通过激活方式实现多环境切换，省去多环境切换时配置参数和文件的修改。

Application.properties 共有配置

Application-dev.properties 开发环境的配置

Application-prod.properties  生产环境的配置

但是yml文件可以一次性写完不同生产环境的配置：

```yml
spring:
	profiles:
		active: prod
spring:
	profiles: dev
  
  server:
  port: 8080
  
```

profiles的方法主要用于端口号、数据库、安全证书的位置

#### 如何实现 Spring Boot 应用程序的安全性？

为了实现 Spring Boot 的安全性，我们使用 spring-boot-starter-security 依赖项，并且必须添加安全配置。它只需要很少的代码。配置类将必须扩展WebSecurityConfigurerAdapter 并覆盖其方法。


#### 比较一下 Spring Security 和 Shiro ?

由于 Spring Boot 官方提供了大量的非常方便的开箱即用的 Starter ，包括 Spring Security 的 Starter ，使得在 Spring Boot 中使用 Spring Security 变得更加容易，甚至只需要添加一个依赖就可以保护所有的接口，所以，如果是 Spring Boot 项目，一般选择 Spring Security 。当然这只是一个建议的组合，单纯从技术上来说，无论怎么组合，都是没有问题的。Shiro 和 Spring Security 相比，主要有如下一些特点：

Spring Security 是一个重量级的安全管理框架；Shiro 则是一个轻量级的安全管理框架
Spring Security 概念复杂，配置繁琐；Shiro 概念简单、配置简单
Spring Security 功能强大；Shiro 功能简单

#### 跨域问题如何解决？

后端通过CORS，cross-origin resource sharing来解决跨域问题，通过在服务器端设置响应头，把发起跨域的原始域名添加到Access-Control-Allow-Origin 即可。重点放在Response headers，它可以帮助我们在服务器进行跨域授权，例如允许哪些原始域可放行，是否需要携带Cookie信息等。

- 方式1：返回新的CorsFilter
- 方式2：重写WebMvcConfigurer
- 方式3：使用注解（[@CrossOrigin](https://github.com/CrossOrigin)）
- 方式4：手工设置响应头（HttpServletResponse ）
- 如果使用了Nginx，在配置文件中加三行，add_header

#### 什么是 CSRF 攻击？

CSRF 代表跨站请求伪造。这是一种攻击，迫使最终用户在当前通过身份验证的Web 应用程序上执行不需要的操作。CSRF 攻击专门针对状态改变请求，而不是数据窃取，因为攻击者无法查看对伪造请求的响应。

#### 监视器是什么？

spring boot actuator 在应用程序里提供众多 Web 接口，通过它们了解应用程序运行时的内部状况

#### websockets

websockets是一种计算机通信协议，通过单个TCP连接提供全双工通信信道。它是双向的，客户端和服务端都可以发起通讯；与http相比，数据交互要轻得多

#### spring data

spring data是Spring的子项目，用来访问数据库。Spring Data Jpa 通过实现持久层的接口，就能实现一些简单的crud操作。通过方法的名字来确定逻辑！

#### Swagger

可视化API，可以用来维护前后端分离项目的接口文档，前端可以进行在线测试。同步更新接口文档。

```
@EnableSwagger2
```

#### Kafka ActiveMQ 

分布式发布-订阅消息系统

#### 热部署

DEV工具 DEVtools

#### 我使用了哪些maven依赖？



#### springboot的starter？

其实就是提供了自动化配置，根据条件注解、有没有引入依赖等等来提供一些列的默认配置

spring-boot-starter-parent：定义了java编译版本，编码格式，执行打包操作的配置，自动化的资源过滤，自动化的插件配置。

#### springboot打包的jar有什么不同？

它的jar是可执行的jar包，不可以作为普通的依赖包。

#### 异常处理？



#### 微服务的session处理？

spring session + redis，将所有微服务的session都保存在redis上。

#### 定时任务？

@Scheduled注解 + cron表达式

@EnableScheduling



#### 认证和授权问题

##### 认证Authentication-你是谁

cookie和session都是用来跟踪浏览器用户身份的会话方式，但是两者的应用场景不太一样。

**cookie一般存在 客户端，用来保存用户的信息；**

+ 在cookie中保存已经登陆过的用户信息，下次访问网站的时候，页面可以自动填写一些登录的基本信息。除此之外，还能保存用户的首选项，默认主题等设置信息；
+ 使用cookie保存session或者token，向后端请求的时候带上cookie，后端可以获取到session或者token，这样就能记录用户的状态，因为HTTP是无状态的协议；
+ cookie还可以用来记录分析用户行为。

**如何在服务端使用cookie？**

+ new Cookie()，response.addCookie(xx)；设置cookie，并返回给客户端
+ 使用spring框架提供的@CookieValue注解获取特定cookie的值；
+ request.getCookies()，可以获取所有cookie的值；



session是通过服务端来记录用户的状态。服务端给特定的用户创建特定的session之后就可以标识这个用户并跟踪这个用户。相对cookie，session的安全性更高。如果需要在cookie中写入一些敏感信息，最好能讲cookie的信息加密存储，用到的时候在服务端解密。

**如何通过session来进行身份验证：**

+ 用户登录成功之后，返回给客户端具有sessionId的cookie，当用户向后端发起请求的时候会把sessionId带上，而服务端的sessionId存在redis当中。服务端会将收到的sessionId与存储在内存或者数据库中的sessionId信息进行比较，以验证用户的身份，返回给客户端响应信息的时候会附上用户当前的状态。
+ 需要注意，客户端需要开启cookie；
+ session的过期时间



**如果没有开启cookie的话，session还能用吗？**

一般是通过cookie来存放sessionId的，但是cookie只是session的一种实现方案，还可以将sessionId放在url里面。



**为什么cookie无法防止CSRF攻击，但是token可以？**

CSRF跨域请求伪造：别人通过cookie拿到了sessionId之后代替我的身份访问系统。而token一般存储在local storage中，不会被泄露。



**什么是token？**

session不适合移动端使用，token则可以用在移动端。JWT（JSON Web Token）就是这种方式的实现，通过这种方式服务器端就不需要保存session数据，只用在客户端保存服务端返回的token就可以。

JWT本质上是一段签名的JSON格式的数据，有3部分组成：

+ Header：定义了生成签名的算法以及token的类型；
+ PayLoad：用来存放实际需要传递的数据；
+ Signature：服务器通过Payload、Header、和一个密钥使用指定加密算法生成；

服务器通过加密算法生成token，并发送给客户端，客户端将token保存在cookie中或者localstorage中，以后客户端发出的所有请求都会携带这个令牌，token最好放在header中的authorization中。服务端检查token并从中获取用户信息。



##### 授权Authorization-你能干什么

Spring Security默认进行URL访问进行拦截，并提供了验证的登录页面。

springboot中的权限管理，一般来说用spring security，但是也可以用shiro。

shiro需要自定义组件realm：

+ 



**什么事OAuth2.0**

OAuth事一个行业的标准授权协议，主要用于第三方应用获取有限的权限。2.0版本更快，更容易实现。

本质上是一种授权机制，它的最终目的是为第三方应用颁发了一个有时效性的token，使得第三方应用能够通过该token获取相关的资源。

常用于：第三方登录，支付场景和开放平台



##### SSO单点登录

Single sign on 用户登录多个子系统的其中一个就有权访问与其相关的其他系统。

### 源码

#### Springboot的启动流程

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

启动的时候调用SpringApplication的run方法，这个run方法会构造一个SpringApplication的实例，然后再调用这里实例的run方法就表示启动springboot。

```java
public SpringApplication(Object... sources) {
  // sources目前是一个MyApplication的class对象
  initialize(sources); 
}

private void initialize(Object[] sources) {
  if (sources != null && sources.length > 0) {
    this.sources.addAll(Arrays.asList(sources)); 
    // 把sources设置到SpringApplication的sources属性中，目前只是一个MyApplication类对象
  }
  this.webEnvironment = deduceWebEnvironment(); 
  // 判断是否是web程序(javax.servlet.Servlet和org.springframework.web.context.ConfigurableWebApplicationContext都必须在类加载器中存在)，并设置到webEnvironment属性中
  // 从spring.factories文件中找出key为ApplicationContextInitializer的类并实例化后设置到SpringApplication的initializers属性中。这个过程也就是找出所有的应用程序初始化器
  setInitializers((Collection) getSpringFactoriesInstances(
      ApplicationContextInitializer.class));
  // 从spring.factories文件中找出key为ApplicationListener的类并实例化后设置到SpringApplication的listeners属性中。这个过程就是找出所有的应用程序事件监听器
  setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
  // 找出main类，这里是MyApplication类
  this.mainApplicationClass = deduceMainApplicationClass();
}
```

ApplicationContextInitializer，应用程序初始化器，做一些初始化的工作：

ApplicationListener，应用程序事件(ApplicationEvent)监听器：

```java
public ConfigurableApplicationContext run(String... args) {
  StopWatch stopWatch = new StopWatch(); // 构造一个任务执行观察器
  stopWatch.start(); // 开始执行，记录开始时间
  ConfigurableApplicationContext context = null;
  configureHeadlessProperty();
  // 获取SpringApplicationRunListeners，内部只有一个EventPublishingRunListener
  SpringApplicationRunListeners listeners = getRunListeners(args);
  // 上面分析过，会封装成SpringApplicationEvent事件然后广播出去给SpringApplication中的listeners所监听
  // 这里接受ApplicationStartedEvent事件的listener会执行相应的操作
  listeners.started();
  try {
    // 构造一个应用程序参数持有类
    ApplicationArguments applicationArguments = new DefaultApplicationArguments(
        args);
    // 创建Spring容器
    context = createAndRefreshContext(listeners, applicationArguments);
    // 容器创建完成之后执行额外一些操作
    afterRefresh(context, applicationArguments);
    // 广播出ApplicationReadyEvent事件给相应的监听器执行
    listeners.finished(context, null);
    stopWatch.stop(); // 执行结束，记录执行时间
    if (this.logStartupInfo) {
      new StartupInfoLogger(this.mainApplicationClass)
          .logStarted(getApplicationLog(), stopWatch);
    }
    return context; // 返回Spring容器
  }
  catch (Throwable ex) {
    handleRunFailure(context, listeners, ex); // 这个过程报错的话会执行一些异常操作、然后广播出ApplicationFailedEvent事件给相应的监听器执行
    throw new IllegalStateException(ex);
  }
}
```



### 我的项目

copd项目中的springboot中配置的启动类如下：

开启了异步线程、定时任务；新建一个connector，给http用；原来的端口留给https

如果使用了Nginx就不需要在springboot里面配置http/https了，在nginx的config中配置

```java
@EnableAsync
@SpringBootApplication
@EnableScheduling
public class Application {

    @Bean
    public ServletWebServerFactory servletContainer() {
        TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
        tomcat.addAdditionalTomcatConnectors(createStandardConnector());
        return tomcat;
    }

    private Connector createStandardConnector() {
        Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
        connector.setScheme("http");
        connector.setPort(8443);
        connector.setSecure(false);
        return connector;
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

#### 

drools：@configuration在配置类中注入KieContainer；kmodule.xml中配置drl文件；

```java
KieSession kieSession = kieContainer.newKieSession("prmanage-rule");
kieSession.insert(evaluateData);
kieSession.insert(prManager);
kieSession.insert(prEvaluate);
kieSession.fireAllRules();
kieSession.dispose();
```

public class CommonJsonException **extends RuntimeException** 自定义异常错误类，继承runtimeexception

`@ControllerAdvice`是一个特殊的`@Component`，用于标识一个类，这个类中被以下三种注解标识的方法：`@ExceptionHandler`，`@InitBinder`，`@ModelAttribute`，将作用于所有的`@Controller`类的接口上。

**@ExceptionHandler**

统一异常处理，也可以指定要处理的异常类型

```java
@ControllerAdvice
@ResponseBody
public class GlobalExceptionHandler {
   private Logger logger = LoggerFactory.getLogger(this.getClass().getName());

   @ExceptionHandler(value = Exception.class)
   public Result defaultErrorHandler(HttpServletRequest req, Exception e) {
      String errorPosition = "";
      //如果错误堆栈信息存在
      if (e.getStackTrace().length > 0) {
         StackTraceElement element = e.getStackTrace()[0];
         String fileName = element.getFileName() == null ? "未找到错误文件" : element.getFileName();
         int lineNumber = element.getLineNumber();
         errorPosition = fileName + ":" + lineNumber;
      }
      Result result = new Result(ErrorEnum.E_400.getErrorCode(), e.toString() + "  错误位置:" + errorPosition);
      logger.error("异常", e);
      return result;
 }
```

#### 项目的难点有哪些？

规则引擎Drools：为了把慢病管理的规则逻辑和其他业务逻辑分离开来，我们项目选用了Drools规则引擎。用规则引擎的好处在于，如果想修改规则，只要在DRL规则文件里做修改，而不需要改动调用规则的方法。困难的地方在于，规则引擎应用demo并不多，能参考学习的资料太少了。官方文档给的例子太简单了，有些用法只有自己摸索、测试。

跨域问题：本来小程序配合springboot是不需要考虑跨域问题的，但是后来后端还需要和另一个前端配合工作，所以出现了跨域问题。用crossorigin的注解将部分controller做了跨域处理，在origins参数中写了另一个前端的域名，结果小程序不能请求了。因为不知道线上小程序的域名，只好用*来处理了。

#### 项目的亮点？

