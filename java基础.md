## java语言基础班

1. java虚拟机 java virtual machine JVM：java代码的运行环境，使java程序具有跨平台性。而jvm本身不具备跨平台性，每个操作系统有各自版本的jvm。

2. JRE：java runtime enviroment，java程序的运行时环境，包含JVM和运行时所需要的核心类库。

3. JDK：java development kit，是java程序开发工具包，包含JRE和开发人员使用的工具。

   

5. helloword：源代码.java --> 字节码.class   javac编译器；java解释器

6. ``` java
   public class demo {
   	public static void main(String[] args){
       
     }
   }
   ```

7. ascii 48--0 65--A 97--a

![image-20200204202323694](/Users/chengleiyi/Documents/Lernen/笔记/截屏2020-02-01下午3.18.41.png)

![截屏2020-02-01下午3.26.27](/Users/chengleiyi/Documents/Lernen/笔记/截屏2020-02-01下午3.26.27.png)

![截屏2020-02-01下午3.46.49](/Users/chengleiyi/Documents/Lernen/笔记/截屏2020-02-01下午3.46.49.png)

<img src="/Users/chengleiyi/Documents/Lernen/笔记/截屏2020-02-01下午3.50.56.png" alt="截屏2020-02-01下午3.50.56" style="zoom:50%;" />

![截屏2020-02-02上午11.06.41](/Users/chengleiyi/Documents/Lernen/笔记/截屏2020-02-02上午11.06.41.png)

![截屏2020-02-02下午2.59.03](/Users/chengleiyi/Documents/Lernen/笔记/截屏2020-02-02下午2.59.03.png)

![截屏2020-02-02下午3.02.20](/Users/chengleiyi/Documents/Lernen/笔记/截屏2020-02-02下午3.02.20.png)

![截屏2020-02-02下午4.14.06](/Users/chengleiyi/Documents/Lernen/笔记/截屏2020-02-02下午4.14.06.png)

![截屏2020-02-02下午4.23.52](/Users/chengleiyi/Documents/Lernen/笔记/截屏2020-02-02下午4.23.52.png)

![截屏2020-02-04下午3.49.57](/Users/chengleiyi/Documents/Lernen/笔记/截屏2020-02-04下午3.49.57.png)

### 进程与线程

**进程**：一个内存中运行的应用程序，每个进程都有独立的内存空间；一个应用程序可以运行多个进程。

**线程**：线程是一个进程中的执行单元，负责当前进程中程序的执行；一个进程中至少有一个线程，可以有多个线程。

线程调度：分时调度 or 抢占式调度

主线程：执行main方法

java属于抢占式调度，那个线程抢到了就先执行。

**类 extend Thread 或者 implements Runnable； 实现runnable的优势：还能实现其他接口，继承其他类；增强程序扩展性，将设置线程任务和开启新线程进行了分离。**

Thread类中的方法

```java
// 获取线程名称
String getName();
static Thread currentThread();
Thread.currentThread().getName();
// 设置线程名称
void setName();
// 暂停该线程xx毫秒,可以当定时器
public static void sleep(long millis);
```

**线程同步：**

+ 同步代码块

  ```java
  synchronized(lock){
  	// 需要同步操作的代码
  }
  
  public synchronized void sell(){
    
  }
  ```

  

+ 同步方法

+ 锁机制

### IO流

![javaIO流](/Users/chengleiyi/Documents/Lernen/笔记/javaIO流.png)

其中，以Stream结尾的为字节流，以Writer或者Reader结尾的为字符流。

所有的输入流都是抽象类InputStream（字节输入流）或者抽象类Reader（字符输入流）的子类，所有的输出流都是抽象类OutputStream(字节输出流)或者抽象类Writer(字符输出流)的子类。

字符流能实现的功能字节流都能实现，反之不一定。如：图片，视频等二进制文件，只能使用字节流读写。

1. 4大超类：outputStream/inputStream/Writer/Reader，两个字节流，两个字符流

2. 字节流常用FileOutputStream和FileInputStream，构造方法参数可以是File或者String类型，输出流可以指定是否为append；

3. 字符流常用FileReader和FileWriter，构造参数同上；区别，字符流自带缓冲区，因此可以读字符，包括中文；所以多一个flush方法

4. 缓冲流：BufferedOutputStream/BufferedInputStream/BufferedWriter/BufferedReader，是对4个Filexxx常用流的增强，创建缓冲流时会创建一个默认大小的缓冲区，减少系统IO次数，提高读写效率；

   构造参数：outputStream/inputStream/Writer/Reader

5. 转换流：InputStreamReader/OutputStreamWriter，可以指定charset解码/编码方式；构造参数：OutputStream/InputStream和charsetName编码表名称（不区分大小写）

6. 对象操作流：ObjectInputStream和ObjectOutputStream；构造参数：OutputStream/InputStream。保持的对象需要实现serializable接口（标记型接口）可以进行序列化。writeObject/readObject。

7. 打印流：PrintStream

### 函数式编程

+ 函数式接口作为参数
+ 函数式接口作为返回值



### Collection Framework

+ Set：Set代表无序、不可重复的集合；如果试图把两个相同的元素加入同一个Set集合中，则添加操作失败，add()方法返回false，且新元素不会被加入。
+ List：
+ Queue：队列集合；新元素插入（offer）到队列的尾部，访问元素（poll）操作会返回队列头部的元素。通常，队列不允许随机访问队列中的元素。
+ Map：Map代表具有映射关系的集合

Java的集合类主要由两个接口派生而出：**Collection和Map**。继承关系如下：



<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200412104508523.png" alt="image-20200412104508523" style="zoom:40%;" />



<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200412104721365.png" alt="image-20200412104721365" style="zoom:80%;" />

`add()` ` addAll()` 添加

`clear()` 清空 `remove()` `removeAll()` 

```java
public class IteratorExample {
    public static void main(String[] args){
        //创建集合，添加元素  
        Collection<Day> days = new ArrayList<Day>();
        for(int i =0;i<10;i++){
            Day day = new Day(i,i*60,i*3600);
            days.add(day);
        }
        //获取days集合的迭代器
      	//修改迭代变量的值对集合元素本身没有任何影响。
        Iterator<Day> iterator = days.iterator();
        while(iterator.hasNext()){//判断是否有下一个元素
            Day next = iterator.next();//取出该元素
            //逐个遍历，取得元素后进行后续操作
            .....
        }
}}
```



**LinkedList**:

 **void addFirst(E e):**将指定元素插入此列表的开头。
 **void addLast(E e):** 将指定元素添加到此列表的结尾。

**boolean add(E e) **在尾部插入

 **boolean offer(E e)** 在尾部插入

 **boolean offerFirst(E e):** 在此列表的开头插入指定的元素。
 **boolean offerLast(E e):**  在此列表末尾插入指定的元素。 **E getFirst(E e):**  返回此列表的第一个元素。

 **E getLast(E e):** 返回此列表的最后一个元素。

 **E peekFirst(E e):**  获取但不移除此列表的第一个元素；如果此列表为空，则返回 null。
 **E peekLast(E e):**  获取但不移除此列表的最后一个元素；如果此列表为空，则返回 null。
 **E pollFirst(E e):** 获取并移除此列表的第一个元素；如果此列表为空，则返回 null。
 **E pollLast(E e):** 获取并移除此列表的最后一个元素；如果此列表为空，则返回 null。
 **E removeFirst(E e):**  移除并返回此列表的第一个元素。
 **boolean removeFirstOccurrence(Objcet o):** 从此列表中移除第一次出现的指定元素（从头部到尾部遍历列表时）。
 **E removeLast(E e):** 移除并返回此列表的最后一个元素。
 **boolean removeLastOccurrence(Objcet o):**     从此列表中移除最后一次出现的指定元素（从头部到尾部遍历列表时）。



## JAVA核心技术 -- 卷一

### 第二章：程序设计环境

javac程序：java编译器，将xxx.java编译成xxx.class；

java程序：启动java虚拟机jvm，虚拟机执行编译器放在class文件中的**字节码**；

### 第三章：java的基本程序设计结构

数据类型：java是一种强类型语言，8种数据类型primitive type，包括4种整型、2种浮点类型、1个保存unicode编码的字符单元的字符类型、1个boolean；

![image-20200120155053710](/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200120155053710.png)

byte --> short --> int --> long

1 --> 2 --> 4 --> 8 字节

![image-20200120155231139](/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200120155231139.png)

```java
// 正整数 / 0，正无穷大；
// 0/0 或 负数开方 ，NaN;
// 整数 / 0，异常；
// 浮点数 / 0， 正无穷或者NAN；
```

> **更多异常情况需要明确**

Char类型： **单引号！！！** 转义字符 

> char和unicode之间的关系没看懂？？？

变量：变量声明后需要初始化，不可以使用未经初始化的变量，会报错！

常量：使用**final**表示，常量名全大写

```JAVA
final PI;  // 单个方法中定义使用
static final PI; // 类中共用
public static final PI; // 外部可以用
```

**使用strictfp标记的方法，必须遵循严格的浮点运算

![image-20200120160832047](/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200120160832047.png)

![image-20200120161214510](/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200120161214510.png)

![image-20200120161439762](/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200120161439762.png)

字符串：java没有内置的字符串类型，而是在标准java类库中定义了一个预定义类String，**双引号**

不能修改字符串中的字符，不可变字符串的优点：**编译器可让字符串共享**

不能使用==来判断字符串相等，使用 equals()

```java
// 检查是否是空串或者null
if (str != null && str.length()==0)
```

length方法返回**代码单元**数量；codePointCount(0,str.length())返回实际**码点数量**；

charAt(n)返回位置n的**代码单元**；

int index = str.offsetByCodePoints (0,i) ; int cp = str.codePointAt (index) ;返回第i个**码点**；

```java
// 遍历一个字符串，并且依次査看每一个码点，可以使用下列语句：
int cp = sentence.codePointAt(i); 
if (Character.isSupplementaryCodePoint(cp)) i += 2;
else i ++ ;

// 可以使用下列语句实现回退操作 ：
i-- ；
if (Character.isSurrogate(sentence.charAt(i))) i--;
int cp = sentence.codePointAt(i);

//string api
char charAt(int index);
int codePointAt(int index);
int offsetByCodePoints(int startPoint, int cpCount);
int compareTo(String other);
IntStream codePoints();
new String(int[] codePoints, int offset, int count);
boolean equals(Object other);
boolean equalsIgnoreCase(String other);
boolean startsWith(String pre);
boolean endsWith(String sub);
int indexOf(String str, [int fromIndex]);
int lastIndexOf(String str, [int fromIndex]);
int length(String str);
int codePointCount(int startIndex, int endIndex);
String replace(String/Stringbuilder old, String newStr);
String substring();
String trim();
String join();
```

**stringBuilder vs stringBuffer**

stringBuffer 效率较低，但允许多线程调用；stringBuilder 效率较高，通常使用stringBuilder

```java
//stringBuilder api
StringBuilder();
int lenght();
StringBuilder append(String str);
StringBuilder appendCodePoint(int cp);
void setCharAt(int i, char c);
StringBuilder insert(int offset, String str);
StringBuilder delete(int startIndex, int endIndex);
String toString();
```

**输入输出**

```java
Scanner in = new Scanner(System.in);
String name = in.nextLine();
in.next(); in.nextInt(); in.nextDouble();
hasNext(); hasNextInt(); hasNextDouble();
```

![image-20200121115023925](/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200121115023925.png)

*时间类输出*

文件输入和输出：Scanner in = new Scanner(Paths.get(xxx.txt),"UTF-8");

PrintWriter out = new PrintWriter("xxx.txt","UTF-8");

**JAVA 不同于C++，不能在嵌套块中重复声明变量**

**大数值**：BigInteger 任意精度的整数运算；BigDecimal 任意精度的浮点数运算；

BigInteger a = BigInteger.valueOf(100); 不能进行运算符重载，所以要使用add，multiply等方法

```java
BI add(BI);
BI subtract(BI);
BI multiply(BI);
BI divide(BI);
BI mod(BI);
int compareTo(BI);
```

**数组**：数组长度不要求是常量，可以为n；初始化后，数字默认为0，boolean默认为false，对象（字符串）默认为null；数组一旦创建，**不可以改变大小**；可扩展大小的数组--》数组列表arraylist；

java中的for each： for（ele：list）{}

```java
int [ ] small Primes = { 2 , 3 , 5 , 7 , 11 , 13 } ;
small Primes = new int [ ] { 17 , 19 , 23 , 29 , 31 , 37 };
```

直接赋值，两个变量指向同一个数组；Arrays.copyOf(nums ,newlength) ;深入拷贝，通常用于增长数组。

命令行参数：main()方法接受一个命令行参数数组String[] args = (java xxx) **-h xcxcxsdfsdf**

```java
Arrays.sort(); //优化的快速排序算法
copyOf(arr[], lenght);
copyOfRange(arr[], start, end);
binarySearch(arr[], [start, end,] target); //二分查找
fill(arr[], target); // 将数组所有值都设为target
equals(arr[],arr2[]); // 数组长度相等，每个对应元素相等；
```

### 第四章：对象与类

**面向对象程序设计**：OOP

类构造对象，创建类的实例

**类之间的关系**：依赖关系、聚合关系、继承关系：

依赖：一个类的方法操控另一个类的对象。**耦合**

聚合：类a中有个量就是类b

继承：特殊与一般的关系，

并不是所有类都具有面向对象特征，math类只封装了功能，没有数据

**构造器**：java使用构造器创造新实例，构造器是一种特殊的方法，用来构造并初始化对象实例。构造器的名字与类名相同。对象变量可以引用该类型的对象。

如果将一个方法作用于null，则会产生运行时错误；局部变量不会自动变为null

**时间Date**：UTC 1970年1月1日00:00:00 不同于GMT

源文件名是EmployeeTest.java，文件名必须与 public 类的名字相匹配。在一个源文件中，只能有一个公有类，但可以有任意数目的非公有类。编译器编译这段源代码会产生多个xxx.class文件。

**静态域与静态方法**

static：静态域，类域，属于类，不属于任何一个对象，但是任何一个对象都有一个拷贝；

静态常量：

静态方法：不能向对象实施操作，可以通过类名直接调用，而不是对象；特殊使用--工厂方法，无法命名构造器的时候，需要改变返回的类型的时候；

main方法：静态的main方法将执行并创建程序所需要的对象；每个类都可以有一个main方法，

**方法参数**：java总是按值调用；但是可以通过对象引用的拷贝来改变对象的值；

**对象构造**：java允许任何方法的重载，重载指方法名相同，参数不同，不考虑返回类型！没有构造器的情况下，会默认提供无参构造器；只要提供了构造器，就不会默认给出无参数构造器；

实例域初始化并不一定是常量，还可能调用函数；

this ( " Employee # " + nextld , s ) ;当调用 new Employee (60000) 时 Employee (double) 构造器将调用Employee (String, double)，构造器第一行调用另一个构造器

初始化代码块优先于构造器；

**包**：一个类可以使用所属包中的所有类，以及其他包中的公有类（public class)；只能使用星号 （ * ) 导入一个包

```java
javac com/myconipany/Payrol1App.java 
java com.mycompany.PayrollApp
```

**包作用域**：

+ public 的部分可以被任意的类使 。 
+ private 的部分只能被定义它们的类使用
+ protected的部分本包和所有子类可见 
+ 默认：没有指定 public 或 private , 这个部分（ 类、方法或变量 ）可以被同一个包中的所有方法访问

**类路径是所有包含类文件的路径的集合**

**类路径的设置？？？**

类注释：放在import之后，类定义之前；

域注释：只需要对共有域（静态常量）

### 第五章：继承

extends表示继承。所有继承都是公有继承。

**超类**、基类、父类 -- 我是你爹！！

**子类**、孩子类、派生类 -- 崽崽！！

```java
// 覆盖超类的方法，调用使用super
// 崽崽可以增加域、增加方法、覆盖爹爹的方法，但是不能删除爹爹的域和方法
// 爸爸的私房钱你也不能动，超类的私有方法不能被访问
public double getSalary(){ 
	double baseSalary = super.getSalary();
	return baseSalary + bonus ;
}
super(n ,s ,year ,month ,day);
// 调用超类 Employee 中含有 n s year month 和 day 参数的构造器
// super 调用构造器的语句必须是子类构造器的第一条语句
// 如果子类的构造器没有显式地调用超类的构造器，则将自动地调用超类默认（ 没有参数 ) 的构造器
```

爹爹类变量，可以引用爹爹类对象，也可以引用崽崽类对象 ——**多态**

在运行时能够自动地选择调用哪个方法的现象，具有虚拟特征——**动态绑定**

由一个公共超类（你大爷）派生出来的所有类的集合——**继承层次**

从某个特定的类到其祖先的路径被称为该类的**——继承链**

**java不支持多继承！！！！！**没有隔壁老王，你只有一个爹，如何实现多个爸爸，使用接口？？

对象变量是多态的。（祖传的房子能装子子孙孙好多人啊哈哈哈

但是崽崽在爹爹的家里不能乱搞，所以超类变量引用的字类对象不能使用子类方法

不能将超类对象引用赋给字类变量。崽崽房子太小了呜呜呜崽不配

**静态绑定**？？动态绑定和静态绑定有何区别？

如果是private、static、final方法，则是静态绑定

子类方法不能低于超类方法的可见性

final类，有final方法，但是没有final域；

只能在继承层次内进行类型转换， 在将超类转换成子类之前，应该使用instanceof进行检查。

**abstract：包含一个或多个抽象方法的类本身必须被声明为抽象的**

扩展抽象类的两种选择：*什么鬼话没看懂*

1. 在抽象类种定义部分抽象类方啊或不定义抽象类方法，将子类也标记为抽象类；
2. 定义全部的抽象方法，子类就不是抽象类

抽象类不能被实例化，但是可以定义抽象类的对象变量，这个变量只能引用非抽象子类。

抽象类的用途，就是可以将阿猫阿狗都放在动物数组里，抽象类中定义的抽象方法在子类中被定义，抽象变量引用子类对象，就可以调用该方法。但是阿猫阿狗的同名方法内容各不相同。

protected：受保护的域和方法可以被子类访问，但对其他类来说是私有的；常用于保护爸爸的方法；受保护的部分，子类和同一个包中的类都可见！！一家人不见外哈哈哈哈

**老祖宗Object（我是所有人的爸爸哈哈哈哈**

没有明确爸爸的崽崽，都是Object的乖儿子

equals：自带的equals方法会两个对象是否是相同的引用，所以实际上没啥用，需要检测两个对象状态的相等性；

getclass方法返回对象所属的类

相等测试与继承：

1. 自反性 x==x
2. 对称性 x==y y==x
3. 传递性 x==y y==z 所以x==z
4. getClass 和 instanceof进行检测；

```java
class Person
{
    private String name;
    private int age;

    public String getName(){   return name;}   
    public int getAge(){   return age;}   
    @Override public boolean equals(Object otherObject){   
        if(this == otherObject) // 检测是否引用同一对象？
            return true;
        if(otherObject == null) // 如果other为null？
            return false;        
       if(getClass() != otherObject.getClass()) 
            return false; // 两者是否属于同一个类？
        Person other = (Person)otherObject;
      // 比较各个域？
        return (name.equals(other.name) && (age == other.age));
    }   
}
class Employee extends Person
{
    private String occupation;
		@Override 
  	public boolean equals(Object otherObject){   
        if(this == otherObject)
            return true;
        if(otherObject == null)
            return false;
        if(getClass() != otherObject.getClass())
            return false;
        if(!super.equals(otherObject))
            return false;
        Employee other = (Employee)otherObject;
        return occupation.equals(other.occupation);
     }
}
```

hashcode：散列码，由对象导出的一个整型值，为对象的存储地址

String字符串的散列码是由内容导出的，但是StringBuffer没有重新定义hashcode，是默认的object。

如果重新定义了equals方法，就应该重新定义hashcode，以便将对象插入到散列表中；

**数组的toString()方法没有用，要用Arrays.toString()**

ArraryList是一个采用类型参数的泛型类。arraylist.size() 类似array.length

对象包装器类是不可变的，一旦构建，不能改变其中的值，ArraryList<>尖括号内不允许是基本类型。所以list.add ( 3 ) ;将自动地变换成list.add(Integer.value0f(3)); 自动装箱：**autoboxing**

![image-20200131133052887](/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200131133052887.png)



**反射**

能够分析类能力的程序称为反射；

反射机制的作用：

1. 在运行时分析类的能力；
2. 在运行时查看对象，
3. 实现通用的数组操作代码；
4. 利用method对象，很像西嘉嘉中的函数指针。

在java运行时期间，java运行时系统始终为所有对象维护一个被称为运行时的类型标识，这个信息跟踪着每个对象所属的类。虚拟机利用运行时类型信息选择相应的方法执行。

#### 5.8 继承的技巧

+ 将公共操作和域放在超类中；
+ 不要使用受保护的域；
+ 使用继承实现“is-a”关系；
+ 除非所有继承的方法都有意义 ，否则不要使用继承除非所有继承的方法都有意义 ；
+ 在覆盖方法时 ，不要改变预期的行为；
+ 使用多态，而非类型信息；
+ 不要过多使用多态；

### 第六章：接口、lambda表达式与内部类

#### 6.1 接口

**接口**：接口不是类，而是对类的一组需求描述，并没有定义具体的实现。接口中的所有方法自动地属于public，不需要写。

接口不能有实例域，因为接口没有实例。接口中的域都默认为public static final常量。

**implement**

**每个类只能继承一个超类，但是可以实现多个接口**

接口和抽象类的区别：接口一个有好几个，只能继承一个抽象类。

java8 之后可以在接口中定义static静态方法，default默认方法（用于接口演化兼容性）。

> 默认方法冲突
>
> 超类优先 ，接口同名方法被忽略 》两个接口中方法冲突，即使有一个没有提供默认方法，也必须覆盖重写

#### 6.2 接口示例

> 接口与回调callback

实现了ActionListener接口，所以可以作为回调对象

> Comparator 接口

Arrays.sort(arr[], comparator);比较器是实现了Comparator接口的类的实例，它的compare方法有两个参数。

> 对象克隆

object的默认clone方法是浅拷贝，只拷贝了基本类型或者数值，没有克隆对象引用。

实现Cloneable接口，是一种标记接口

在java se 1.4 之前 clone 方法的返回类型总是 Object , 而现在可以为你的 clone 方法指定正确的返回类型，这是**协变返回类型**的一个例子 。

所有数组类型都有一个public的clone方法，可以使用！

#### 6.3 lambda表达式

如果一个 lambda 表达式只在某些分支返回一个值， 而在另外一些分支不返回值，这是不合法的 。

**函数式接口（functional interface)** ：对于只有一个抽象方法的接口，需要这种接口的对象时，就可以提供一个 lambda 表达式。

Arrays.sort(words , ( first , second ) -> first. length() - second .length() ) ;

#### 6.4 内部类

inner class就是定义在另一个类中的类。内部类可以直接访问外部类对象的数据域。

```java
OuterClass.this;
OuterClass.InnerClass innerClass = outerClass.new InnerClass();
```

**使用内部类最吸引人的原因是**：每个内部类都能**独立地继承一个（接口的）实现**，所以无论外围类是否已经继承了某个（接口的）实现，对于内部类都没有影响。

**局部内部类**

在方法中定义的内部类，不能用public或者private来修饰，作用域仅在当前块中。

局部内部类还可以访问局部变量（实际上是final的），局部内部类会备份这些局部变量。

**匿名内部类**

只创建这个类一个对象的时候，不需要命名。此时有时可以用lambda表达式来替代

```java
// 双括号初始化
invite(new ArrayList<String>(){{ add("harray");add("tom");}});
// 第一对括号建立了ArrayList的匿名子类，内层括号是一个对象构造块
```

**静态内部类**

只是希望把一个类隐藏在另一个类的内部，并不需要内部类引用外围类对象。此时，可以将内部类声明为static，以便取消产生的引用。



### 第七章：异常、断言和日志

#### 7.1 处理错误

Throwable ==>  Error 运行时系统的内部错误和资源耗尽错误；不该抛出，只能尽量安全的终止程序。

​					==> Exception ==> RuntimeException（程序本身导致）==> 非受查异常  <=//

​											  ==> 其他（IO等导致的）                      ==> 受查异常（checked）编译器会检查





### 第八章：泛型程序设计



### 第九章：集合

#### 9.1 Java集合框架



#### 9.4 视图和包装器

视图：keySet方法返回了一个实现Set接口的类对象，通过这个类的方法可以对原来的Map对象进行操作。

**轻量级集合包装器**

Arrays.asList()方法，返回的不是一个ArrayList对象，而是一个视图对象，这个对象可以get、set，但是不能进行改变数组大小的操作（add、remove）。

类似的例子还有Collections中的nCopies()方法，返回的List集合对象是一个视图，不能修改大小，存储代价很小。

```java
List<String> stringlist = Collections.nCpies(100,"hehe");
List<String> stringlist = Arrays.asList(strings);
```

**子范围**

给集合建立子范围视图，子范围清空之后，原本的集合中对应的元素也会不见。

```java
List<String> list = new ArrayList<>();
List<String> sublist = list.sublist(9,20);
```

**不可修改的视图**

Collections中的几个方法，可以对集合创建不可修改的视图（set也不可以），否则会抛出异常。

```java
Collections.unmodifiableCollection();
Collections.unmodifiableList();
Collections.unmodifiableSet();
Collections.unmodifiableSortedSet();
// 等等，只是后面的名字改了，后面的名字就代表视图实现了的接口
```

**同步视图**								

如果有多个线程访问集合，可以确保集合不被意外的破坏，此时没必要使用线程安全的集合类。用一个包装类包装了Map，对所有方法都用synchronized加锁，这样获得的线程安全性比concurrent集合要低很多。

```java
Map<String, String> map = Collections.synchrizedMap(new HashMap<String, String>());
```

**受查视图**

Collections中的一个方法，可以保证检查类型

```java
List<String> list2 = Collections.checkedList(list, String.class);
List List3 = list2; //类型擦除
list.add(10); // 这里运行时会报错，classCastException
```











































## 知识点

1. JDK开发工具包，包含JRE和编译debug等开发时需要的工具。JRE运行时环境又包括虚拟机和运行时所需要的核心类库（JVM+Runtime Library）。

2. 8种基础类型：4个整型byte、short、int、long；2个浮点类型float、double；1个字符类型char；一个布尔类型boolean。java没有无符号数！

3. 单个java文件中可以有一个public类和多个非公有类，在编译的时候会变成多个class文件，解释器运行包含main方法的类以启动程序。

4. 构造器与类同名，可以有多个构造器，多个参数，没有返回值，总是伴随new一起调用；

5. 隐式参数：方法的调用/接受者，用关键字this来表示；显式参数：括号里的参数。

6. 一个方法可以访问所属类的所有对象的私有数据。

7. final域是在构造对象的时候就初始化了。

8. 静态域属于类，静态方法可以调用静态域，但是不能访问实例域。静态方法可以用来作为工厂方法，构造函数无法改名，而希望不同的构造函数取不同的名字的情况，或者需要改变返回值类型的情况下使用。

9. Java总是按值调用，call by value。方法不能修改传递给它的任何参数变量的内容。但是参数是一个对象引用的时候，方法可以改变所引用对象的内容。因为对象引用和对象引用的拷贝都指向同一个对象。

   即：不能修改基本数据类型的参数，不能让对象参数引用一个新的对象，但是可以改变一个对象参数的状态。

10. **重载overloading**：相同名字、不同参数

    方法签名：方法名和参数类型

11. 对象构造步骤：所有数据被赋予默认值；按照类中声明的顺序，依次执行所有域初始化语句和初始化块；构造器第一行如果调用了第二个构造器，先执行第二个；执行本构造器。

12. 如果源文件中没有写package语句，这个源文件中的类就被放置在一个默认包中。默认包是一个没有名字的包。

13. **覆盖override**：

14. 子类**不能直接访问**超类的私有域，但是可以使用 **super.getxxx()**  来调用

15. 子类构造器，可以显式的使用super构造器来对超类的私有域进行赋值；否则自动调用超类默认构造器（无参数的）

16. 只能在继承层次内进行类型转换；在将超类转换成子类之前，应该使用instanceof进行检查。

17. 只有抽象的类，里面才会有抽象的方法，也可能还有别的具体域或者具体方法。

    抽象方法，是一个占位墩子，他的具体实现在子类中。

    不含抽象方法，也可以是抽象类。

    抽象类不能创建实例，但是可以引用非抽象子类。

18. ArrayList定义的时候也可以带上容量，这样能够一定程度提高效率。

19. -128～127之间的short和int被包装到固定的对象中！！！

20. 数量可变的参数：`（double... values）`

21. 枚举类型`Enum`：每个枚举类型都有一个静态的 values 方法 ，它将返回一个包含全部枚举值的数组。

     ordinal 方 法 返 冋 enum 声 明 中 枚 举 常 量 的 位 置 ，

    ```java
    Size size = Enum.valueOf( Size.class, input);
    Size[] values = Size values() ;
    Size.MEDIUN.ordinal() // 返回
    ```

22. 反射，可以动态操纵java代码

23. **接口：**接口不是类，而是对类的一种需求描述。接口中的方法自动属于public。接口中没有实例，但是可以有常量，默认为public static final，现在可以实现一些简单的方法（static或者default）。**使用接口不用抽象类的原因：接口可以继承多个，超类却只有一个。**

24. Arrays.sort()可以对对象数组进行排序，前提是对象实现了Comparable接口（compareTo方法）。整型可以直接相减，浮点数需要用double.compare(x,y)。

    在不能修改对象的comparable的情况下，可以另外实现Comparator接口中的compare方法，comparator作为sort()方法的第二个参数。

25. 接口中可以不声明public，但是实现中必须写public。

26. 接口默认方法、超类同名方法、另一接口同名方法，并存的时候，**超类优先**，接口冲突的时候（只要有一个接口提供默认实现）必须覆盖这个方法来解决冲突。两个接口都没有提供默认实现的时候，根本就不存在冲突。

27. 接口回调：就是把接口作为方法的参数，让具备该方法类A可以使用该接口的实现类。此时，接口作为一个中间件，成为A和实现类之间的中介。

28. labmda表达式的用法：函数式接口

    方法引用：System.out::println  String::compareToIgnoreCase   this::equals   super::greet

    构造器引用：Person::new

    ```java
    // 构造器引用
    ArrayList<String> names = ...; // 使用string数组来创建Person List
    List<Person> people = names.stream().map(Person::new).collect(Collectors.toList());
    
    // 使用构造器引用，将stream流转化为T[n] 数组
    Person[] people = stream.toArray(Person[]::new);
    ```

    一个lambda表达式，包括一个代码块、参数、自由变量的值（非参数且不定义在代码中）。此时，lambda表达式必须存储自由变量的值，**捕获**。

29. ![image-20200522135114607](/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200522135114607.png)

    常用函数式接口的使用方法：

    ```java
    // consumer<T> 消费型接口
    // 	void accept(T) 有参数无返回
    Consumer<String> con1 = s->System.out.println("name"+s.split(",")[0]);
    Consumer<String> con2 = s->System.out.println("color"+s.split(",")[1]);
    String[] strings = {"jam, while", "farm, black"};
    for(String s: strings){
      con1.andThen(con2).accept(s);
    }
    // supplier<T> 供给型接口
    // T get() 无参数有返回
    Supplier<String> supplier = () -> "woshifanhuizhi";
    System.out.println(supplier.get());
    // function<T,R> 函数型接口
    // R apply(T) 可以用来转变数据类型 compose(Function before) andThen(Function after)
    Function<Integer, Integer> fun1 = (e)-> e*6;
    Function<Integer, Integer> fun2 = e -> e * e;
    fun1.compose(fun2).apply(3);
    fun1.andThen(fun2).apply(3);
    // Predicate<T> 断言型接口
    // boolean test(T t) and(Predicate ) or(Predicate)
    
    ```

30. **内部类**：

    内部类和静态内部类的区别：

31. **代理**

32. Exception和RuntimeException区别：Exception可能是checked exception，强制要求被处理；而runtimeException可以不被处理，是unchecked exception。自定义exception的时候，继承哪一个都要根据实际情况来判断，如果是必须处理的异常，那么就继承exception。

33. 







