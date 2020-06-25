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

