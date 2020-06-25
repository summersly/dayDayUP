## Spring面试问题整理

spring的核心特征：控制反转（IOC）/依赖注入（DI）和面向切面编程（AOP）

通过依赖注入和面向接口实现松耦合；基于切面和管理进行声明式编程，减少样板式代码

通过IOC容器管理POJO对象以及他们之间的耦合关系；通过AOP以动态非侵入的方式增强服务

### 总体

#### 优缺点

Spring的优点就是他的几个特点：能够通过控制反转，将对所有对象的创建和依赖关系的维护交给spring来管理；面向切面的编程以动态非侵入的方式增强服务；声明式事务管理之需要配置无需编程；支持Junit4，通过注解方便测试Spring程序；可以和其他框架结合；对JavaEE种一些难用的API进行了封装（比如说JDBC），降低使用难度。

缺点是作为一个轻量级框架，功能大而全，导致难以入门？？？算什么缺点EXM？

#### 由哪些模块构成？

6个模块：核心容器core/AOP切面和设备支持/数据集成访问/Web/消息Message/Test测试

#### 用了哪些设计模式？

工厂模式：BeanFactory（兵工厂），用来创建对象的实例；

单例模式：Bean默认都是单例

代理模式：AOP用到了JDK的动态代理和CGLIB字节码生成技术；

模版方法：解决代码重复问题：JpaTemplate

观察者模式：Spring种Listener的实现-- ApplicationListener

#### 详细讲一下核心容器模块

Spring通过应用上下文（ApplicationContext）来行使容器的工作，Spring自带了多种类型的应用上下文实现，比如ClassPathXmlApplicationContext就是其中一个。这样一来就把应用的配置和依赖关系交给了容器来处理。ApplicationContext其实是BeanFactory的子类，但是因为BeanFactory太古老，不能支持AOP和Web，所以现在一般用应用上下文来工作。

+ AnnotationConfigApplicationContext；
+ AnnotationConfigWebApplicationContext；
+ ClassPathXmlApplicationContext；
+ FileSystemXmlApplicationContext；
+ XmlWebApplicationContext；

#### Spring框架中有哪些事件-5个

+ 上下文开始事件
+ 更新
+ 停止
+ 关闭
+ 请求处理

#### Spring应用程序包含哪些组件？

Bean配置文件/Bean类/接口/用户程序/AOP

### 什么是IOC？

IOC就是控制反转，把传统由程序代码直接操控的对象的调用权交给容器，通过容器来实现对象组件的装配和管理。

**作用：**方便对象依赖关系的管理；解耦，由容器来维护对象；托管了类的生产过程，特别是用到代理的时候，不用再关心是如何完成代理。

**优点**：降低开发应用的代码量；降低耦合。

**实现原理**：工厂模式+反射机制

```java
f = (Fruit) Class.forName("ClassName").newInstance();
```

**依赖注入方式**：

+ 构造器注入：没有部分注入，不会覆盖setter属性，任意修改都会创建新实例，适用于设置很多属性
+ setter注入：
+ @Autowiered

#### Bean是什么？

Bean就是被容器初始化，装配和管理的对象，一个Bean的定义，通常包括所有配置元数据，如何构造，生命周期和依赖关系。

有哪些方式？XML文件，注解等方式

scope属性可以定义类的作用域，常用的作用域是singleton和prototype，还有request，session

spring中的bean默认是单例，但是并非线程安全的。实际上大部分时候bean都是无状态的，比如说在dao层中，在某种意义上它是安全的。如果在多线程环境下，可以采用ThreadLocal来处理线程安全问题。ThreadLocal就是以空间换时间的方式，为每个线程提供变量副本。

#### 源码解读

在new applicationContext的时候，调用了他的构造方法：config的位置，refresh默认是true；

在构造方法中用了一个refresh方法，这个方法是其核心。其中应用同步锁保证IOC容器创建不被扰乱，最开始它做了一些准备工作，比如说记录启动时间，检查xml配置文件；接下来是关键的obtainFreshBeanFactory，如果有旧的beanfactory就关掉创建新的，解析配置文件中bean定义，并且注册BeanDefinitionHolder，放进beanDefinitionMap中。之后对一些特殊的bean做了处理。然后初始化了所有的单例bean，关键在于一个finishBeanFactoryInitialization的方法，它是abstractApplicationContext.class中的方法。

**ApplicationContext 继承自 BeanFactory，但是它不应该被理解为 BeanFactory 的实现类，而是说其内部持有一个实例化的 BeanFactory（DefaultListableBeanFactory）**

在finishBeanFactoryInitialization方法中，调用了beanFactory.preInstantieteSingletons() 就又回到 DefaultListableBeanFactory 这个类。

在这个方法中，遍历 beanDefinitionMap ，它包含了xml中bean的定义，包括bean名字。根据bean的名字得到bean的配置信息，如果它是单例，并且不是抽象的、不是懒加载的也不是工厂类，就直接创建

通过名字获取配置信息的细节：先从已注册的单例bean中查找，检查是否已经存在；然后获取当前bean的依赖dependsOn，如果有则先注册依赖关系，初始化被依赖对象。

 doCreateBean 中的三个细节：一个是创建 Bean 实例的 createBeanInstance 方法，一个是依赖注入的 populateBean 方法，还有就是回调方法 initializeBean。

DefaulSingletonBeanRegistry的singletonObjects是一个concurrentHashMap，缓存管理单例bean。新创建的单例bean会放进这个map中

#### 自动装配

**xml**中默认不自动装配，通过设置ref来进行装配bean；

通过名字装配；通过type装配；通过构造函数装配；自动探测

**@Autowired**：首先通过类型查找，找到多个类型，再通过名称查找，如果查找结果为空，会抛出异常。required=false，可以用在构造函数，setter方法和成员变量上

**@Resource**：默认是按照名字来装配的，只有找不到同名才会用类型

```xml
<bean id="map" class="xxx">
<!-- 注入空字符串值 -->
   <property name="emptyValue">
       <value></value>
   </property>
   <!-- 注入null值 -->
   <property name="nullValue">  
       <null />
   </property>
</bean>
```

#### 基于注解的配置，举个例子

@Configuration和@Bean

需要在spring的配置文件中配置`<context:annotation-config/>`

@Repository：这个注解是具有类似用途和功能的 @Component 注解的特化。它为 DAO 提供了额外的好处。**它将 DAO 导入 IoC 容器，并使未经检查的异常有资格转换为 Spring DataAccessException。**

#### @Qualifier 注解有什么作用

当您创建多个相同类型的 bean 并希望仅使用属性装配其中一个 bean 时，您可以使用@Qualifier 注解和 @Autowired 通过指定应该装配哪个确切的 bean 来消除歧义。

```java
@Component    
public class FooService {        
  @Autowired        
  @Qualifier("fooFormatter")        
  private Formatter formatter; //如果有两个类都实现了formatter接口，不用qualifier会发生异常
}
```



### JdbcTemplate

Spring 支持集成主流的ORM框架，也可以通过Spring框架提供的模版类JdbcTemplate进行JDBC操作（数据库连接）。

DAO是数据访问对象的意思，通过DAO层，可以使数据访问以一种更统一的方式进行，负责与数据库进行联络的一些任务

#### JdbcTemplate是什么？

提供了很多便利的方法解决诸如把数据库数据转变成基本数据类型或对象，执行写好的或可调用的数据库操作语句，提供自定义的数据错误处理。













