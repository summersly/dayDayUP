### 数据结构与算法分析--java

----

#### java集合类对比分析

1. 底层数据结构
2. 增删改查方式
3. 初始容量，扩容方式，扩容时机
4. 线程安全与否
5. 是否允许空，是否允许重复，是否有序

**ArrayList**

初始容量：10，底层是object数组

增删慢：在每次添加新的元素时，ArrayList都会检查是否需要进行扩容操作（1.5倍），数据向新数组的重新拷贝；**解决办法**：在构造ArrayList时可以给ArrayList指定一个初始容量，这样就会减少扩容时数据的拷贝问题；当然在添加大量元素前，应用程序也可以使用ensureCapacity操作来增加ArrayList实例的容量，这可以减少递增式再分配的数量。

不同步：如果多个线程同时访问一个ArrayList实例。同步处理：

```java
List list = Collections.synchronizedList(new ArrayList(...)); 
```

