## Netty、Redis、Zookeeper高并发实战

Java NIO、Reactor模式、高性能通信、分布式锁、分布式ID、分布式缓存、高并发架构等技术相关的面试题

开发Java项目所必需的技术栈：分布式Java框架、Redis缓存、分布式搜索ElasticSearch、分布式协调ZooKeeper、消息队列Kafka、高性能通信框架Netty。

普通Web，QPS（Query per second，每秒查询率）峰值看你在1000以内，属于重复性的CRUD体力活。

分布式、高并发的IM系统，面临的QPS峰值可能在十万、百万、甚至上亿级别。

### **IM：Instant Messaging 即时通讯**

一切高实时性通信、消息推送的应用场景，都需要高并发IM。高并发典型的应用场景如下：**私信、聊天、大规模推送、视频会议、弹幕、抽奖、互动游戏、基于位置的应用、在线教育、智能居家、互动游戏**。



### 一. 高并发IO的底层原理

#### IO读写的基础原理

用户程序进行IO的读写，依赖于底层的IO读写，基本上会用到底层的read&write两大系统调用。

上层应用无论是调用操作系统的read，还是调用操作系统的write，都会涉及缓冲区。**具体来说，调用操作系统的read，是把数据从内核缓冲区复制到进程缓冲区；而write系统调用，是把数据从进程缓冲区复制到内核缓冲区。**

上层程序的IO操作，**实际上不是物理设备级别的读写，而是缓存的复制**。

而内核缓冲区和物理设备（如磁盘）之间的交换，是由操作系统内核（Kernel）来完成的。

<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200702153745291.png" alt="image-20200702153745291" style="zoom:40%;" />

是在Java服务器端，完成一次socket请求和响应，完整的流程如下：

1.  客户端请求：Linux通过网卡读取客户端的请求数据，将数据读取到内核缓冲区。
2. 获取请求数据：Java服务器通过read系统调用，从Linux内核缓冲区读取数据，再送入Java进程缓冲区。
3. 服务器端业务处理：Java服务器在自己的用户空间中处理客户端的请求。
4. 服务器端返回数据：Java服务器完成处理后，构建好的响应数据，将这些数据从用户缓冲区写入内核缓冲区。这里用到的是write系统调用。
5. 发送给客户端：Linux内核通过网络IO，将内核缓冲区中的数据写入网卡，网卡通过底层的通信协议，会将数据发送给目标客户端。



#### 四种主要的IO模型

**同步vs异步**：同步---- 用户空间的线程是主动发起IO请求的一方，内核空间是被动接受方；异步IO是反过来，系统内核是主动发起IO的一方，用户空间的线程是被动接受方。

**阻塞vs非阻塞**：阻塞----需要内核IO操作彻底完成之后，才返回到用户空间执行用户的操作，阻塞的是用户空间程序的执行状态；在java中默认创建的socket都是阻塞的。

+ 同步阻塞IO：BIO
  + 优点：程序简单，在阻塞期间，用户线程不会占用CPU资源；
  + 缺点：一个线程维护一个连接的IO操作，在高并发的场景下，线程切换开销巨大；
+ 同步非阻塞IO：NIO
  + 在内核缓冲区没有数据的情况下，系统调用会立即返回一个调用失败的信息；
  + 在内核缓冲区中有数据的情况下，是阻塞的，直到数据从内核缓冲复制到用户进程缓冲。复制完成后，系统返回调用成功。
  + 所以为了读取到最终的数据，用户线程需要不断的发起IO系统调用；轮询数据是否准备好。
  + 优点：每次发起IO系统调用，在内核等待数据的过程中可以立即返回，用户线程不会阻塞，实时性较好；
  + 缺点：不断的轮询内核，将占用大量的CPU时间，效率低下。一般不会使用。
+ IO多路复用：IO Multiplexing
  + **Java中的NIO（new IO）**
  + 在IO多路复用模型中，**引入了一种新的系统调用，查询IO的就绪状态。**在Linux系统中，对应的系统调用为select/epoll系统调用。
  + 通过该系统调用，一个进程可以监视多个文件描述符，一旦某个描述符就绪（一般是内核缓冲区可读/可写），内核能够将就绪的状态返回给应用程序。随后，应用程序根据就绪的状态，进行相应的IO系统调用。
  + 优点：使用select/epoll的最大优势在于，一个选择器查询线程可以同时处理成千上万个连接connection。系统不必创建大量的线程，减少了系统开销。
  + 缺点：本质上，select/epoll系统调用是阻塞的，属于同步IO。还是需要在读写事件就绪之后，由系统调用本身负责进行读写，也就是说这个读写的过程是阻塞的。
+ 异步IO：AIO
  + 用户线程通过系统调用，向内核注册某个IO操作。内核在整个IO（包括数据准备、数据复制）完成之后，通知用户程序，用户执行后续的业务操作。
  + 优点：在内核等待数据、复制数据的两个阶段，用户线程都**不是阻塞的**。用户线程需要接收内核的IO操作完成的事件，或者用户线程需要注册一个IO操作完成的回调函数。正因为如此，异步IO有的时候也被称为信号驱动IO。
  + 缺点：需要操作系统底层内核提供支持。**windows系统通过IOCP实现了真正的异步IO；但是Linux系统的异步IO底层实现仍然使用epoll，与多路复用是一样的。**

#### select/poll/epoll比较

+ poll的实现和select非常相似，只是描述fd集合的方式不同，poll使用pollfd结构而不是select的fd_set结构；并且poll没有最大文件描述符数量的限制。
+ poll和select同样存在一个缺点就是，包含大量文件描述符的数组被整体复制于用户态和内核的地址空间之间，而不论这些文件描述符是否就绪，它的开销随着文件描述符数量的增加而线性增大。
+ 在 select/poll中，进程只有在调用一定的方法后，内核才对所有监视的文件描述符进行扫描，而epoll事先通过epoll_ctl()来注册一 个文件描述符，一旦基于某个文件描述符就绪时，内核会采用类似callback的回调机制，迅速激活这个文件描述符，当进程调用epoll_wait() 时便得到通知。(此处去掉了遍历文件描述符，而是**通过监听回调的的机制**。这正是epoll的魅力所在。)
+ 表面上看epoll的性能最好，但是在连接数少并且连接都十分活跃的情况下，select和poll的性能可能比epoll好，毕竟epoll的通知机制需要很多函数回调。
+ select低效是因为每次它都需要轮询。但低效也是相对的，视情况而定，也可通过良好的设计改善 



#### Linux并发配置

**文件句柄，也叫文件描述符**。在Linux系统中，文件可分为：普通文件、目录文件、链接文件和设备文件。文件描述符（File Descriptor）是内核为了高效管理已被打开的文件所创建的索引，它是一个非负整数（通常是小整数），**用于指代被打开的文件**。所有的IO系统调用，包括socket的读写调用，都是通过文件描述符完成的。

在生产环境Linux系统中，基本上都需要解除文件句柄数的限制。原因是，Linux的系统默认值为1024，也就是说，一个进程最多可以接受1024个socket连接。这是远远不够的。

```sh
ulimit -n
#显示和修改当前用户进程一些基础限制的命令，-n命令选项用于引用或设置当前的文件句柄数量的限制值。
#在终端中使用
ulimit -n 1000000
#只能在当前终端有效，一旦断开连接，就变回去了
#如果想永久地把设置值保存下来，可以root用户权限下编辑/etc/rc.local开机启动文件，在文件中添加如下内容：
#选项-S表示软性极限值，-H表示硬性极限值
ulimit -SHn 1000000

#终极解除Linux系统的最大文件打开数量的限制，可以通过编辑Linux的极限配置文件/etc/security/limits.conf来解决，修改此文件，加入以下内容
soft nofile 1000000
hard mofile 1000000
```

### 二. Java NIO通信基础详解

现在主流的技术框架或中间件服务器，都使用了Java NIO技术，譬如Tomcat、Jetty、Netty。

#### Java NIO简介

1.4版本之后出现的，非阻塞IO。

有三大核心组件：

+ Channel 通道
+ Buffer 缓冲区
+ Selector 选择器

##### NIO和OIO的区别

+ OIO是面向流的，NIO是面向缓冲区的：

  OIO是面向字节流或者字符流的，在一般的OIO操作中，我们以流式的方式顺序地从一个流中读取一个或多个字节，因此我们不能随意的改变读取指针的位置。而在NIO操作中，NIO引入流Channel和Buffer的概念，读取和写入，只需要从通道中读取数据到缓冲区，或者将数据从缓冲区写入通道，可以随意读取buffer中任意位置的数据。

+ OIO是阻塞的，NIO是非阻塞的：OIO中，调用read方法的线程会被阻塞，直到read操作完成。而在NIO的非阻塞中，read方法没有读取到数据，会直接返回。因为它使用了通道和通道的多路复用技术。
+ OIO没有选择器selector的概念，而NIO基于底层操作系统实现了选择器。



#### Buffer缓冲区

应用程序与通道（Channel）主要的交互操作，就是进行数据的read读取和write写入。

为了完成如此大任，NIO为大家准备了第三个重要的组件——NIO Buffer（NIO缓冲区）。通道的读取，就是将数据从通道读取到缓冲区中；通道的写入，就是将数据从缓冲区中写入到通道中。

Buffer类是一个抽象类，其内部是一个内存块（数组）。与普通数组不同的是，Buffer对象提供了一组更加有效的方法，用来进行写入和读取的交替访问。**Buffer类是一个非线程安全类**

**<u>Buffer的具体实现</u>**

8种缓冲区分类：ByteBuffer、CharBuffer、DoubleBuffer、FloatBuffer、IntBuffer、LongBuffer、ShortBuffer、MappedByteBuffer（专门用于内存映射的一种ByteBuffer类型）

**<u>Buffer的重要属性</u>**

capacity（容量）：表示内部容量的大小。一旦写入的对象数量超过了capacity容量，缓冲区就满了，不能再写入了。一旦初始化，就不能再改变。

position（读写位置）：表示当前的位置。position属性与缓冲区的读写模式有关。在不同的模式下，position属性的值是不同的。当缓冲区进行读写的模式改变时，position会进行调整。

limit（读写的限制）：表示读写的最大上限。在写模式下，limit属性值的含义为可以写入的数据最大上限。在刚进入到写模式时，limit的值会被设置成缓冲区的capacity容量值，表示可以一直将缓冲区的容量写满。在读模式下，limit的值含义为最多能从缓冲区中读取到多少数据。

mark（标记），可以将当前的position临时存入mark中；需要的时候，可以再从mark标记恢复到position位置。

**<u>Buffer的重要方法</u>**

```java
IntBuffer intBuffer = IntBuffer.allocate(20);
//创建了一个Intbuffer实例对象，并且分配了20 * 4个字节的内存空间。
intBuffer.put(1);
//写入数据
intBuffer.flip();
//翻转缓冲区，从写模式变成读模式
intBuffer.clear();//清空，
//在读取模式下，调用clear()方法将缓冲区切换为写入模式。此方法会将position清零，limit设置为capacity最大容量值，可以一直写入，直到缓冲区写满。
intBuffer.compact();//压缩
//缓冲区转换为写模式
int j = intBuffer.get();
//读数据，当position和limit数据相等，表示所有的数据读取完成，此时再读，会抛出BufferUnderflowException异常。
intBuffer.rewind();
// （1）position重置为0，所以可以重读缓冲区中的所有数据。（2）limit保持不变，数据量还是一样的，仍然表示能从缓冲区中读取多少个元素。（3）mark标记被清理，表示之前的临时位置不能再用了。
intBuffer.mark();
intBuffer.reset(); // 把前面保存的mark中的值回复到position
// 两个方法配套使用
```

**缓冲区可以重复读取！！**



#### Channel通道

在OIO中，同一个网络连接会关联到两个流：一个输入流，一个输出流。通过两个流，不断进行输入和输出的操作。

在NIO中，同一个网络连接使用一个通道表示，所有的NIO的IO操作都是从通道开始的。一个通道类似于OIO中的两个流的结合体，既可以从通道读取，也可以向通道写入。

**<u>Channel具体实现</u>**

##### **FileChannel文件通道**

用于文件的数据读写。可以从一个文件中读取数据，也可以将数据写入到文件中。

**只能是阻塞模式，不能设置为非阻塞**

```java
//获取FileChannel通道
FileInputStream fis = new FileInputStream(srcFile); // 创建一条文件输入流
FileOutputStream fos = new FileOutputStream(destFile);// 创建一条文件输出流
FileChannel inchannel = fis.getChannel();
FileChannel outchannel = fos.getChannel(); // 获取文件流的通道
// 也可以通过文件随机访问类，获取文件通道
RandomAccessFile aFile = new RandomAccessFile("filename.txt", "rw");
FileChannel inchannel = aFile.getChannel();

// 读取FileChannel通道
//从通道读取数据都会调用通道的int read（ByteBufferbuf）方法，它从通道读取到数据写入到ByteBuffer缓冲区，并且返回读取到的数据量。
ByteBuffer buf = ByteBuffer.allocate(Capacity);
int length = -1;
while((length = inChannel.read(buf))!= -1){
  ////////
}

//写入数据到通道
buf.flip();
int outlength = 0;
while((outlength = outchannel.write(buf)) != 0){
  System.out.println("写入的字节数" + outlength);
}

// 关闭通道
outchannel.close();
// 强制刷新到磁盘
outchannel.force(true);

// 复制文件
long size = inChannel.size();
long pos = 0;
long count = 0;
while (pos < size) {
    //每次复制最多1024个字节，没有就复制剩余的
    count = size - pos > 1024 ? 1024 : size - pos;
    //复制内存,偏移量pos + count长度
    pos += outChannel.transferFrom(inChannel, pos, count);
}

//强制刷新磁盘
outChannel.force(true);
```



##### **SocketChannel套接字通道**

用于Socket套接字TCP连接的数据读写。

**ServerSocketChannel服务器嵌套字通道**（或服务器监听通道），允许我们监听TCP连接请求，为每个监听到的请求，创建一个SocketChannel套接字通道。

所以，一个连接，两端都有一个负责传输的socketChannel传输通道。这两个都支持阻塞和非阻塞两种模式。

socketChannel的configureBlocking方法，（false）表示非阻塞模式。

在阻塞模式下，socketChannel通道的connect、read、write操作都是同步和阻塞的。在效率上与OIO面向流的阻塞式读写操作相同。

在非阻塞模式下，通道的建立，读取，关闭：

```java
// 获取socketChannel传输通道
// 在客户端，先通过socketChannel的静态方法open()获得一个套接字传输通道；
SocketChannel socketChannel = SocketChannel.open();
// 设置为非阻塞的模式
socketChannel.configureBlocking(false);
// 对服务器的IP和端口发起连接
socketChannel.connect(new InetSocketAddress("127.0.0.1",80));
// 非阻塞的情况下，与服务器的连接可能还没有真正建立，socketChannel.connect方法就返回了，因此需要不断自旋，检查是否连接了
while(!socketChannel.finishConnect()){
  // xxxxxxxxxx
}

// 在服务端，新连接事件到来，首先通过事件获取服务器监听通道
ServerSocketChannel server = (ServerSocketChannel) key.channel();
// 	获取新连接的 套接字通道
ServerChannel socketChannel = server.accept();
// 设置为非阻塞模式
socketChannel.configureBlocking(false);
```

```java
// 读取
ByteBuffer buf = ByteBuffer.allocate(1024);
int bytesRead = socketChannel.read(buf);

// 写入
buf.flip();
socketChannel.write(buf);

// 关闭
// 终止输出方法，向对方发送一个输出的结束标志，关闭套接字连接
socketChannel.shutdownOutput();
IOUtil.closeQuitely(socketChannel);
public static void closeQuietly(java.io.Closeable o){
    if (null == o) return;
    try{
        o.close();
    } catch (IOException e){
        e.printStackTrace();
    }
}
```

**使用SocketChannel发送文件的实践案例???**



##### **DatagramChannel数据报通道**

用于UDP协议的数据读写。

```java
DatagramChannel channel = DatagramChannel.open();
channel.configureBlocking(false);
// 调用bind方法绑定一个数据报的监听端口
channel.socket().bind(new InetSocketAddress("IP",18080));

// 读取
ByteBuffer buf = ByteBuffer.allocate(1024);
SocketAddress clientAddress = datagramChannel.receive(buf);
// 返回值是socket address类型，表示返回发送端的连接地址（IP和端口）

// 写入
buf.flip();
datagramChannel.send(buffer, new InetSocketAddress(Server_IP), Server_port);
buf.clear();
```



#### Selector选择器

什么是IO多路复用：一个进程/线程可以同时监视多个文件描述符（网络连接在操作系统底层也是使用文件描述符来表示的），一旦其中一个或者多个文件描述符可读或者可写，系统内核就通知该进程/线程。

在Java应用层面，是通过Selector实现对多个文件描述符的监视的。

Selector是一个IO事件查询器，一个线程可以查询多个通道的IO事件的就绪状态。

1. 获取选择器实例：静态工厂方法open()  Selector selector = Selector.open();

   > open方法内部，是通过默认的选择器服务提供者（SPI， service provider interface）对象请求，获取一个新的选择器实例。SPI是Java中一种可扩展的服务提供和发现机制，其他服务提供商可以通过SPI方式，提供定制化的选择器的动态替换或者扩展。

2. 把通道注册到选择器中 (Channel.register(Selector selctor, int options))

   options有4种IO事件类型可以选择：**表示通道具备完成某个IO操作的条件。**

   + 可读：SelectionKey.OP_READ
   + 可写：SelectionKey.OP_WRITE
   + 连接：SelectionKey.OP_CONNECT
   + 接收：SelectionKey.OP_ACCEPT
   + 按位或，多选：int key = SelectionKey.OP_READ | SelectionKey.OP_WRITE

   *<u>FileChannel文件通道，不可以被选择器监控。只有继承了SelectableChannel的通道才可以。</u>*

3. 然后使用Selector的select()方法来查询这些注册的通道是否有已经就绪的IO事件；将就绪的IO事件放入SelectorKey选择键的集合。

   选择键：就是被选择器选中的IO事件

   ```java
   While(selector.select() > 0){
     Set selectedKeys = selector.selectedKeys();
     Iterator keyIterator = selectedKeys.iterator();
     while(keyIterator.hasNext()){
       SelectionKey key = keyIterator.next();
       if(key.isAcceptable()){
         
       } else if(key.isConnectable()){
         
       } else if....
       keyIterator.remove();
     }
   }
   ```

   

##### Discard服务器实践

功能：仅仅读取客户端通道的输入数据，读取完成后直接关闭客户端通道；并且读取到的数据直接抛弃掉（Discard）。

##### 在服务器端接收文件的实践



### 三. Reactor反应器模式

Reactor反应器模式是高性能网络编程在设计和架构层面的基础模式。

到目前为止，高性能网络编程都绕不开反应器模式。很多著名的服务器软件或者中间件都是基于反应器模式实现的。Nginx、Redis、Netty都是基于反应器模式的。

#### 简介

反应器模式由Reactor反应器线程、Handlers处理器两大角色组成：

（1）Reactor反应器线程的职责：负责响应IO事件，并且分发到Handlers处理器。

（2）Handlers处理器的职责：非阻塞的执行业务处理逻辑。与IO事件（或者选择键）绑定，负责IO事件的处理。完成真正的连接建立、通道的读取、处理业务逻辑、负责将结果写出到通道等。

OIO最原始的网络服务器程序，用的是一个while循环，不断监听是否有新的连接。这种方法的最大问题在于，如果前一个网络连接的handle（socket）没有处理完成，那么后面的连接请求没办法被接受，于是后面的请求统统会被阻塞，服务器的吞吐量太低。

于是，Connection Per Thread模式（一个线程处理一个连接）出现，它的优点是：解决了前面的新连接被严重阻塞的问题，在一定程度上，极大地提高了服务器的吞吐量。但是实际上没有用，因为OIO还是阻塞的，后面的IO操作没有办法并发执行。而且，线程的反复创建、销毁、线程的切换也需要代价。

#### 单线程的Reactor反应器模式

单线程：Reactor反应器和Handlers处理器都处于同一个线程中。

<img src="/Users/chengleiyi/Library/Application Support/typora-user-images/image-20200704135432957.png" alt="image-20200704135432957" style="zoom:90%;" />

<u>void attach(Object o)：</u>

需要将Handler处理器实例，作为附件添加到SelectionKey实例，相当于setter

<u>Object attachment()：</u>

取出之前通过attach(Object o)添加到SelectionKey选择键实例的附件，相当于getter方法，与attach(Object o)配套使用。

在单线程反应器模式中，Reactor反应器和Handler处理器，都执行在同一条线程上。这样，带来了一个问题：当其中某个Handler阻塞时，会导致其他所有的Handler都得不到执行。另外，目前的服务器都是多核的，单线程反应器模式模型不能充分利用多核资源。总之，在高性能服务器应用场景中，单线程反应器模式实际使用的很少。



#### 多线程的Reactor反应器模式

多线程池Reactor反应器的演进，分为两个方面：

（1）首先是升级Handler处理器。既要使用多线程，又要尽可能的高效率，则可以考虑使用线程池。

（2）其次是升级Reactor反应器。可以考虑引入多个Selector选择器，提升选择大量通道的能力。

多线程池反应器的模式，大致如下：

（1）将负责输入输出处理的IOHandler处理器的执行，放入独立的线程池中。这样，业务处理线程与负责服务监听和IO事件查询的反应器线程相隔离，避免服务器的连接监听受到阻塞。

（2）如果服务器为多核的CPU，可以将反应器线程拆分为多个子反应器（SubReactor）线程；同时，引入多个选择器，每一个SubReactor子线程负责一个选择器。这样，充分释放了系统资源的能力；也提高了反应器管理大量连接，提升选择大量通道的能力。



#### 反应器模式VS其他模式

**反应器模式VS生产者消费者模式**

在一定程度上，反应器模式类似生产者消费者模式，但是在生产者消费者模式中，生产者将事件加入到一个队列中，一个或者多个消费者主动的从这个队列中提取事件来处理。而反应器模式，是基于查询的，没有专门的队列去缓冲存储IO事件，查询到事件之后就直接分发给处理器了。

**反应器模式VS观察者模式**

相似之处：观察者模式定义了一种依赖关系，发布/订阅的过程，让多个观察者同时监听同一个主题。区别在于：观察者模式的同一个主题可以被订阅过的多个观察者处理，而在反应器模式中，一个事件绑定一个Handler处理器。

反应器模式的优点如下：

· 响应快，虽然同一反应器线程本身是同步的，但不会被单个连接的同步IO所阻塞；· 编程相对简单，最大程度避免了复杂的多线程同步，也避免了多线程的各个进程之间切换的开销；· 可扩展，可以方便地通过增加反应器线程的个数来充分利用CPU资源。

反应器模式的缺点如下：

· 反应器模式增加了一定的复杂性，因而有一定的门槛，并且不易于调试。· 反应器模式需要操作系统底层的IO多路复用的支持，如Linux中的epoll。如果操作系统的底层不支持IO多路复用，反应器模式不会有那么高效。· 同一个Handler业务线程中，如果出现一个长时间的数据读写，会影响这个反应器中其他通道的IO处理。例如在大文件传输时，IO操作就会影响其他客户端（Client）的响应时间。因而对于这种操作，还需要进一步对反应器模式进行改进。



### 四. 异步回调技术

**<u>join异步阻塞：</u>**

join操作的原理是：阻塞当前的线程，直到准备合并的目标线程的执行完成。

#### FutureTask异步回调

为了获取异步线程的返回结果，Java在1.5版本之后提供了一种新的多线程的创建方式——FutureTask方式。FutureTask方式包含了一系列的Java相关的类，在java.util.concurrent包中。其中最为重要的是FutureTask类和Callable接口

**<u>Future接口</u>**

V get()：获取并发任务执行的结果。注意，这个方法是阻塞性的。如果并发任务没有执行完成，调用此方法的线程会一直阻塞，直到并发任务执行完成。

V get(Long timeout, TimeUnit unit)：获取并发任务执行的结果。也是阻塞性的，但是会有阻塞的时间限制，如果阻塞时间超过设定的timeout时间，该方法将抛出异常。

boolean isDone()：获取并发任务的执行状态。如果任务执行结束，则返回true。

boolean isCancelled()：获取并发任务的取消状态。如果任务完成前被取消，则返回true。

boolean cancel(booleanmayInterruptRunning)：取消并发任务的执行。

**通过FutureTask的get方法，确实可以获取异步结果，但是get方法本身是阻塞的，所有主线程还是会被阻塞。所以还是和join一样，都是异步阻塞模式。**

**如果需要实现非阻塞的一步结果获取方法，就要用到额外的框架，Guava、Netty**



#### Guava异步回调

总体来说，Guava的主要手段是增强而不是另起炉灶。为了实现非阻塞获取异步线程的结果，Guava对Java的异步回调机制，做了以下的增强：

（1）引入了一个新的接口**ListenableFuture**，继承了Java的Future接口，使得Java的Future异步任务，在Guava中能被监控和获得非阻塞异步执行的结果。实现异步任务Callable和FutureCallback结果回调之间的监控关系。

（2）引入了一个新的接口**FutureCallback**，这是一个独立的新接口。该接口的目的，是在异步任务执行完成后，根据异步结果，完成不同的回调处理，并且可以处理异步结果。

**FutureCallback**

这个接口，是用来填写异步任务执行完后的监听逻辑。拥有两个回调方法，onSuccess，onFailure

（1）onSuccess方法，在异步任务执行成功后被回调；调用时，异步任务的执行结果，作为onSuccess方法的参数被传入。

（2）onFailure方法，在异步任务执行过程中，抛出异常时被回调；调用时，异步任务所抛出的异常，作为onFailure方法的参数被传入。

**ListenableFuture**

ListenableFuture仅仅增加了一个方法——addListener方法。它的作用就是将前一小节的FutureCallback善后回调工作，封装成一个内部的Runnable异步回调任务，在Callable异步任务完成后，回调FutureCallback进行善后处理。

在实际编程中，如何将FutureCallback回调逻辑绑定到异步的ListenableFuture任务呢？可以使用Guava的Futures工具类，它有一个addCallback静态方法，可以将FutureCallback的回调实例绑定到ListenableFuture异步任务。下面是一个简单的绑定实例：

**总结一下，Guava异步回调的流程如下：**

第1步：实现Java的Callable接口，创建异步执行逻辑。还有一种情况，如果不需要返回值，异步执行逻辑也可以实现Java的Runnable接口。

第2步：创建Guava线程池。Guava线程池是对Java线程池的装饰。

第3步：将第1步创建的Callable/Runnable异步执行逻辑的实例，通过submit提交到Guava线程池，从而获取ListenableFuture异步任务实例。

第4步：创建FutureCallback回调实例，通过Futures.addCallback将回调实例绑定到ListenableFuture异步任务上。

完成以上四步，当Callable/Runnable异步执行逻辑完成后，就会回调异步回调实例FutureCallback的回调方法onSuccess/onFailure。



**Guava异步回调和Java的FutureTask异步回调，本质的不同在于：**

**· Guava是非阻塞的异步回调，调用线程是不阻塞的，可以继续执行自己的业务逻辑。**

**· FutureTask是阻塞的异步回调，调用线程是阻塞的，在获取异步结果的过程中，一直阻塞，等待异步线程返回结果。**



#### Netty的异步回调

Netty继承和扩展了JDK Future系列异步回调的API，定义了自身的Future系列接口和类，实现了异步任务的监控、异步执行结果的获取。

总体来说，Netty对JavaFuture异步任务的扩展如下：

（1）**继承Java的Future接口**（包名不同），得到了一个新的属于Netty自己的Future异步任务接口；该接口对原有的接口进行了增强，使得Netty异步任务能够以非阻塞的方式处理回调的结果；Netty的Future接口一般不会直接使用，而是会使用子接口：ChannelFuture通道IO操作的异步任务。

（2）引入了一个**新接口——GenericFutureListener**，用于表示异步执行完成的监听器。这个接口和Guava的FutureCallbak回调接口不同。Netty使用了监听器的模式，异步任务的执行完成后的回调逻辑抽象成了Listener监听器接口。可以将Netty的GenericFutureListener监听器接口加入Netty异步任务Future中，实现对异步任务执行状态的事件监听。**拥有一个回调方法operationComplete，表示异步任务操作完成。一般使用Netty中提供的某个子接口，如ChannelFutureListener接口**



### 五. Netty

Netty 是JBOSS提供的一个开源框架，是基于NIO的编程框架，可以开发高并发、高可用、高可靠性的程序。

Kafka、RocketMQ等消息中间件，ElasticSearch开源搜索引擎，大数据处理Hadoop的RPC框架Avro，主流的分布式通信框架Dubbo，都是使用Netty开发的应用。

Netty提供异步的、事件驱动的网络应用程序框架和工具。提供的所有IO操作都是异步非阻塞的，通过Future-Listener机制，用户可以方便地主动获取或者通过机制获得IO操作结果。

与JDK原生的NIO相比，Netty提供了相对十分简单易用的API，因而非常适合网络编程。

#### Netty对比Tomcat

最大的区别在于通信协议：Tomcat是基于Http协议的，实质是一个基于http协议的web容器。但是netty不一样，他能通过编程自定义各种协议，

如果是做TCP开发，netty是不二选择。现在的高并发分布式网站架构一般采用nginx+Netty/Tomcat。

Netty行业应用：

+ 作为异步高性能通信框架，作为基础通信组件，被RPC（remote procedure call）框架使用。例如Dubbo的RPC框架使用Dubbo协议进行节点间通信，Dubbo协议默认使用Netty作为基础通信组件，用于实现各节点之间的内部通信。

#### 5.1 反应器模式

**IO事件的处理流程**

+ 一个IO事件一定属于通道，所以要先把通道注册到选择器Selector中；Selector会查询到IO事件。
+ 反应器模式中的反应器会负责一个线程，轮询选择器Selector中的选择键--IO事件；
+ 如果查询到IO事件，则分发给与IO事件有绑定关系的Handler业务处理器；
+ 完成真正的IO操作和业务处理。

Netty中的反应器有多个实现类，与Channel通道类有关系。对应于NioSocketChannel通道，Netty的反应器类为：NioEventLoop。

一个NioEventLoop拥有一个Thread线程，负责一个Java NIO Selector选择器的IO事件轮询。



#### 5.2 Bootstrap启动器类

Bootstrap类是Netty提供的一个便利的工厂类，可以通过它来完成Netty的客户端或服务器端的**Netty组件的组装，以及Netty程序的初始化**。（当然也可以手动组装）

在Netty中，有两个启动器类：

+ 客户端：Bootstrap；
+ 服务端：ServerBootstrap；

**父子通道**

操作系统底层的socket描述符分为两类：·

+ **连接监听类型**。连接监听类型的socket描述符，放在服务器端，它负责接收客户端的套接字连接；在服务器端，一个“连接监听类型”的socket描述符可以接受（Accept）成千上万的传输类的socket描述符。
+  **传输数据类型**。数据传输类的socket描述符负责传输数据。同一条TCP的Socket传输链路，在服务器和客户端，都分别会有一个与之相对应的数据传输类型的socket描述符。
+ Netty中NioServerSocketChannel，异步无阻塞，服务端监听通道，底层封装的是连接监听类型的socket描述符；
+ NioSocketChannel，异步非阻塞，TCP socket传输通道，底层封装的是数据传输类型的socket描述符。
+ 在Netty中，**将有接收关系的NioServerSocketChannel和NioSocketChannel，叫作父子通道**。其中，NioServerSocketChannel负责服务器连接监听和接收，也叫父通道（Parent Channel）。对应于每一个接收到的NioSocketChannel传输类通道，也叫子通道（Child Channel）。

**EventLoopGroup线程组**

Netty的EventLoopGroup线程组就是一个多线程版本的反应器。而其中的单个EventLoop线程对应于一个子反应器（SubReactor）。

EventLoopGroup的构造函数参数：指定内部的线程数。如果没有传入参数或者参数为0，那么EventLoopGroup内部的线程数为最大可用的CPU处理器数量的2倍。

**在服务器端，一般有两个独立的反应器，一个反应器负责新连接的监听和接受，另一个反应器负责IO事件处理。对应到Netty服务器程序中，则是设置两个EventLoopGroup线程组，一个EventLoopGroup负责新连接的监听和接受，一个EventLoopGroup负责IO事件处理。**

```java
public void runServer() {
        //0. 创建reactor 线程组
  			// boss就是负责监听和接收的父通道，worker是负责处理IO事件的子通道反应器
        EventLoopGroup bossLoopGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerLoopGroup = new NioEventLoopGroup();

        try {
            //1 设置reactor 线程组，也可以只配置1个线程组，这样监听可能会被更耗时的处理业务阻塞
            b.group(bossLoopGroup, workerLoopGroup);
            //2 设置nio类型的channel，一般不会在netty中设置BIO吧
            b.channel(NioServerSocketChannel.class);
            //3 设置监听端口
            b.localAddress(serverPort);
            //4 设置通道的参数：option()是父通道设置，childOption()是子通道设置
            b.option(ChannelOption.SO_KEEPALIVE, true);
            b.option(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT);
            b.childOption(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT);

            //5 装配子通道流水线
            b.childHandler(new ChannelInitializer<SocketChannel>() {
                //有连接到达时会创建一个channel
                protected void initChannel(SocketChannel ch) throws Exception {
                    // pipeline管理子通道channel中的Handler
                    // 向子channel流水线添加一个handler处理器
                    ch.pipeline().addLast(new NettyDiscardHandler());
                }
            });
// 为什么不装配父通道的流水线？父通道也就是NioServerSocketChannel连接接受通道，它的内部业务处理是固定的：接受新连接后，创建子通道，然后初始化子通道，所以不需要特别的配置。
            // 6 开始绑定server
            // 通过调用sync同步方法阻塞直到绑定成功
            ChannelFuture channelFuture = b.bind().sync();
            Logger.info(" 服务器启动成功，监听端口: " +
                    channelFuture.channel().localAddress());

            // 7 等待通道关闭的异步任务结束
            // 服务监听通道会一直等待通道关闭的异步任务结束
            ChannelFuture closeFuture = channelFuture.channel().closeFuture();
            closeFuture.sync();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 8 优雅关闭EventLoopGroup，
            // 释放掉所有资源包括创建的线程
            workerLoopGroup.shutdownGracefully();
            bossLoopGroup.shutdownGracefully();
        }

    }
```



**ChannelOption通道选项**

+ SO_RCVBUF、SO_SNDBUF：TCP参数，每个TCP socket在内核中都有一个发送缓冲区和接收缓冲区，这两个选项就是用来设置TCP连接的这两个缓冲区大小。TCP的全双工的工作模式以及TCP的滑动窗口便是依赖于这两个独立的缓冲区以及其填充状态。
+ TCP_NODELAY：TCP参数，表示立即发送数据。Netty默认为true，这时候禁用Nagle算法，从而最小化豹纹传输的延时。
+ SO_KEEPALIVE：TCP参数，表示底层TCP协议的心跳机制，true表示保持长连接。
+ SO_REUSEADDR：TCP参数，



#### 5.3 Channel通道

Netty中不直接使用Java NIO的Channel通道组件，对Channel通道组件进行了自己的封装。

在Netty中，对于每一种通信连接协议，Netty都实现了自己的通道。并且都有阻塞和非阻塞两个版本。

在Netty的NioSocketChannel**内部封装了一个Java NIO的SelectableChannel成员**。通过这个内部的Java NIO通道，Netty的NioSocketChannel通道上的IO操作，最终会落地到Java NIO的SelectableChannel底层通道。

> 这个selectableChannel在Java NIO中提到过，只有实现了SelectableChannel的通道才能注册到selector中

**通道类的主要成员和方法**

+ parent属性：表示通道的父通道，连接监听通道的parent是null；传输通道的parent是接收到该连接的监听通道；
+ pipeline属性：表示处理器的流水线，一个channel通道拥有一条流水线；
+ **ChannelFuture connect(SocketAddress address)**：客户端的传输通道连接远程服务器；
+ **ChannelFuture bind（SocketAddress address）**：服务器的监听通道绑定监听地址；
+ **ChannelFuture close()**：关闭通道连接；
+ **Channel read()**：从内部的JavaNIO Channel通道读取数据，然后启动内部的Pipeline流水线，开启数据读取的入站处理。此方法的返回通道自身用于链式调用。
+ **ChannelFuture write（Object o）**：启程出站流水处理，把处理后的最终数据写到底层Java NIO通道。此方法的返回值为出站处理的异步处理任务。
+ **Channel flush()**：将缓冲区中的数据立即写出到对端。



#### 5.4 Handler业务处理器

Netty的Handler处理器分为两大类：

+ 第一类是ChannelInboundHandler通道**入站处理器**；
+ 第二类是ChannelOutboundHandler通道**出站处理器**。

二者都继承了ChannelHandler处理器接口。

这两个业务处理接口都有各自的默认实现：ChannelInboundHandlerAdapter，通道入站处理适配器；ChanneloutBoundHandlerAdapter，叫作通道出站处理适配器。

这两个默认的通道处理适配器，分别实现了入站操作和出站操作的基本功能。如果要实现自己的业务处理器，不需要从零开始去实现处理器的接口，只需要**继承通道处理适配器**即可。

整个的IO处理操作环节包括：从通道读数据包、**数据包解码、业务处理、目标数据编码、把数据包写到通道**，然后由通道发送到对端。

**按照这种方向来分，前面数据包解码、业务处理两个环节——属于入站处理器的工作；后面目标数据编码、把数据包写到通道中两个环节——属于出站处理器的工作。**











#### 5.5 Pipeline流水线

通道和Handler处理器之间的关系：多对多。一个通道的IO事件被多个的Handler实例处理；一个Handler处理器实例也能绑定到很多的通道，处理多个通道的IO事件。

所以Netty中使用ChannelPipeline通道流水线，将绑定到一个通道的多个Handler处理器实例串在一起，形成一条流水线。**默认实现：双向链表**。

**Handler的处理是按照既定的次序**，而不是从前到后的次序呢？Netty是这样规定的：入站处理器Handler的执行次序，是从前到后；出站处理器Handler的执行次序，是从后到前。总之，IO事件在流水线上的执行次序，与IO事件的类型是有关系的。



#### 5.6 ByteBuf缓冲区





#### 5.7 Decoder和Encoder



### Redis

Remote Dictionary Server远程字典服务器的缩写--Redis。使用C语言开发，将数据保存在内存中，一些经常用并创建时间较长的内容，可以缓存到Redis中，而应用程序能以极快的速度读取这些内容。

Redis通过键值对的形式来存储数据，类似于Java中的Map映射。Redis的key，只能是String类型；Redis的value，可以是String、map、list、set、sortedset。

Redis的主要应用场景：缓存（数据查询、短连接、新闻内容、商品内容），分布式回话session，聊天室的在线好友列表，任务队列（秒杀、抢购、12306），应用排行榜，访问统计，数据过期处理。

Redis的优点：

+ 速度快： 不需要等待磁盘的IO，在内存之间进行的数据存储和查询，速度非常快。当然，缓存的数据总量不能太大，因为受到物理内存空间大小的限制。
+ 丰富的数据结构 ：除了string之外，还有list、hash、set、sortedset，一共五种类型。
+ 单线程：避免了线程切换和锁机制的性能消耗。
+ 可持久化： 支持RDB与AOF两种方式，将内存中的数据写入外部的物理存储设备。
+ 支持发布/订阅。
+ 支持Lua脚本。
+ 支持分布式锁：在分布式系统中，如果不同的节点需要访同到一个资源，往往需要通过互斥机制来防止彼此干扰，并且保证数据的一致性。在这种情况下，需要使用到分布式锁。分布式锁和Java的锁用于实现不同线程之间的同步访问，原理上是类似的。
+ 支持原子操作和事务Redis事务是一组命令的集合。一个事务中的命令要么都执行，要么都不执行。如果命令在运行期间出现错误，不会自动回滚。
+ 支持主-从（Master-Slave）复制与高可用（Redis Sentinel）集群（3.0版本以上）
+ 支持管道：Redis管道是指客户端可以将多个命令一次性发送到服务器，然后由服务器一次性返回所有结果。管道技术的优点是：在批量执行命令的应用场景中，可以大大减少网络传输的开销，提高性能。





### Zookeeper

用来协调分布式环境，实现了分布式环境的数据一致性。Zookeeper提供的功能都是分布式系统中非常底层必不可少的基本功能，如果开发者自己来实现这些功能并且达到高吞吐、低延迟同时还要保证一致性和可用性，非常困难。









