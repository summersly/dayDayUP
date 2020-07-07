# SSM框架

## Spring框架

### IOC（inversion of control）控制反转

**资源的控制方式**：

1. 主动式：需要什么资源，使用new来创建；
2. 被动式：资源的获取交给容器来创建；

容器：管理所有的组件（有功能的类）；控制反转就是主动方式反转到被动，资源被容器管理。

DI（depedency injection）：依赖注入，容器根据需要，利用反射给属性赋值，将对象注入到目的对象中。

```xml
<!-- spring核心5个包 beans context core expression commons-logging
	// src 下 ioc.xml 中注册对象 Person
	// class:全类名 id：类对象的唯一标识 -->
<bean class="com.xxxx.person" id="person01">
  <property name="lastname" value="zhangsan"></property>
  <property name="age" value="18"></property>
</bean>

<!-- 有参构造器注册 -->
<bean class="com.xxxx.person" id="person02">
  <constructor-arg name="lastname" value="xiaoming"></constructor>
</bean>
<!-- 有参构造器注册 index 型式 ，省略name属性，必须按照顺序赋值/或者使用index参数指定位置
  	如果构造函数重载可以用type指定参数类型--> 
<!-- 通过p名称空间为bean赋值，防止标签重复 -->
<bean class="com.xxxx.person" id="person03"
      p:name="zhangsan" p:age=18>     
</bean>
```

```xml
<!-- null赋值 -->
<bean class="com.xxxx.person" id="person02">
  <property name="lastname">
  	<null/>  
  </property>
</bean>
<!-- ref引用 car01是已创建的car类型的bean -->
<bean class="com.xxxx.person" id="person02">
  <property name="car" ref="car01"></propertyr>
</bean>
<bean class="com.xxxx.person" id="person02">
  <property name="car">
  	<bean class="com.xxxx.car"> <!-- 内部bean，只能内部使用，不能外部获取 -->
      <property name="carname" value="baoma"></property>
    </bean>
  </property>
</bean>
<!-- 复杂的属性赋值 list map-->
<bean class="com.xxxx.person" id="person02">
  <property name="books">
    <list>
      <bean id="book001" class="com.xxxx.book" p:bookname="xiyouji"></bean>
      <ref bean="book002"/> <!-- 引用外部bean -->
    </list>
  </property>
  <!-- ===========================================  -->
  <property name="maps">
     <!-- new linkedhashmap<> -->
    <map>
      <entry key="key01" value="zhangsan"></entry>
      <entry key="key02" value-ref="book01"></entry>  <!-- 引用外部bean -->
      <entry key="key03">
      	<bean id="book001" class="com.xxxx.book" p:bookname="xiyouji"></bean>
      </entry>
    </map>
  </property>
  <!-- properties 对象赋值 =========================  -->
  <property name="properties">
    <props>
      <prop key="username">zhangsan</prop>
      <prop key="password">123456</prop>
    </props>
  </property>
</bean>
```

```xml
<!-- util名称空间建立集合类型的bean，方便引用  -->
<bean id="person03" class="com.xxx.person">
  <propety name="maps" ref=mymap></proterty>
</bean>
<util:map id="mymap">
      <entry key="key01" value="zhangsan"></entry>
      <entry key="key02" value-ref="book01"></entry>  <!-- 引用外部bean -->
</util:map>

<!-- 级联属性   -->
<bean id="person03" class="com.xxx.person">
  <propety name="maps" ref=mymap></proterty>
</bean>
<util:map id="mymap">
      <entry key="key01" value="zhangsan"></entry>
      <entry key="key02" value-ref="book01"></entry>  <!-- 引用外部bean -->
</util:map>

<!-- 通过继承 parent 实现配置信息的重用   -->
<bean id="person04" class="com.xxx.person" parent="person02">
  <!-- 只覆盖部分值   -->
  <propety name="lastname" value="lisi"></proterty>
</bean>

<!-- abstract 抽象，只能被继承  depends-on 改变创建顺序 -->
<bean id="person05" class="com.xxx.person" abstract="true" depends-on="books">
  <propety name="lastname" value="lisi"></proterty>
</bean>
```

```java
// person对象是什么时候创建的？
// 在容器创建的时候，容器中对象就已经创建
// 同一个组件在IOC容器中是单实例的
// javaBean的属性名是由get/set决定的，而不是域的定义
ApplicationContext ioc = new ClassPathXmlApplicationContext("ioc.xml");
Person bean =  ioc.getBean("person01");
// 使用类型找，有两个同类型的就会报错；
Person person = ioc.getBean(Person.class);
// 也可以通过id和类型一起查找
Person person = ioc.getBean("person01"，Person.class);
// 
```

```xml
<!-- bean 的作用域 scope：
 			prototype：多实例，被获取时才被创建
			singleton：单实例（默认）容器启动时创建
			request：在web环境下，同一次请求创建一个
			session：在web环境下，同一次会话创建一个 
-->
<bean id="person04" class="com.xxx.person" scope="prototype">
  <propety name="lastname" value="lisi"></proterty>
</bean>

<!-- bean的默认创建框架利用反射new出来的对象
		工厂模式：实例有工厂（一个类,不需要继承或实现接口！！）来创建；
						静态工厂：不需要创建工厂实例，直接调用静态方法；
						实例工厂：先创建工厂实例对象，工厂对象调用方法；-->
<!-- 1 静态工厂 class指定工厂 factorymethod指定方法名 —-->
<bean id="airplane1" class="com.xxx.airplaneStaticFactory" factory-methos="getAirplane">
	<!-- 为方法指定参数 —-->
  <constructor-arg value="lisi"></constructor-arg>
</bean>

<!-- 2 实例工厂 先注册工厂类，再指定工厂类和方法注册类 —-->
<bean id="airplaneFactory" class="com.xxx.airplaneFactory"></bean>
<bean id="airplane2" class="com.xxx.airplane" 
      factory-bean="airplaneFactory" factory-methos="getAirplane">
	<!-- 为方法指定参数 —-->
  <constructor-arg value="lisi"></constructor-arg>
</bean>

<!-- 实现了FactoryBean接口的实现类，spring自动识别为工厂类（和上面两个工厂不一样）
  	容器启动时不会！！创建对象
airplaneFactoryimpl implement FactoryBean—-->
<bean id="airplaneFactoryimpl" class="com.xxx.airplaneFactoryimpl"></bean>
```

```xml
<!-- 带有生命周期方法的bean 在class指定的类中写初始化和销毁函数，并在bean中指定函数 —-->
<!-- 单实例singleton模式：容器启动、构造器--初始化函数--容器关闭、销毁函数
 		 多实例prototype模式：容器启动--获取bean、构造器--初始化函数--容器关闭不会掉用销毁函数 ！！-->
<bean id="airplaneFactoryimpl" class="com.xxx.airplaneFactoryimpl"
      init-method="myinit" destroy-method="mydestroy"></bean>

<!-- bean的后置处理器BeanPostProcessor 接口 有两个方法：postProcessBeforeInitialiazation(object,string);
postProcessAfterInitialiazation(Object,String);  
class属性指向该接口的实现类 -->
<bean id="BeanPostProcessor" class="com.xxx.BeanPostProcessorImpl"></bean>

<!-- 引用外部属性文件 -->
<context:property-placeholder location="classpath:dbconfig.properties"/>
<bean id="dataSource" class="com.xxx.c3p0.comboPooledDataSource">
  <property name="user" value="${jdbc.username}"></property>
</bean>

<!-- 基于xml的自动装配 仅限自定义类型
 autowired：default/no ：不自动装配
						byName：以属性名作为id去查找bean，没找到就是null
						byType：以类型class去查找，找到多个会抛异常，没找到null
						constructor：找到有参构造器，按照类型查找，再按照参数名作为id进行查找 -->
```

src源码包开始的路径，称为类路径；所有源码包里的东西都会被合并到类路径里面；

一般的java项目：/bin/

Java web项目：/WEB-INF/classes/



**通过注解依赖注入：**

1. 给要添加的组件上标注注解@componet等；
2. 利用context名称空间，在xml中设置自动扫描；
3. 组件注入默认的id是类名首字母小写，@componet（“别的名字”）；默认是单例的，@scope（value=“prototype”）；
4. 需要导入AOP包，以支持注解！

```xml
<context:component-scan base-package="com.zjubiomedit" [use-default-filters="false"]>
	<!-- 扫描的时候可以排除一些不要的组件:
 			 type = "annotation" 排除特定注解，需要写出该注解的全类名
			 type = "assignable" 排除某个具体类，类的全类名         
			 type = "aspectj" aspectj表达式 
			 type = "custom" 自定义一个typefilter，决定过滤哪些
			 type = "regex" 正则表达式                        -->
  <context:exclude-filter type="annotation" 
                          expression="org.???.controller"></context:exclude-filter>
  	<!-- 指定扫描哪些组件: use-default-filters="false"  -->
  <context:include-filter type=""
                          expression=""></context:include-filter>
</context:component-scan>
```

5. @Autowired：默认先用类型查找，如果找多多个匹配，则用id进行查找；也可以通过@Qualifier("newid")指定id；也可以@Autowired(required=false)指明找不到就拉倒；

   autowired也可以用在方法上；

   @autowired ，@resource，@inject 都是自动装配的注解

   

### AOP（aspect oriented programming）面向切面编程

在程序运行中，将某段代码动态切入到某个方法的某个位置进行执行的编程方式。

**横切关注点：**一些具有横越多个模块的行为，每个方法影响应用多处的功能（安全、事务、日志）；

**通知方法**：在横切关注点调用的方法，叫做通知方法；

**切面类**：横切关注点和通知方法构成；横切关注点被模块化为特殊的类，这些类称为切面；

**连接点**：每个方法的每个位置都是一个连接点，一个应用执行过程中能够插入一个切面的点；

**切入点**：连接点真的被切入（感兴趣）的一部分；

**切入点表达式**：在众多连接点中选出需要切入的地方；



**proxy代理对象**（反射包下的）：JDK默认的动态代理，如果对象没有实现接口，是无法为他创建代理对象的

```java
// 使用newProxyInstance方法建立动态代理对象，能够代替原对象执行方法，且该方法被增强了
ClassLoader loader = 被代理对象.getClass().getClassLoader();
Class<?>[] interfaces = 被代理对象.getClass().getInterfaces();
InvocationHandler invocationHandler = new InvocationHandler(){
  //匿名对象
  @override
  public Object invoke(Object proxy, Method method, Object[] args){
    // 除了执行方法的其他增强操作
    return method.invoke(被代理对象，args)；
  }
}
Proxy.newProxyInstance(loader, interfaces, invocationHandler);
```



**spring中的AOP：**

> 基于注解

1. 导包：spring-aspects-xxxx.jar  （aop基础包），cglib/aopalliance/aspectj.weaver （aop加强版，即时目标对象没有任何接口也能实现动态代理）

2. 注解1：@aspect @component 切面类；

3. 注解2: 什么方法在什么时候执行，前置通知@before：目标方法之前；后置通知@after：目标方法执行之后；返回通知@afterReturning：目标方法正常返回之后；异常通知@afterThrowing：目标方法抛出异常之后；环绕通知@Around：环绕；

4. ```java
   @Before(execution([public] int com.xxxx.impl.xxxx.*(int ,int )))
   ```

5. 开启基于注解的aop：

   ```xml
   <aop:aspectj-autoproxy> </aop:aspectj-autoproxy>
   ```

> 基于xml

```xml
<!-- 基于配置xml的AOP -->
<bean id="logutil" class="com.xxx.logutil"></bean>
<bean id="mycalculator" class="com.xxx.logutil"></bean>
<aop:config>
  <aop:aspect ref="logutil">
    <aop:pointcut expression="execution(* com.xxx.impl.*(...))" id="mypc">
  	<aop:before method="logStart" pointcut="execution(*)"/>
    <aop:after method="logEnd" pointcut-ref="mypc" returning="result"/>
  </aop:aspect>
</aop:config>
```

*注意点*：

1. spring aop的底层动态代理，ioc容器中加入的是代理对象$proxy12，不是本类的类型；

2. **cglib为没有接口的对象实现代理** 

3. 切入点表达式的写法：

   *：通配符，匹配任意多个参数；或者任意类型参数；

   .. : 任意多个参数，任意类型；或者匹配任意多层路径 com.xxxx..xxxx.*

   访问权限：不写默认 public，也只能是public

   ```java
   execution(* *(..)) //任意包下任意方法
   ```

4. 通知方法的执行顺序：before--after--afterreturning 或者afterthorwing

5. JoinPoint对象获取参数getArgs(); 获取签名或者方法名：getSignature().getName();

6. ```java
   @AfterReturning(value="execution(xxx)" returning="result") //result 是返回参数名
   public static void logReturn(JoinPoint jp, Object result){}
   
   @AfterThrowing(value="execution(xxx)" throwing="exception") //exception 是异常参数名
   public static void logException(JoinPoint jp, Exception exception){}
   ```

7. spring 对通知方法的约束并不严格，唯一要求**方法的参数列表**，如上对未知参数进行说明；

8. 抽取可重用切入点表达式：@pointcut

   ```java
   @PointCut("execution(xxx)")
   public void myponitcut(){};
   // 使用抽取的表达式
   @Before("myponitcut()")
   @After(value="myponitcut()")
   ```

9. 环绕通知：

   ```java
   @Around("execution(xxx)")
   public Object myAround(ProceedingJoinPoint pjp) throws Throwable{
   	Object[] obj = pjp.getArgs();
   	Object proceed = null;
   	try{
   		// 前置
   		proceed = pjp.proceed(args);
   		// 返回
   	} catch(Exception e){
   		// 异常
   	} finally{
   		// 后置
   	}
   }
   ```

10. 多个切面之间的运行顺序：



**AOP的使用场景**

1. 记录日志，保存到数据库；
2. 环绕通知，权限验证；
3. 事务控制；
4. 注解方便；配置功能完善；



### JdbcTemplate

3个需要的jar包：spring-jdbc-xxx；spring-orm-xxx；spring-tx-xxx；

```xml
<!-- 配置数据源 -->
<context:property-placeholder location="classpath:dbconfig.properties"/>
<bean id="dataSource" class="com.xxx.c3p0.comboPooledDataSource">
  <property name="user" value="${jdbc.username}"></property>
</bean>
<!-- 注入jdbcTemplate -->
<bean id="jdbcTemplate" class="...jdbcTemplate">
	<constructor-arg name="dataSource" ref="dataSource"></constructor-arg>
</bean>
```



### 声明式事务tx

- 声明式事务：告诉spring哪个是事务方法，spring自动控制
- 编程式事务：transactionFilter{ try {} catch(){} }

事务：逻辑上紧密关联形成整体，要么都执行，要么都不执行。

事务管理代码的固定模式可以作为一种**横切关注点**，可以通过AOP方法模块化，进而借助spring AOP框架实现声明式事务管理。

这个切面已经存在，称为事务管理器。PlatFormTransactionManager接口

**有事务的业务逻辑，IOC保存的是其代理对象**

```xml
<!-- 配置数据源 -->
<context:property-placeholder location="classpath:dbconfig.properties"/>
<bean id="dataSource" class="com.xxx.c3p0.comboPooledDataSource">
  <property name="user" value="${jdbc.username}"></property>
</bean>
<!-- 配置事务管理器，包含数据源 -->
<bean id="tm" class="org.springframwork.jdbc.datasource.DataSourceTransactionManager">
  <property name="dataSource" ref="dataSource"></property>
</bean>
<!-- 1.  基于注解@transactional的事务控制模式，tx名称空间，指定是哪个事务管理器 -->
<tx:annotation-driven transaction-manager="tm"/>

<!-- 2.  基于xml配置的事务控制，类似配置切面 -->
<bean id="bookService" class="com.xxx.bookService"></bean>
<aop:config>
	<aop:pointcut id="txPoint" expression="execution(* com.xxx.impl.*(...))"/>
  <!-- 事务建议/事务增强 -->
  <aop:advisor advice-ref="myadvice" pointcut-ref="txPoint" />
</aop:config>
<tx:advice id="myadvie" transaction-manager="tm">
	<tx:attributes>
     <!-- 指明哪些方法是事务方法，切入点表达式只是说明切入了该方法，但不一定加事务 -->
    <tx:method name="checkout" propagation="REQUIRED" timeout="3"/>
  </tx:attributes>
</tx:advice>
```

```java
@Transactional
// 注解的属性：
/*
	isolation: isolation 事务隔离级别；
	propagation: propagation 事务的传播行为；
	传播+行为：即当一个事务被另一个事务方法调用了，该如何传播？
	required * 事务属性来源于大事务！！自己改无效
	requires_new * 开启新事务；
	** 在同一个类中直接调用本类中两个事务，实际上只是一个事务
	supports
	not_supported
	mandatory
	never
	nested
	===============================================
	异常分类：
		运行时异常：不用throw，默认回滚；
		编译时异常：抛出异常，默认不回滚，非检查异常；
	noRollbackFor: class[] 哪些异常事务可以不回滚；{null..exception.class}
	noRollbackForClassname: String[]
	rollbackFor: class[] 哪些异常事务需要回滚
	rollbackForClassname: String[]
	
	readOnly: boolean; 设置事务为只读事务，默认false，改为true可以进行事务优化，加快查询速度
	timeout: int 超时时间，超过之后自动停止并会滚，单位（秒）
*/
```

<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200312095554781.png" alt="image-20200312095554781" style="zoom:80%;" />



### spring-IOC-AOP源码

IOC容器的启动过程？什么时候创建单实例bean？如何管理这些单实例bean？

```java
ApplicationContext ioc = new ClassPathApplicationContext(configlocation);
=> //new调用三个参数的构造器
public ClassPathApplicationContext(String[] {configlocation}, 
                                   boolean refresh = true, 
                                   ApplicationContext parent){
  //构造方法中有一个refresh()方法，创建单实例
  if (refresh){
    refresh();
  }
}
=> //refresh()方法
{
  // 同步锁，保证多线程情况下IOC容器被创建一次
  // beanFactory 一个接口，创建bean工厂，解析xml配置文件并保存
  // initMesssageSource() 国际化功能
  finishBeanFactoryInitlization();
}
=> //finishBeanFactoryInitlization(); 它是abstractApplicationContext.class中的方法
{
  beanFactory.preInstantieteSingletons();
}

=> //preInstantieteSingletons();  defaultListableBeanFactory.class
{
  // beanDefinitionMap 包含了xml中bean的定义，包括bean名字
  for(String beanName: beanNames){
    // 根据bean名字拿到bean的配置信息
    // bean不是抽象、是单例、不是懒加载的，也不是工厂bean就创建getBean(beanName)
    getBean(beanName);
  } 
}
=> // getBean(beanName) 
  ==> doGetBean(name,null,null,false){ //在AbstractBeanFactory中
  // 先从已注册的单实例bean中找，是否已存在？
  // 获取depends on 当前bean所依赖的bean，如果有则循环创建
  // 如果是单例的，创建bean实例
  sharedInstance = getSingleton(beanName, objectFactory);
};
=> getSingleton(beanName, objectFactory){
  //singletonObjects 是一个concurrentHashMap，缓存管理单实例
  Object singletonObject = this.singletonObjects.get(beanName);
  // 如果当前singletonobject不存在，则使用工厂创建，并加入到map中
  singletonObject = singletonFactory.getObject();
  addSingleton(beanName,SingletonObject);
}
// 创建好的对象，最终会保存在DefaulSingletonBeanRegistry的singletonObjects中
```

**beanFactory和ApplicationContext的区别**：

ApplicationContext是beanFactory的子接口；

beanFactory是bean的工厂接口，负责创建bean实例；spring最底层的接口；

 applicationContext容器接口，更多负责容易功能的实现；在beanFactory创建好的bean之上实现容器功能；

spring中最大的模式就是工厂模式！！



## SpringMVC

springMVC是spring为展现层提供的基于MVC设计理念的web框架，是目前最主流的MVC框架之一。它能通过一套MVC注解，使POJO成为处理请求的控制器，但并不需要继承任何接口。支持REST风格的URL请求。提供了松散耦合的可插拔组件结构，比其他MVC框架更具扩展性和灵活性。

springMVC的MVC理念特殊性在于，前端控制器。

<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200312112602044.png" alt="image-20200312112602044" style="zoom:50%;" />



```java
// springMVC需要的jar包
// 核心容器jar包
commons-logging
spring-aop
spring-bean
spring-core
spring-context
spring-expression
// web需要的jar包
spring-web
spring-webmvc

```

```xml
<!-- web.xml  -->
<!-- web.xml中添加 前端控制器 -->
<servlet>
	<servlet-name>springDispatcherServlet</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <!-- springmvc配置文件的位置 -->
  <init-param>
  	<param-name>contextConfigLocation</param-name>
    <param-value>classpath:springmvc.xml</param-value>
  </init-param>
  <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
	<servlet-name>springDispatcherServlet</servlet-name>
  <!-- /* 拦截一切，包括*.jsp
 			 /  不拦截jsp					-->
  <url-pattern>/</url-pattern>
</servlet-mapping>

```

```xml
<!-- springmvc.xml -->
<!-- 注解包扫描 -->
<context:component-scan base-package="com.xxxx"></context:component-scan>
<!-- 视图解析器 拼接页面地址 -->
<bean class="org.springframwork.web.servlet.view.InternalResourceViewResolver">
  <property name="prefix" value="/WEB-INF/pages"></property>
  <property name="suffix" value=".jsp"></property>
</bean>
```



### 基本流程

1. 客户端发送请求；
2. 请求来到tomcat服务器；
3. springMVC前端控制器收到所有请求；
4. 看请求地址和@requestMapping标注的哪个方法匹配；
5. 前端控制器找到了目标处理器类和目标方法，直接利用返回执行目标方法；
6. 方法执行完成以后会有一个返回值；springMVC认为这个返回值就是要去的页面地址；
7. 拿到返回值后，视图解析器拼接完整的页面地址；
8. 拿到页面地址，前端控制器转发到页面；

*** 细节问题：

- @RequestMapping("/hello") 和 @RequestMapping("hello") 都可以
- 前端控制器的参数不配置时，默认是/WEB-INF/springDispatcherServlet-servlet.xml
- Url-pattern：default  jspservlet



### @RequestMapping

可以标在类或者方法上；

**method属性**：限定请求方式 ，默认是可以接受任意方式；

​	method=requestMethod.POST/GET/... 

​	error 405：请求方式错误

**params属性**：规定请求参数；

​	params={"username"}  必须携带username

​	params={"!username"}  不能带username参数

​	params={"username=123"} 必须带并且为123

​	 {"username!=123"}  不带username参数，或者带参数但不是123

​	error 404 :没有使用规定参数

**header属性**：规定请求头；

​	header={"user-agent=xxxxxxxxxx"}

**consumes属性**：规定请求头中的content-type

**produce属性**：给响应中加上content-type

**value属性**：value="/hello" 也可以模糊匹配

​	通配符： ？——能代替任意一个字符； * —— 能代替任意多个字符，或者一层路径； ** ——能代替多层路径

​	多个匹配情况下，精确路径优先；？优先于* ；



### @PathVariable

url中的{xxx}占位符能够绑定到方法参数中

```java
@RequestMapping(value="/user/{id}")
public void getUser(@PathVariable("id")String id){}

```

​		

### RESTful

REST：representational state transfer 资源表现层状态转化，是一种互联网软件架构。希望以简洁的url地址来发送请求，用请求方式来区别增删改查。

资源（resource）：由一个url标识；

表现层（representational）： 资源呈现出来的形式；

状态转化（state transfer）：http是无状态的，所有状态都在服务端，所以转化是建立在表现层上的，即指GET/POST/DELETE/PUT四个操作方式

**springMVC对DELETE和PUT的支持方式**：

1. 在web.xml中配置Filter，能拦截所有请求，并将假装post的DELETE和PUT恢复正常；

   HidenHttpMethodFilter ——  url-pattern：/*

2. 前端发送POST请求，携带参数_method=DELETE/PUT;

3. 8.0级tomcat里面的jsp不允许DELETE和PUT形式的请求，jsp中加上isErrorPage=true。



### 请求处理

#### 1. @RequestParam

springMVC默认方式获取请求参数，方法的参数取相同的名字，就自动获取。

@RequestParam("user") String username //默认必须携带

​	**value属性**：参数的key

​	**required属性**：是否必须存在

​	**defaultValue属性**：默认值



#### 2. @RequestHeader

获取请求头中的信息

@RequestHeader("user-agent") String useragent

属性同上！！



#### 3. @CookieValue

获取某个cookie值；

```java
// 原生获取
cookie：cookies[]  cookies = request.getCookies();
for(Cookie c: cookies){
	if(c.getName().equals("xxxx")){
	 // do something
	}
}
// springMVC
@CookieValue("xxxx")String sessionid
// 属性同上！！
```

 

#### 4. POJO自动封装

如果请求参数是POJO类型，那么springMVC会自动为它赋值；还可以级联封装



### 输出处理

1. 在方法处传入参数model/map/modelMap，addAttribute(string , obj);所有数据都会保存在请求域request中

   model，spring中的接口

   map，jdk中的接口

   modelMap，linkedhashmap的实现类

   map==> modelmap==>

   ​											extendedModelMap ==> BindingAwareModelMap

   Model ============ >

2. 返回值ModelAndView，放在请求request域中

   ModelAndView mv = new ModelAndView("viewname"); //视图名，jsp的名字

   mv.addObject("string",obj);

3. springMVC临时给session域中保存数据的方式：@sessionAttributes(value="msg") 标在类上

   BindingAwareModelMap保存数据的同时给session中也放一份

4. @ModelAttribute



### SpringMVC源码

**前端控制器DispatcherServlet**

```java
// 继承关系HttpServlet ==> HttpServletBean  ==>  FrameworkServlet
// doGet/doPost ==> processRequest(request, response) ==> doService ==> doDispatch(req, res) 
protected void doDispatch(HttpServletRequest request, HttpServletResponse response){
  ModelAndView mv =null;
  // 检查是否是文件上传请求
  processedRequest = checkMultiPart(request);
  // 根据当前请求，找到哪个类(处理器controller)能来处理该请求
  mappedHandler = getHandler(processedRequest);
  // 拿到能执行这个类所有方法的适配器(一个反射工具：annotationMethodHandlerAdapter)
  HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
  // 适配器来执行目标方法，将目标方法执行后的返回值作为视图名，保存到modelandview中
  mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
  // 根据方法最终执行完成后封装的modelandview，转发到对应页面，并且mv中的数据可以在请求域中获取
  processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
}
// getHandler细节：怎么根据当前请求找到处理器？
// 返回值mappedHandler类型为HandlerExcutionChain,目标处理器的执行链
// requestDispatchPath: 当前请求的路径，例如“/hello”
protected HandlerExcutionChain getHandler(HttpServletRequest request){
  for(HandlerMapping hm: this.handlerMappings){
    // 遍历所有HandlerMapping查找包含当前请求路径的处理器
    // HandlerMapping处理器映射，其中有handlerMap（一个linkedHashMap）保存了哪个请求对应哪个处理器
  }
}
// getHandlerAdapter细节
// 同上，遍历所有handlerAdapters
```

**springMVC九大组件**：dispatchServlet中的9个引用属性

springMVC在关键位置的功能，都是由这些组件完成的。

1. multipartResolver：文件上传解析器
2. localeResolver：区域信息解析器，国际化有关
3. themeResolver：主题解析器
4. handlerMapping：映射信息
5. handleAdapter：适配器
6. List<HandlerExceptionResolver>：异常解析器
7. requestToViewNameTranslator
8. flashMapManager：重定向携带数据的功能
9. viewResolver：视图解析器



**执行目标方法**：反射执行method.ivoke(this, args)，如何确定方法的参数值



### 视图解析ViewResolver

方法执行的返回值会被当作视图名称：

1. 相对路径的方法：../../hello.jsp
2. forward:/hello.jsp 转发（服务端的）
3. redirect:/hello.jsp 重定向（客户端的）

请求处理方法完成后，都会返回一个modelandview对象。springMVC借助视图解析器得到最终的view对象（视图对象）。对于最终采取何种视图对象对模型数据进行渲染，处理器并不关心，处理器只关心生产模型数据。视图是无状态的，所以不会发生多线程安全问题。

**视图解析的原理**

```java
// 适配器来执行目标方法，将目标方法执行后的返回值作为视图名，保存到modelandview中
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
// 根据方法最终执行完成后封装的modelandview，转发到对应页面，并且mv中的数据可以在请求域中获取
processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
// ==> 在processDispatchResult方法中，render()渲染页面
render(mv, request, response){
  // viewResolver根据方法返回名得到view对象
  // view对象的render方法
}
```





## MyBatis

### xml配置

Mybatis 有两个重要的xml文件，一个是配置文件，一个是映射文件；

其中配置文件在SSM的整合中可以放在spring的xml文件中进行配置，创建bean对象名为" org. mybatis. spring. SqlSessionFactoryBean"，该对象的property中可以配置数据源和映射文件位置等等，等效于独立的xml配置文件。

**configuration（配置）**

- properties（属性）
- settings（设置）
- typeAliases（类型别名）
- typeHandlers（类型处理器）
- objectFactory（对象工厂）
- plugins（插件）
- environments（环境配置）
  - environment（环境变量）
    - transactionManager（事务管理器）
    - dataSource（数据源）
- databaseIdProvider（数据库厂商标识）
- mappers（映射器）

**properties & setting**

通过**方法参数**传递的属性具有最高优先级，resource/url 属性中指定的配置文件次之，最低优先级的是 properties 属性中指定的属性。mybatis读取属性的顺序与优先级顺序相反。

Settings中定义了mybatis的行为设置。

```xml
<configuration>
	<properties resource="org/mybatis/example/config.properties">
    <!-- environment中的datasource就可以用这里定义的属性 -->
  	<property name="username" value="dev_user"/>
  </properties>
  <settings>
  	<setting name="cacheEnabled" value="true"/>
  	<setting name="lazyLoadingEnabled" value="true"/>
  	<setting name="multipleResultSetsEnabled" value="true"/>
  	<setting name="useColumnLabel" value="true"/>
  	<setting name="useGeneratedKeys" value="false"/>
  	<setting name="autoMappingBehavior" value="PARTIAL"/>
  	<setting name="defaultExecutorType" value="SIMPLE"/>
  	<setting name="defaultStatementTimeout" value="25"/>
  	<setting name="safeRowBoundsEnabled" value="false"/>
  	<setting name="mapUnderscoreToCamelCase" value="false"/>
  	<setting name="localCacheScope" value="SESSION"/>
  	<setting name="jdbcTypeForNull" value="OTHER"/>
  	<setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
  </settings>
</configuration>
```

**typeAliases**

可以给类型全类名换个短名字；

也可以指定包名，那么这个包下的所有bean都会使用别名，即首字母小写的bean；

```xml
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author"/>
</typeAliases>

<typeAliases>
  <package name="domain.blog"/>
</typeAliases>
```

**typeHandlers**

你可以重写类型处理器或创建你自己的类型处理器来处理不支持的或非标准的类型。 具体做法为：实现 `org.apache.ibatis.type.TypeHandler` 接口， 或继承一个很便利的类 `org.apache.ibatis.type.BaseTypeHandler`， 然后可以选择性地将它映射到一个 JDBC 类型

<u>使用typeHandlers来映射枚举类型！</u>

**objectFactory**

MyBatis 每次创建结果对象的新实例时，它都会使用一个对象工厂（ObjectFactory）实例来完成。

**plugins**



**environments**

可以配置多个环境，但是每个sqlsessionfactory都只能选择一种环境；

environment中有两个属性：transactionManager 和dataSource；

其中transactionManager有JDBC和MANAGED两个可选值，一般选JDBC，使用了JDBC的提交和回滚。如果使用 Spring + MyBatis，则没有必要配置事务管理器，因为 Spring 模块会使用自带的管理器来覆盖前面的配置。

dataSource有三个可选项：UNPOOLED/POOL/JNDI

**databaseIdProvider**

用于数据库移植，有点蠢一般不用

```xml
<databaseIdProvider type="DB_VENDOR">
  <property name="SQL Server" value="sqlserver"/>
  <property name="DB2" value="db2"/>
  <property name="Oracle" value="oracle" />
</databaseIdProvider>
```

**mapper**

```xml
<!-- mybatis-config.xml 整体配置文件 -->
<configuration>
  <environments default="development">
    <environment id="development">
    	<transactionManager type="JDBC"/>
      <dataSource type="POOLED">
      	<property name="driver" value=${driver}/>
      </dataSource>
    </environment>
  </environments>
  
  <mappers>
  	<mapper resource="mybatis/StudentDao.xml"/>
    <mapper url="file:///var/mappers/AuthorMapper.xml"/>
    <mapper class="org.mybatis.builder.PostMapper"/>
    <package name="org.mybatis.builder"/>
  </mappers>
</configration>
```

### xml映射文件

- `cache` – 该命名空间的缓存配置。
- `cache-ref` – 引用其它命名空间的缓存配置。
- `resultMap` – 描述如何从数据库结果集中加载对象，是最复杂也是最强大的元素。
- `sql` – 可被其它语句引用的可重用语句块。
- `insert` – 映射插入语句。
- `update` – 映射更新语句。
- `delete` – 映射删除语句。
- `select` – 映射查询语句。

```xml
<!-- 类的映射 -->
<!-- 接口的全类名 -->
<mapper namespace="com.zjubiomedit.studentDao">
  <!-- id：方法名 #{参数} -->
  <select id="getStuById" resultType="com.zjubiomedit.entity.student">
  	select * from Student where id = #{id}
  </select>
  
  <select
 	 	id="selectPerson"
   	parameterType="int"
  	parameterMap="deprecated"
  	resultType="hashmap"
  	resultMap="personResultMap"
  	flushCache="false"
  	useCache="true"
  	timeout="10"
  	fetchSize="256"
  	statementType="PREPARED"
  	resultSetType="FORWARD_ONLY">
  </select>
  
  <insert
  id="insertAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  keyColumn=""
  keyProperty="id"
  useGeneratedKeys="true"
  timeout="20">
  </insert>
</mapper>
```

**#{} VS ${}**

 `${column}` 会被直接替换，拼在sql语句上，不安全；常用于代替表名和字段名；

而 `#{value}` 是一种预编译的方式，会使用 `?` 预处理；常用于处理参数。

**结果映射resultMap**

- `constructor` -- 用于在实例化类时，注入结果到构造方法中
  - `idArg` - ID 参数；标记出作为 ID 的结果可以帮助提高整体性能
- `arg` - 将被注入到构造方法的一个普通结果
- `id` – 一个 ID 结果；标记出作为 ID 的结果可以帮助提高整体性能
- `result` – 注入到字段或 JavaBean 属性的普通结果
- `association` 关联；\<bean>
- 嵌套结果映射 – 关联可以是 `resultMap` 元素，或是对其它结果映射的引用
- `collection`– 集合 List\<bean>
- 嵌套结果映射 – 集合可以是 `resultMap` 元素，或是对其它结果映射的引用
- `discriminator` – 使用结果值来决定使用哪个 `resultMap`
  - case 嵌套结果映射 – `case` 也是一个结果映射，因此具有相同的结构和元素；或者引用其它的结果映射

```xml
<resultMap id="userResultMap" type="User">
  <!-- 主键 也可以用result -->
  <id property="id" column="user_id" />
  <!-- 普通列 -->
  <result property="username" column="user_name"/>
  <result property="password" column="hashed_password"/>
</resultMap>
```

**自动映射**

当自动映射查询结果时，MyBatis 会获取结果中返回的列名并在 Java 类中查找相同名字的属性（忽略大小写）。 这意味着如果发现了 *ID* 列和 *id* 属性，MyBatis 会将列 *ID* 的值赋给 *id* 属性。

通常数据库列使用大写字母组成的单词命名，单词间用下划线分隔；而 Java 属性一般遵循驼峰命名法约定。为了在这两种命名方式之间启用自动映射，需要将 `mapUnderscoreToCamelCase` 设置为 true。

```java
// SqlSessionFactoryBuilder
    SqlSessionFactory build(InputStream inputStream);
    SqlSessionFactory build(InputStream inputStream, String environment);
    SqlSessionFactory build(InputStream inputStream, Properties properties);
    SqlSessionFactory build(InputStream inputStream, String env, Properties props);
    SqlSessionFactory build(Configuration config);
// SqlSessionFactory只需要创建一次。
String resource = "org/mybatis/builder/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory factory = new SqlSessionFactoryBuilder().builder.build(inputStream);
     
// 创建session的方法，sqlsession，相当于connection
    SqlSession openSession();
    SqlSession openSession(boolean autoCommit)
    SqlSession openSession(Connection connection)
    SqlSession openSession(TransactionIsolationLevel level)
    SqlSession openSession(ExecutorType execType,TransactionIsolationLevel level)
    SqlSession openSession(ExecutorType execType)
    SqlSession openSession(ExecutorType execType, boolean autoCommit)
    SqlSession openSession(ExecutorType execType, Connection connection)
    Configuration getConfiguration();

// 得到Dao，是接口的代理对象，mybatis自动创建的
StudentDao studentDao = opensession.getMapper(StudentDao.class);
Student s = studentDao.getStuById(1);
```

### 动态SQL

- `if`

  ```xml
  <select id="findActiveBlogLike"
       resultType="Blog">
    SELECT * FROM BLOG WHERE state = ‘ACTIVE’
    <if test="title != null">
      AND title like #{title}
    </if>
  </select>
  ```

- `choose` (when, otherwise) 相当于在sql中使用switch

  ```xml
  <select id="findActiveBlogLike"
       resultType="Blog">
    SELECT * FROM BLOG WHERE state = ‘ACTIVE’
    <choose>
      <when test="title != null">
        AND title like #{title}
      </when>
      <when test="author != null and author.name != null">
        AND author_name like #{author.name}
      </when>
      <otherwise>
        AND featured = 1
      </otherwise>
    </choose>
  </select>
  ```

- `trim` (where, set)

  ```xml
  <trim prefix="WHERE" prefixOverrides="AND |OR ">
      <!-- 等价于<where> 在select中使用 有必要的时候加上where，去掉多余的 and -->
  </trim>
  <trim prefix="SET" suffixOverrides=",">
    <!-- 等价于<set> 在update中使用 有必要的时候加上set，去掉多余的， -->
  </trim>
  ```

- `foreach`

  ```
  <select id="selectPostIn" resultType="domain.blog.Post">
    SELECT * FROM POST P
    WHERE ID in
    <foreach item="itemName" index="index" collection="listName"
        open="(" separator="," close=")">
          #{item}
    </foreach>
  </select>
  ```

- OGNL（object graph navigation language）对象图导航语言



### 缓存机制

1. 一级缓存：本地缓存；线程级别；只有本sqlsession可用；

   一级缓存会失效的4种情况：

   + 不是同一个sqlsession
   + 同样的方法，不同的参数
   + 使用了增删改方法
   + 手动清空了缓存

2. 二级缓存：全局范围；

默认情况下，只启用了本地的会话缓存；要启用全局的二级缓存：

```xml
<cache/>
```

- 映射语句文件中的所有 select 语句的结果将会被缓存。
- 映射语句文件中的所有 insert、update 和 delete 语句会刷新缓存。
- 缓存会使用最近最少使用算法（LRU, Least Recently Used）算法来清除不需要的缓存。
- 缓存不会定时进行刷新（也就是说，没有刷新间隔）。
- 缓存会保存列表或对象（无论查询方法返回哪种）的 1024 个引用。
- 缓存会被视为读/写缓存，这意味着获取到的对象并不是共享的，可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。





## SSM整合

### 依赖jar包

```javascript
// spring
// 		ioc 核心
spring-beans;
spring-context;
spring-core;
spring-expression;
spring-aop;
commons-logging;
// 		aop 核心
spring-aspects;
com.springsource.org.aopalliance;
com.springsource.org.aspectj.weaver;
com.springsource.net.sf.cglib;
// 		jdbc 核心
spring-jdbc;
spring-orm;
spring-tx;
// 		test
spring-test;

// springMVC
// 		一般
spring-webmvc;
spring-web;
// 		文件下载，jsp-jstl，jsr303
jstl;
commons-fileupload;
commons-io;
standard;
// 		ajax
jackson-core;
jackson-annotation;
jackson-databind;

// mybatis
mybatis;
mybatis-spring;

// 数据源，驱动
mysql-connector;
c3p0;
druid;
```















