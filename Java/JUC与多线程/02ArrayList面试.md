**ArrayList底层是什么类型**

底层是Object数组

**数组大小的初始值是多少？**

最开始是空引用 当add添加数据后是10 HashMap是16

**底层的扩容**

10 ->扩容到15 扩容一半

拷贝方法：Arrays.copyOf()

第二次扩容是 15/2 = 7

 15+7 = 22

HashMap是原来的一倍

ArrayList**是线程安全还是不安全**？

不安全

ArrayList**线程不安全的实例**

 

1、故障现象

 ![image-20191226141830872](img\image-20191226141830872.png)

<img src="img\image-20191226141952210.png" alt="image-20191226141952210" style="zoom:50%;" />





![image-20191226142614016](img\image-20191226142614016.png)



2、导致原因 报错

java.util.ConcurrentModificationException

![image-20191226143841151](img\image-20191226143841151.png)

3、解决方法 

Vector 是线程安全的，但是不用 性能低

![image-20191226142744915](img\image-20191226142744915.png)

上面的都可以用 但是不建议使用

4、优化建议

CopyOnWriteList() // 写时复制  解决线程不安全的问题

```java
public class CopyOnWriteArrayList<E> 
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {}
```

![image-20191226145430150](img\image-20191226145430150.png)









