##    Java SE

### 基础部分

#### 基本知识

**JRE、JDK、JVM之间的关系**

JDK是开发工具包，通常包含JRE和编译debug等开发时需要的工具；JRE是java的运行环境，通常包含JVM和一些运行时需要的核心类库；JVM是java虚拟机，是实现java跨平台的关键。

**常用的JDK包**

Java.lang ：包装类，线程

Java.match：用BigDecimal精确数字类型

java.util ：并发，集合相关

**基本数据类型**

一共有8种基础类型：4个整型byte、short、int、long；2个浮点类型float、double；1个字符类型char；一个布尔类型boolean。需要注意的是，java没有无符号数！

**java文件 & class文件**

单个java文件中可以有一个public类和多个非public类，在编译的时候会变成多个class文件，解释器运行包含main方法的类以启动程序。

**面向对象 VS 面向过程**

面向过程的性能更高一些，因为面向对象需要实例化，开销比较大。在嵌入式开发等情况下，一般是采用面向过程开发。但是面向对象开发更易维护，易复用、易扩展，具有封装、继承、多态的特性，可以设计出低耦合的系统。

**String VS StringBuilder VS StringBuffer**

String是不可变对象，每次修改都要重新创建对象。stringBuffer 是线程安全的，效率较低，但允许多线程调用；stringBuilder 效率较高，通常使用stringBuilder。

<u>字符串的比较：用equals()</u>

**负数的取余、除法**

java中/ 和 %的结果，符号都和<u>被除数</u>保持一致。

**call by value**

Java总是按值调用，call by value。方法不能修改传递给它的任何参数变量的内容。但是参数是一个对象引用的时候，方法可以改变所引用对象的内容。因为对象引用和对象引用的拷贝都指向同一个对象。

即：不能修改基本数据类型的参数，不能让对象参数引用一个新的对象，但是可以改变一个对象参数的状态。

**局部变量问题**

java不能在嵌套块中重复声明变量。

**& 和 &&**

&：逻辑与（and）。 运算符两边的表达式均为true时，整个结果才为true。

&&：短路与 。如果第一个表达式为false时，第二个表达式就不会计算了。

**== 和 equals**

== ： 基本数据类型：比较值；引用类型：比较地址；

equals：不能比较基本数据类型； 如果重写了equals方法的，就是调用这个方法进行比较；如果没有重写，就使用的object中的equals，即比较地址，这时候就和==是一样的。

**hashCode 和 equals**

hashCode的作用是获取哈希码（散列码），哈希码决定了这个对象在哈希表中的索引位置。所以HashSet在检查重复的时候，先算hashCode是否相同，再用equals去检查是否真的是同一个对象。

两个对象相等，意味着他们的hashcode相等，调用equals方法也相等，但是如果不同时重写hashcode和equals方法，两个对象是不可能相等的（地址绝不相等）

**int 和 Integer**

- Integer是int的包装类；int是基本数据类型；
- Integer变量必须实例化后才能使用；int变量不需要；
- Integer实际是对象的引用，指向此new的Integer对象；int是直接存储数据值 ；
- Integer的默认值是null；int的默认值是0。
- 通过new创建的Integer永远都不相等！
- Integer变量和int变量比较时，只要两个变量的值是向等的，则结果为true！
- 非new生成的Integer变量和new Integer()生成的变量比较时，结果为false。因为非new生成的Integer变量指向的是静态常量池中cache数组中存储的指向了堆中的Integer对象，而new Integer()生成的变量指向堆中新建的对象，两者在内存中的对象引用（地址）不同。
- 对于两个非new生成的Integer对象，进行比较时，如果两个变量的值在区间-128到127之间，则比较结果为true，如果两个变量的值不在此区间，则比较结果为false。而java API中对Integer类型的valueOf的定义如下，对于-128到127之间的数，会进行缓存。

**序列化和反序列化？**

序列化：一种用来处理对象流的机制，将对象的内容转化为二进制流，可以将对象持久化或者网络传输。

反序列化：将二进制流还原为对象的过程

如何实现序列化：通过实现Serializable接口



**Comparable (内) & Comparator (外)**

Arrays.sort()可以对对象数组进行排序，前提是**对象实现了Comparable接口**（重写compareTo方法）。整型可以直接相减，浮点数需要用Double.compare(x,y)。

在不能修改对象的comparable的情况下，可以另外**实现Comparator接口中的compare方法**，comparator作为sort()方法的第二个参数。这里可以用lambda表达式来简化写法。



**Error 和 Exception**

有共同的父类Throwable

Error 表示系统级的错误 和 程序不必处理的异常，是java运行环境中的内部错误或者硬件问题。比如：内存资源不足等。对于这种错误，程序基本无能为力，除了退出运行外别无选择，它是由`Java虚拟机抛出`的。

Exception和RuntimeException区别：Exception可能是checked exception，强制要求被处理，同时也可以处理修复这个问题；而runtimeException可以不被处理，是unchecked exception。非受查异常不处理的时候，由虚拟机接管，向外层抛出，要么线程中止，要么主程序终止。

自定义exception的时候，继承哪一个都要根据实际情况来判断，如果是必须处理的异常，那么就继承exception。



**Object类中常见的方法？为什么wait和notify在这里？**

Clone() / equals() / hashcode() / finalize() / wait() / notify() 

因为Java提供的锁是对象级别的，每个对象都有对象头，来存放锁



**新建对象的方式**

+ new关键字
+ 反射机制 newInstance()
+ clone方法
+ 反序列化，IO的时候



**正则表达式**

java有四个内置的正则表达式方法，matches()、split()、replaceFirst()、replaceAll()

还可以使用Pattern类和Matcher类进行匹配：

+ 首先，通过正则表达式创建Pattern对象;

+ 通过模式对象 `Pattern`，根据指定字符串创建匹配对象 `Matcher`

+ 通过匹配对象 `Matcher`，操作字符串。

  ```java
  Pattern pattern = Pattern.compile("\s+");
  Matcher matcher = pattern.matcher(text);
  matcher.find();
  matcher.start();
  matcher.end();
  matcher.replaceAll("\t");
  ```



**一个十进制的数在内存中是怎么保存的？**

二进制补码形式，最高位不变，原码取反+1

****

#### 关键字

**static**

静态的域或者静态的方法，可以通过类来调用，而不需要对象。在类加载的时候就存在了（方法区中），先于对象（堆）。

+ 静态方法只能访问静态变量，但是非静态方法可以访问静态或者非静态；
+ 静态方法中不能用this，super，因为它不知道this，super是谁；
+ 主函数是static的。



**final**

不可以被继承、修改的。



**public > protected > 缺省 > private**

<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200606155244419.png" alt="image-20200606155244419" style="zoom:100%;" />



**swtich条件**

可以是byte、short、int、char、string、枚举，但是不能是long



**short **

```java
short s1 = 1;
s1 = s1 + 1; // 编译错误，需要强制转换（short）
s1 += 1; // 可以编译，隐含强转的意思了
```



****

#### 面向对象

**类的构造方法**

类的构造器与类同名，可以有多个构造器，多个参数，没有返回值，总是伴随new一起调用；

没有定义构造器的时候，会提供一个默认无参数的构造器，但是一旦提供了构造器，那么无参数构造器就要自己写，不会默认提供。

**对象构造步骤**

+ 所有数据被赋予默认值；
+ 按照类中声明的顺序，依次执行所有域初始化语句和初始化块；
+ 构造器第一行如果调用了第二个构造器，先执行第二个；
+ 执行本构造器。

**隐式参数 VS 显示参数**

隐式参数：方法的调用/接受者，用关键字this来表示；显式参数：括号里的参数。

**域 & 对象**

+ 一个方法可以访问所属类的所有对象的私有数据。

+ final域是在构造对象的时候就初始化了。

+ 静态域static属于类，静态方法可以调用静态域，但是不能访问实例域。静态方法可以用来作为工厂方法，构造函数无法改名，而希望不同的构造函数取不同的名字的情况，或者需要改变返回值类型的情况下使用。

**继承和多态**

java是单继承，一个类只允许有一个父类，但是可以实现多个接口。多态指的是，在父类中定义的属性和方法被子类继承之后，表现出不同的数据类型或者不同的行为，使得同一个方法在父类和子类中可以有不同的含义。

java中实现多态有几个条件：

+ 首先，多态中必须存在具有继承关系的子类和父类；
+ 其次，子类对父类中的某些方法进行重写；
+ 最后，将子类的引用赋值给父类对象

这样才能使用统一的逻辑实现代码处理不同的对象，从而达到不同的执行行为。

子类**不能直接访问**超类的私有域，但是可以使用 **super.getxxx()**  来调用子类构造器，可以显式的使用super构造器来对超类的私有域进行赋值；否则自动调用超类默认构造器（无参数的）只能在继承层次内进行类型转换；在将超类转换成子类之前，应该使用instanceof进行检查。

**抽象类和接口的区别**

抽象类是他的所有子类的公共属性集合，是包含一个或多个抽象方法的类。只有抽象的类，里面才会有抽象的方法，也可能还有别的具体域或者具体方法。抽象方法，是一个占位墩子，他的具体实现在子类中。不含抽象方法，也可以是抽象类。抽象类不能创建实例，但是可以引用非抽象子类。

而接口是一系列方法特征的集合。

+ 抽象类可以有构造方法，但是接口不行；
+ 抽象类可以有普通成员变量，但是接口不行（接口中的属性都是public static final）；
+ 抽象类可以有非抽象的普通方法，但是接口不行（接口中所有方法默认是public，1.8之后可以提供default、static方法）；
+ 抽象类中的抽象方法可以是public、protected，但是接口只能是public；
+ 一个类可以实现多个接口，但是只能继承一个抽象类。

**重写 & 重载**

重写：override，子类中把继承来的方法重新写了一遍。此时方法名、参数列表、返回类型（子类的返回类型可以是父类分会类型的子类）都一样，子类函数的访问权限不能少于父类。

重载：overloading，此时方法名称一样，但是参数列表不同，不规定返回类型。

重载在编译时就可以确定对应执行的方法，而重写实现的是运行时的多态，需要动态分派/绑定来确定执行哪个方法。

**接口、超类出现冲突**

接口默认方法、超类同名方法、另一接口同名方法，并存的时候，**超类优先**；

接口冲突的时候（只要有一个接口提供默认实现）必须覆盖这个方法来解决冲突。

两个接口都没有提供默认实现的时候，根本就不存在冲突。



**内部类**

局部内部类和匿名内部类访问局部变量的时候，为什么变量必须加上final？

是因为**生命周期不一致**， 局部变量直接存储在栈中，当方法执行结束后，非final的局部变量就被销毁。而局部内部类对局部变量的引用依然存在，如果局部内部类要调用局部变量时，就会出错。加了final，可以确保局部内部类使用的变量与外层的局部变量区分开，解决了这个问题。



**面向对象开发的6个基本原则**

1. 单一原则：一个类就只做它该做的事情；
2. 开闭原则：软件实体对扩展开放，对修改关闭；要做到开闭原则，有两个点
   + 抽象是关键，没有抽象类或者接口的系统很难扩展；
   + 封装可变性：将系统中的各种可变因素封装到一个继承结构中，不要将多个可变因素混杂在一起
3. 里氏替换原则：任何时候都可以用子类来替换超类，因为子类是超类能力的增强
4. 依赖倒置原则：面向接口编程，高层模块不能依赖底层模块
5. 接口隔离原则：一个接口应该只能描述一种能力，高度内聚，不能大而全
6. 迪米特法则：最少知识原则，一个对象应该对其他对象尽可能少的了解

**项目中用到了哪些？**应用了单一原则，和里氏替换原则



**泛型限定符extends / super**

< ? extends Fruit >确定了上界，只能是它的子类

<? super Apple> 确定了下界，只能是apple的超类 



#### 委托Delegation和继承Inheritance

**委托/委派**

一个对象请求另一个对象的功能，捕获一个操作并将其发送到另一个对象；

+ Dependency依赖：临时性的委派

  在这种关系中，一个类使用另一个类，但是不是作为属性，是一种use关系，另一个类可以是一个参数，或者是一个方法中的使用到的。public class CourseSchedule {List<Course> courses = new ArrayList<>();}

  B是A某个方法的参数，并且A的方法中调用了B的方法

+ Association关联：永久性的委派

  允许一个对象实例让另一个对象实例代表它自己做其他事情，属于has关系，并不在A的方法中调用B的方法，只是单纯将A，B通过属性连接起来。class Teacher { private Student [] students;  }

+ Composition：

  在A类中实例化B类对象，并在方法中调用B的方法



****

#### IO/NIO/AIO

BIO：Block IO 同步阻塞式 IO，就是我们平常使用的传统 IO，它的特点是模式简单使用方便，并发处理能力低。
NIO：Non IO 同步非阻塞 IO，是传统 IO 的升级，客户端和服务器端通过 Channel（通道）通讯，实现了多路复用。
AIO：Asynchronous IO 是 NIO 的升级，也叫 NIO2，实现了异步非堵塞 IO ，异步 IO 的操作基于事件和回调机制。



#### 自定义注解？？？







****

### 集合

#### 两套继承体系：Collection & Map

List 和 Set接口继承自Collection接口；Map接口是单独出来的。

#### HashMap如何解决哈希冲突

1. 使用链地址法（使用散列表）来链接拥有相同hash值的数据；
2. 使用2次扰动函数（hash函数）来降低哈希冲突的概率，使得数据分布更平均；
3. 引入红黑树进一步降低遍历的时间复杂度，使得遍历更快；

#### 相似类型的性能/特性对比

**Hashtable 和 HashMap**

Hashtable：是线程安全的，Hashtable中的方法都是synchronized修饰的，在多线程并发的环境下，可以直接使用Hashtable，不需要自己为它的方法实现同步。继承自Dictionary类。既不支持Null key也不支持Null value；

HashMap：是非线程安全的。需要自己手动增加同步处理。继承自AbstractMap类。支持null作为键，这样的键只有一个，同时，它还支持可以有一个或多个键所对应的值为null。

**ArrayList 和 LinkedList**

+ ArrayList是动态数组，LinkedList是双向链表的数据结构；
+ ArrayList的随机访问效率更高
+ LinkedList删除、插入效率更高
+ LinkedList内存空间占用更大
+ 两者都不是线程安全的

ArrayList的优点：

+ 底层是数组，是一种随机访问模式，查找快；
+ 顺序添加非常快

ArrayList缺点：

+ 删除、插入操作耗费性能

**ArrayList 和 Vector**

Vector所有方法都是同步的，而ArrayList不是。他们都继承了List接口。

**ArrayList的扩容机制**

ArrayList中有一个capacity容量，如果插入新值超过了capacity会触发扩容；

默认情况下，新的容量会是原容量的1.5倍；但如果还是小于期望容量，那就返回期望容量。

使用1.5倍这个数值而不是直接使用期望容量，是为了防止频繁扩容影响性能；

1.8之前的初始化容量是10，1.8之后默认是0，第一次使用add的时候扩容为10，防止未使用的arraylist占用空间。

```javascript
MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8
```

**HashMap 1.7和1.8 的区别**

- 1.7，数组+链表；

- 1.8，数据结构有数组+链表和红黑树，使用红黑树是为了能够提高查询效率。在链表长度达到8，并且数组长度大于等于64时，将链表转换成红黑树，如果数组长度小于64，只是对数组进行扩容。
- Hashmap用红黑树而不是平衡树：红黑树也是一种平衡树，只不过不是严格平衡的，它的效率会比严格平衡树要高。

**HashMap扩容机制**

1. HashMap 在 new 后并不会立即分配bucket数组，而是第一次 put 时初始化，类似 ArrayList 在第一次 add 时分配空间。
2. HashMap 的 bucket 数组大小一定是2的幂，如果 new 的时候指定了容量且不是2的幂，实际容量会是最接近(大于)指定容量的2的幂，比如 new HashMap<>(19)，比19大且最接近的2的幂是32，实际容量就是32。
3. HashMap 在 put 的元素数量大于 Capacity * LoadFactor（默认16 * 0.75） 之后会进行扩容，2倍扩容
4. 1.8尾插法，按照扩容后的规律计算，1.7头插法，真正初始化哈希表（初始化存储数组`table`）是在第1次添加键值对时，即第1次调用`put（）`时

**HashMap在并发下会产生什么问题？**

HashMap并发下产生问题：在发生hash冲突，插入链表的时候，多线程会造成环链，再get的时候变成死循环，Map.size()不准确，数据丢失。

**有什么替代方案**?(HashTable, ConcurrentHashMap)。它们两者的实现原理

- HashTable: 通过synchronized来修饰，效率低，多线程put的时候，只能有一个线程成功，其他线程都处于阻塞状态

- ConcurrentHashMap：
  1.7 采用锁分段技术提高并发访问率
  1.8 数据依旧是分段存储，但锁采用了synchronized，内部采用Node数组+链表+红黑树的结构存储，当单个链表存储数量达到红黑树阈值8时（此时链表已有元素7），并且数组长度大于64时，存储结构转换为红黑树来存储，否则只进行数组的扩容



**LinkedHashMap实现LRU**

LinkedHashMap其实就是HashMap和LinkedList结合起来，即有序的HashMap

构造函数第三个参数为true的时候，**在此基础上再根据访问顺序排序，即最近访问的在后**

```java
Map<String, String> map = new LinkedHashMap<String, String>(16,0.75f,true);
Iterator iter = map.entrySet().iterator();
while (iter.hasNext()) {
    Map.Entry entry = (Map.Entry) iter.next();
    System.out.println(entry.getKey() + "=" + entry.getValue());
}

public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private int initialCapacity;
    public LRUCache(int initialCapacity) {
        //true表示按照访问顺序排序
        super(initialCapacity, 0.75f, true);
        this.initialCapacity = initialCapacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        if (size() > initialCapacity) {
            return true;
        }
        return false;
    }
}
```



****

#### Collection和Collections

java.util.Collection 是一个集合接口（集合类的一个顶级接口）。它提供了对集合对象进行基本操作的通用接口方法。Collection接口在Java 类库中有很多具体的实现。Collection接口的意义是为各种具体的集合提供了最大化的统一操作方式，其直接继承接口有List与Set。
Collections则是集合类的一个工具类/帮助类，其中提供了一系列静态方法，用于对集合中元素进行排序、搜索以及线程安全等各种操作。

```java
List<String> synchronizedList = Collections.synchronizedList(list);
```



#### 快速失败 & 安全失败

fail-fast：当多个线程对集合进行结构上的改变操作的时候，集合在被遍历期间会检查modCount值，如果这个值发生改变，可能会出现fail-fast，抛出ConcurrentModificationException异常

#### 同步集合

#### 迭代器

```java
List<String> list = new ArrayList<>();
Iterator<String> it = list. iterator();
while(it. hasNext()){
  String obj = it. next();
  System. out. println(obj);
}
```

迭代器的特点就是只能单向遍历，但是更安全，它可以保证，如果出现多线程异常，会抛出异常。

遍历List的三种办法：

+ for循环，适合ArrayList
+ 迭代器遍历
+ foreach循环：内部其实也用的iterator





****

### 线程

#### 线程6个状态：

+ New：新创建，未执行；
+ Runnable：运行中，正在执行run()方法；
+ Blocked：运行中，因为某些操作被阻塞而挂起；
+ Waiting：运行中，因为某些操作在等待中，执行wait()之后；
+ Time Waiting：运行中，执行sleep()方法正在计时；
+ Terminated：终止，run()方法执行完毕。

#### 多线程的实现方式

+ 继承Thread类：Thread类本质上是一个实现了Runnable接口的实例；
+ 实现Runable接口
+ 实现Callable接口，重写call方法：这种实现可以抛出异常，可以有future返回值，了解任务的执行情况；
+ 基于线程池的方式；
+ @EnableAsync 和 @Async注解来定义线程任务。

#### run方法和start方法的区别

直接使用run方法，并不能新建线程，而是继续在主线程中执行；

start方法可以新建线程，异步执行。

#### sleep方法和wait方法的区别

两者都能够用来放弃CPU一段时间；区别在于：

+ sleep不放弃锁，线程进入Time waiting状态，经过特定时间就能醒来
+ wait释放了锁，线程进入waiting状态，需要notify才能唤醒，并且需要重新获取锁









### JVM



#### 反射

**什么是反射机制？**

在运行状态，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性。**动态获取信息和动态调用对象的能力！**

**优缺点**

运行期类型的判断，动态加载类，提高代码的灵活度；

性能瓶颈。

**应用场景**

+ 框架
+ 动态代理模式
+ 动态配置

```java
// 创建对象
Class c1 = new Student().getClass();
// 路径
Class c2 = Class.forName("xxx.Student");
// 类名
Class c3 = Student.class;
```



****

#### GC

##### 内存泄漏 & 内存溢出

内存泄漏：分配出去的内存没有被收回来，失去了对该内存区域的控制，造成资源的浪费；

内存溢出：程序需要的内存超过了系统所能分配的上限；

java一般不会内存泄漏，因为垃圾回收机制会自动回收。但是如果对象一直保存了它的引用，就可能会一直不被回收。





#### 类加载

#### 

