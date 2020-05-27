

# JAVA进阶

## 数据库

### SQL

structure query language 结构化查询语言；定义了一种操作所有关系型数据库的规则。

**方言**：每一种数据库的操作方式存在不一样的地方。

不区分大小写，分号结尾，注释-- xxx

#### SQL语句分类

+ DDL（data definition language）:数据库定义语言 CREATE DROP ALTER
+ DML（data manipulation language）：增删改 INSERT DELETE UPDATE
+ DQL（data query language）：查询 SELECT WHERE
+ DCL（data control language）：控制语言，权限安全访问 GRANT REVOKE

CRUD操作：create retrieve update delete

**DDL操作库：**

1. 查询所有数据库的名称：show databases；查看某个数据库的字符集 ：Show create database xxx；
2. create xxx；create database if not exists xxx; create database xxx character set gbk;（默认字符集UTF-8）
3. Alter database xxx chatacter set utf8;
4. drop database xxx; drop database if exists xxx;
5. 查询正在使用的数据库名称：select database(); 使用数据库： use xxx;

**DDL操作表：**

1. 查询某个数据库中所有表的名称：show tables; 查询表结构：desc tablename；

2. 创建表：create table tablename(

   ​	col1  data-type1, 	#data-type数据类型：int/double(位数,小数位数)/date/datetime/timestamp

   ​	col2 data-type2,

   ......

   ​	coln data-typen

   );

   > data-type数据类型：int/double(位数,小数位数)/date/datetime/timestamp/varchar(20字符数)

   Create table newname like tablename;

3. Drop table xxx;  drop table if exists xxx;

4. 修改表名字：**alter table** oldname **rename to** newname;

   修改表的字符集：**alter table** oldname **character set** newname;

   添加一列：**alter table** tablename **add** columnname datatype;

   修改列名或数据类型：alter table tablename change oldcolname newcolname newdatatype; alter table tablename modify colname newdatatype;

   删除列：altert table tablename drop colname;

**DML&DQL增删改查：**

1. 添加数据：insert into tablename(colname1,colname2, ..., colnameN) values (data1, data2, ... ,dataN)；

   列名和值要一一对应；不定义列名，则默认插入所有列的值；除了数字，其他类型都要用引号；

2. 删除数据：delete from tablename [where xxx]; 不加条件则删除所有数据，不建议使用（效率低），truncate table tablename；（删除整张表，并创建一个一样的空表）；

3. 修改数据：update tablename set colname1 = value1, colname2 = value2 [where 条件]; 如果不加条件，所有记录都会修改。

4. 查询数据：select * from tablename;

   ```mysql
   Select [distinct] 
   	columnname, (col1 + ifnull(col2,0)) [as] newname
   from
   	tablename, tablename2
   where
   	[between ... and ...][in][like '_x%'][is / is not null][and &&][or ||][not |][> < =]
   	# _ 单个任意字符
   	# % 多个任意字符
   Group by
   	分组字段 [分组之后只能有分组字段，或者聚合函数]
   having
   	分组之后的条件
   Order by
   	colnamexxx [desc][asc] , 第二排序条件 [desc]
   limit # 方言
   	开始的索引，得到的条数
   ```

   聚合函数✖️5：count (colname) /max/min/sum/avg；**默认排除null值**

> Where vs having ：where分组前的条件，不满足则不参与分组；having分组后的限定条件，可以跟聚合函数进行判断，而where不可以。

**DCL管理授权**

1. 查询用户：切换到mysql数据库，查询user表；
2. 创建用户：create user 'username'@'hostname' identified by 'password';
3. 删除用户：drop user 'username'@'hostname';
4. 修改用户密码：update user set password = password('newpw') where user = 'username';
5. 查询权限：show grants for 'username'@'hostname';
6. 授予权限：grant 权限列表[all] on dbname.tablename[\*.\*] to  'username'@'hostname';
7. 撤销权限：revoke 权限列表[all] on dbname.tablename[\*.\*] from  'username'@'hostname';

#### 约束

约束是对表中的数据进行限定，保证数据的准确性、完整性和有效性。

+ 主键约束：primary key

  ```mysql
  # 主键：非空且唯一；一张表只能有一个字段为主键，表中记录的唯一标识
  Create table student(
    id int primary key,
  	Name varchar(20)
  );
  # 删除主键
  alter table student drop primary key;
  # 添加主键
  alter table student modify id primary key;
  ###########################################
  # 自动增长
  Create table student(
    id int primary key auto_increment,
  	Name varchar(20)
  );
  # 删除自动增长，但是主键没有被删
  alter table student modify id int;
  alter table student modify id int auto_increment;
  ```

+ 非空约束：not null

  ```mysql
  # 创建表格时 指定非空
  Create table student(
  	Name varchar(20) not null
  );
  # 删除 非空约束
  alter table student modify name varchar(20);
  -- 添加 非空约束
  alter table student modify name varchar(20) not null;
  ```

+ 唯一约束：unique

  ```mysql
  # 创建表格时 指定唯一：但是可以都是null（mysql）
  Create table student(
  	Name varchar(20) not null,
    phone varchar(20) unique
  );
  # 删除 唯一约束
  alter table student drop index phone;
  # 添加 唯一约束
  alter table student modify phone varchar(20) unique;
  ```

+ 外键约束：foreign key

  ```mysql
  # 添加外键
  Create table employee(
    id int primary key auto_increment,
  	Name varchar(20),
    dep_id int,
    constraint emp_dep_fk foreign key dep_id references department(id) #department表现存在
  );
  # 删除
  alter table employee drop foreign key emp_dep_fk;
  # 添加外键
  alter table employee 
  	add constraint emp_dep_fk foreign key dep_id references department(id);
  # 级联更新 on update cascade
  # 主表修改主键，外键也对应修改
  # 级联删除 on delete cascade
  alter table employee add constraint emp_dep_fk foreign key dep_id references department(id) on update cascade;
  ```



#### 多表关系与数据库设计

**多表关系**

+ 一对一关系
+ 一对多关系：多表的外键指向一的主键；
+ 多对多关系：中间表至少包含两个字段，作为外键分别指向两张表的主键；

**设计范式**

+ 第一范式（1NF）：每一列都是不可分割的原子数据项；

  <u>存在问题</u>：存在非常多的冗余数据；数据删除/添加也存在问题；

+ 第二范式（2NF）：在第一范式的基础上，非码属性必须完全依赖于候选码，消除非主属性对主码的部分函数依赖；

  <u>函数依赖</u>：如果通过A属性能唯一确定B属性的值，则称B依赖于A；

  <u>完全函数依赖</u>：通过A属性组（学号、课程名称）确定唯一B属性，B依赖A组中所有属性；

  <u>部分函数依赖</u>：B属性值的确定只需要依赖于A属性组中的部分值；

  <u>传递函数依赖</u>：A确定B，B确定C，则C传递函数依赖于A；

  <u>码</u>：一个属性或属性组，被其他所有属性所完全依赖，则称这个属性为该表的码。码属性组中的所有属性；

+ 第三范式：在第二范式基础上，任何非主属性不依赖于其他非主属性，消除传递依赖；

**多表查询**

```mysql
# 笛卡尔积
select * from employee, department;
# 内连接查询
## 隐式内连接
select * from employee e, department d where e.dept_id = d.id;
## 显式内连接
select * from employee e inner join department d on e.dept_id = d.id;
# 外连接查询
## 左外 left join ... on ... 左表信息全
## 右外 right join ... on ... 

# 子查询
select * from employee where ...[in]

```



#### 事务

如果一个包含多个步骤的业务操作，被事务管理，那么这些操作要么同时成功，要么同时失败。

+ start transaction
+ rollback
+ commit

mysql执行DML增删改，会自动commit；开启事务后需要手动commit。

```mysql
# 查看事务的默认提交方式 --1 自动提交 --0 手动提交
select @@autocommit;
set @@autocommit = 0;
```

**事务的四大特征：**

1. 原子性：不可分割的最小操作单位，同时成功/失败；
2. 持久性：提交或回滚之后，数据库会持久化保存数据；
3. 隔离性：多个事务之间，相互独立；
4. 一致性：事务前后，数据总量不变；

**事务的隔离级别**：多个事务之间是隔离的，但是如果有多个事务操作同一批数据，则会引发一些问题，需要设置不同的隔离级别来解决。

+ 脏读：一个事务读到了另一个事务没有提交的数据；

+ 不可重复读（虚读）：同一个事务中，两次读到的数据不一样；

+ 幻读：一个事务操作（DML）数据表中所有记录，另一个事务添加了一条数据，则第一个事务查询不到自己的修改；

  + Read uncommitted
  + read committed
  + Repeatable read
  + serializable

  安全性越来越高，效率越来越低

  ```mysql
  select @@tx_isolation;
  set global transaction isolation level serializable;
  ```



### JDBC

java database connection 使用java语言连接操作数据库：sun公司定义了操作所有关系型数据库的规则，即一套接口；每个数据库厂商写了不同的实现类，提供数据库驱动jar包，能够通过jdbc接口操作不同数据库。

```java
// 导入驱动jar包
// 复制xxx.jar到lib ，add as library

// 注册驱动
Class.forName("com.mysql.jdbc.Driver");
// 获取数据库连接对象
Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/db","root","pw");
// 定义sql
String sql = "select * from table";
// 获取执行sql语句的对象
Statement stmt = conn.createStatement();
// 执行sql语句
stmt.execute(sql);
//释放资源
stmt.close();
conn.close();
```

#### 对象说明

**DriverManager：驱动管理对象**

+ 注册驱动：Class.forName("com.mysql.jdbc.Driver"); 这个包中有一个静态代码块，自动注册驱动，创建了DriverManager
+ 获取数据库连接：DriverManager.getConnection("jdbc:mysql://localhost:3306/db","root","pw"); 如果url是mysql并且默认端口3306，则简写 “jdbc:mysql:///db”

**Connection：数据库连接对象**

+ 获取statement
+ 管理事务：开启 setAutoCommit( false )、提交 commit() 、回滚rollback()

**Statement：执行sql 的对象 静态sql**

+ boolean execute( sql ) ：任意sql
+ 影响的行数 int executeUpdate( sql ) : 表DDL和数据DML增删改， 
+ ResultSet executeQuery( sql )：DQL查询
+ sql注入问题：在拼接sql的时候，有一些sql的特殊关键字参与字符串的拼接；

**PreparedStatement：**执行sql的对象 预编译动态sql

+ 带参数的查询，使用？作为占位符
+ preparedStatement  ps = conn.preparedStatement( 带？的sql )
+ ps.setxxx( 参数? 的位置，参数值)；？**从1开始计数**

**ResultSet：结果集对象**

+ Next() 下移一行
+ getXxx( 参数 ) 获取数据 getInt() getString() .......参数可以是int列编号（从1开始），或者string列名称



#### 数据库连接池

数据库连接资源的集合容器。

**标准接口：DataSource**，在javax.sql包下。数据库厂商完成实现，**c3p0和druid**。

**c3p0**

配置文件c3p0.properties或者c3p0-config.xml，需要放在类路径src下；

创建核心对象，数据库连接池对象：comboPooledDataSource;

```java
DataSource ds = new comboPooledDataSource();
Connection conn = ds.getConnection();
```

获取连接，getConnection;

**Druid**

配置文件是properties文件，任意名称放在任意目录下；

通过工厂来获取数据库连接池对象：DruidDataSourceFactory；

获取连接，getConnection;

```java
Properties pro = new Properties(); //需要InputStream参数
InputStream is = DruidDemo.class.getClassLoader().getResourceAsStream("druid.properties");
pro.load(is);
DataSource ds = DruidDataSourceFactory.createDataSource(pro); //根据properties对象创建连接池
Connection conn = ds.getConnection();
```



**Spring JDBC**

Spring对JDBC进行简单封装，提供了一个JDBCTemplate对象简化JDBC的操作。

JdbcTemplate  temp = new JdbcTemplate( dataSource ) // 根据数据源创建JdbcTemplate对象

+ Update() 执行DML操作
+ QueryForMap() 
+ QueryForList()
+ Query()：查询结果封装为javaBean
+ queryForObject()



### 数据库面试题

1.数据库隔离级别有哪些，各自的含义是什么，MYsql默认的隔离级别是是什么，各个存储引擎优缺点
2.高并发下，如何做到安全的修改同一行数据，乐观锁和悲观锁是什么，INNODB的行级锁有哪2种，解释其含义
3.SQL优化的一般步骤是什么，怎么看执行计划，如何理解其中各个字段的含义，索引的原理？
4.数据库会死锁吗，举一个死锁的例子，mysql怎么解决死锁
5.MYsql的索引实现方式
6.聚集索引和非聚集索引的区别
7.数据库中 BTREE和B+tree区别

## JAVA Web

### XML

Extensible markup language 可扩展标记语言，W3C万维网联盟

<u>**可扩展**</u>：标签都是自定义的  <user></user>

1. 文件后缀 .xml
2. 第一行必须是文档声明：<?xml version='1.0' ?>
3. 有且只有一个根标签
4. 属性值必须使用引号（单双都可以）
5. 标签必须闭合
6. 标签区分大小写

**组层部分**

+ 文档声明：<?xml version='1.0' encoding='gbk' ?>

  版本version（必须）编码方式encoding（默认值ISO-8859-1）是否独立standalone（yes/no）

+ CDATA区域，希望区域内的内容原样展示 <![CDATA[neirong]]>

**约束**

约定xml文档的书写规则

1. dtd约束

   内部dtd：将约束规则放在xml文件中；

   外部dtd：将约束规则定义在外部dtd文件中

+ 本地：<!DOCTYPE 根标签名 SYSTEM "dtd文件的位置">

+ 网络：<!DOCTYPE 根标签名 PUBLIC "dtd文件的名字" "dtd文件的位置url">

  *但是dtd文档不能限制标签内的内容*

2. schema约束

   xsd文件

   <img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200226094013979.png" alt="image-20200226094013979" style="zoom:50%;" />

**解析XML**

解析的两种思想：

1. DOM：将标记语言文档一次性加载进内存，在内存中形成一棵dom树；操作方便，能对文档进行crud所有操作；内存消耗大。
2. SAX：逐行读取，属于事件驱动；内存消耗小，但是只能读取，不能修改。

常见解析器：

1. JAXP：垃圾
2. DOM4J：可以
3. Jsoup：html解析器
4. PULL：安卓内置解析器，SAX思想

**Jsoup**



### Tomcat

**部署项目**

<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200226150754846.png" alt="image-20200226150754846" style="zoom:50%;" />



**目录结构**

<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200226151032267.png" alt="image-20200226151032267" style="zoom:50%;" />

### servlet

server applet：运行在服务器端的小程序；一个接口，定义了java类被tomcat识别的规则

```xml
<!-- web.xml -->
<servlet>
	<servlet-name>demo</servlet-name>
  <servlet-class>com.zjubiomedit.demo.demoservlet</servlet-class>
  <!--指定创建时间 
			负数例如-1，第一次被访问时创建（默认）；
			非负数，服务器启动时被创建
	-->
  <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
  <servlet-name>demo</servlet-name>
  <url-pattern>/demo1</url-pattern>
</servlet-mapping>
```

servlet生命周期：

1. 被创建时执行init（），只执行一次；
2. 提供服务service（）；
3. 被销毁前destroy（）释放资源，只有服务器正常关闭才会执行；

servlet是单例的，多个用户访问时可能会出现线程安全问题：尽量不要在servlet中定义成员变量

可以进行注解配置servlet，@webServlet( urlPattern = {"/demo"}) 或@webServlet( "/demo" )

<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200227152219156.png" alt="image-20200227152219156" style="zoom:50%;" />

**servletContext对象**：

代表整个web应用，可以和应用的容器通信；

功能：

+ 获取MIME类型：

  MIME类型：在互联网通信过程中定义的文件数据类型 text/html；

  string getMimeType(string file);

+ 域对象：共享数据

+ 获取文件的真实路径

**request功能**

```java
// 获取请求方法
String getMethod();
// 获取虚拟目录
String getContextPath();
// 获取servlet路径
String getServletPath();
// 获取get方式请求参数
String getQueryString();
// 获取请求URI
String getRequestURI(); // /xunimulu/fangfaming
StringBuffer getRequestURL(); // http://localhost/xunimulu/fangfa
// 获取协议及版本
String getProtocol();
// 获取客户机的IP地址
String getRemoteAddr();
// 获取请求头
String getHeader(String name);
Enumeration<String> getHeaderNames();
// 获取请求体
BufferedReader getReader(); // 获取字符输入流
ServletInputStream getInputStream(); // 获取字节输入流

// 其他功能 获取通用请求参数
String getParameter(String name);
String[] getParameterValues(String name);
Enumeration<String> getParameterNames();
Map<String, String[]> getPatameterMap();
// 请求转发 通过request对象获得请求转发器对象requestDispatcher, 地址栏并不发生变化，浏览器只发送一次请求
RequestDispatcher getRequestDispatcher(String path);
forward(ServletRequest request, ServletResponse response);
//
ServletContext getServletContext();
```

**response功能**

<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200228153016929.png" alt="image-20200228153016929" style="zoom:50%;" />

重定向与转发的区别：

<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200228155654294.png" alt="image-20200228155654294" style="zoom:50%;" />

<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200228161801085.png" alt="image-20200228161801085" style="zoom:50%;" />

```java
setStatus(int statuscode);
setHeader(String name, String value);
sendRedirect(String path);
// 获取输出流，字符/字节
PrintWriter getWriter();
ServletOutputStream getOutputStream();
```

**request域**：一次请求的范围，用于转发数据共享

+ setAttribute(String name ,Object obj);
+ Object getAttribute(String name);
+ Void removeAttribute(String name);

**session域**：

服务器端会话技术：HttpSession。

<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200229131153648.png" alt="image-20200229131153648" style="zoom:50%;" />



**cookie域**：

cookie特点和作用：存储在客户端浏览器，对单个cookie有大小限制（4kb），同一个域名下的cookie数量也有限制（20）；一般用于存储少量不敏感的数据，在不登陆的情况下，完成服务器对客户端的识别。

<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200229124031524.png" alt="image-20200229124031524" style="zoom:50%;" />

<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200229124132133.png" alt="image-20200229124132133" style="zoom:50%;" />

<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200229124221729.png" alt="image-20200229124221729" style="zoom:40%;" />



**JSP**

**jsp本质上是一个servlet**，继承了httpservlet类

```jsp
<!--jsp脚本-->
<% %> 		<!-- 定义的脚本在service方法中 -->
<%!  %> 	<!-- 定义的是成员变量或者成员方法(使用少，线程不安全) -->
<%=  %>		<!-- 输出到页面上 -->
<%@ page contentType="text/html;charset=UTF-8" %>
<%@ include file="head.jsp" %>  <!-- 导入页面资源文件 -->
<%@ taglib prefix="x" url="http://java.sun.com/jsp/jstl/core" %>  <!-- 导入资源 -->
```

<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200302195149829.png" alt="image-20200302195149829" style="zoom:70%;" />

**JSTL???**

> **javaBean：标准的java类，类是public，域private，必须有空参构造器，提供公共setter和getter**

**动态代理**

<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200303101714832.png" alt="image-20200303101714832" style="zoom:50%;" />



**Listener监听器**

<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200303103853030.png" alt="image-20200303103853030" style="zoom:50%;" />



###Redis

nosql非关系型数据库，数据之间没有关联关系，数据存储在内存当中。

<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200303105822587.png" alt="image-20200303105822587" style="zoom:70%;" />



### MVC

### Maven









## 面试题

### 基础与框架

1.String类能被继承吗，为什么
2.String，Stringbuffer，StringBuilder的区别？
3.ArrayList和LinkedList有什么区别
4.类的实例化顺序，比如父类静态数据，构造函数，字段，子类静态数据，构造函数，字段，他们的执行顺序
5.用过哪些Map，都有什么区别，HashMap是线程安全的吗,并发下使用的Map是什么，他们内部原理分别是什么，比如hashcode，扩容等
6.HashMap为什么get和set那么快，concurrentHashMap为什么能提高并发
7.抽象类和接口的区别，类可以继承多个类么，接口可以继承多个接口么,类可以实现多个接口么
8.什么情况下会发生栈内存溢出
9.什么是nio，原理
10.反射中，Class.forName和ClassLoader区别
11.tomcat结构，类加载器流程
12.讲讲Spring事务的传播属性,AOP原理，动态代理与cglib实现的区别，AOP有哪几种实现方式
13.Spring的beanFactory和factoryBean的区别
14.Spring加载流程
15.Spring如何管理事务的

### 多线程

1.线城池的最大线程数目根据什么确定
2.多线程的几种实现方式，什么是线程安全，什么是重排序
3.volatile的原理，作用，能代替锁么
4.sleep和wait的区别，以及wait的实现原理
5.Lock与synchronized 的区别，synchronized 的原理，什么是自旋锁，偏向锁，轻量级锁，什么叫可重入锁，什么叫公平锁和非公平锁
6.用过哪些原子类，他们的参数以及原理是什么
7.用过哪些线程池，他们的原理简单概括下，构造函数的各个参数的含义，比如coreSize，maxsize等
8.有一个第三方接口，有很多个线程去调用获取数据，现在规定每秒钟最多有10个线程同时调用它，如何做到。
9.spring的controller是单例还是多例，怎么保证并发的安全
10.用三个线程按顺序循环打印abc三个字母，比如abcabcabc
11.ThreadLocal用过么，原理是什么，用的时候要注意什么
12.如果让你实现一个并发安全的链表，你会怎么做





### 架构设计与分布式

1.tomcat如何调优，各种参数的意义
2.常见的缓存策略有哪些，你们项目中用到了什么缓存系统，如何设计的，Redis的使用要注意什么，持久化方式，内存设置，集群，淘汰策略等
3.如何防止缓存雪崩
4.用java自己实现一个LRU
5.分布式集群下如何做到唯一序列号
6.设计一个秒杀系统，30分钟没付款就自动关闭交易
7.如何做一个分布式锁
8.用过哪些MQ，怎么用的，和其他mq比较有什么优缺点，MQ的连接是线程安全的吗
9.MQ系统的数据如何保证不丢失
10.分布式事务的原理，如何使用分布式事务
11.什么是一致性hash
12.什么是restful，讲讲你理解的restful
13.如何设计建立和保持100w的长连接？
14.解释什么是MESI协议(缓存一致性)
15.说说你知道的几种HASH算法，简单的也可以
16.什么是paxos算法
17.redis和memcached 的内存管理的区别
18.一个在线文档系统，文档可以被编辑，如何防止多人同时对同一份文档进行编辑更新