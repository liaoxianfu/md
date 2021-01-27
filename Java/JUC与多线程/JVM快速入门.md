### JVM快速入门

jvm的位置

jvm是运行在操作系统之上，与底层平台没有关系。

![image-20200409161024584](D:\MarkDown\Java\JUC与多线程\img\image-20200409161024584.png)

JVM 体系结构概览

 4 个问题

什么是类装载器ClassLoader？

有哪几种？

双亲委派机制？

沙箱安全机制以及为啥使用沙箱安全？



![image-20200409161224188](D:\MarkDown\Java\JUC与多线程\img\image-20200409161224188.png)

**类装载器ClassLoader**

负责加载class文件，class文件在文件开头由特定的文件标识。

**方法区不是放方法的区域**

方法区是放类信息的地方，也就是模板的地方。ClassLoader只负责class的文件加载，至于是否是可以运行，则由Execution Engine决定。

**java有几种加载器？**

•虚拟机自带的加载器

•启动类加载器（Bootstrap）C++

•扩展类加载器（Extension）Java

•应用程序类加载器（AppClassLoader）Java也叫系统类加载器，加载当前应用的classpath的所有

•用户自定义加载器 Java.lang.ClassLoader的子类，用户可以定制类的加载方式


```java
public class MyObject {
    public static void main(String[] args) {
        Object o = new Object();
        System.out.println(o.getClass().getClassLoader().getParent().getParent()); // NullpointException
        System.out.println(o.getClass().getClassLoader().getParent());// NullpointException
        System.out.println(o.getClass().getClassLoader()); //null
        


        MyObject myObject = new MyObject();
        System.out.println(myObject.getClass().getClassLoader().getParent().getParent()); // null
        System.out.println(myObject.getClass().getClassLoader().getParent()); // sun.misc.Launcher$ExtClassLoader@1b6d3586
        System.out.println(myObject.getClass().getClassLoader());// sun.misc.Launcher$AppClassLoader@18b4aac2

    }
}

```


![image-20200409162506693](D:\MarkDown\Java\JUC与多线程\img\image-20200409162506693.png)



**双亲委派机制和沙箱安全机制**

**双亲委派机制**

当一个类收到了类加载请求，他首先不会尝试自己去加载这个类，而是把这个请求委派给父类去完成，每一个层次类加载器都是如此，因此所有的加载请求都应该传送到启动类加载其中，只有当父类加载器反馈自己无法完成这个请求的时候（在它的加载路径下没有找到所需加载的Class），子类加载器才会尝试自己去加载。

采用双亲委派的一个好处是比如加载位于 rt.jar 包中的类 java.lang.Object，不管是哪个加载器加载这个类，最终都是委托给顶层的启动类加载器进行加载，这样就保证了使用不同的类加载器最终得到的都是同样一个 Object对象。 



“我爸是李刚，有事找我爹”，也就是从顶部开始往下找，找到就用，找不到就继续往下找，一直找不到就报ClassNotFoundException。

例如写了一个java.lang.String 类

```java
package java.lang;

public class String{
    public static void main(String[] args){
        System.out.printlin("hello")
    }    
}
// 在类java.lang.String中找不到main方法
```

从Bootstrap根加载器中区寻找，找到了String的类。在rt.jar中找到了String的类，出现了报错。

**native方法**

native方法调用的是底层系统的方法，将数据存放在本地方法栈,普通方法放在java栈里。

 本地接口的作用是融合不同的编程语言为 Java 所用，它的初衷是融合 C/C++程序，Java 诞生的时候是 C/C++横行的时候，要想立足，必须有调用 C/C++程序，于是就在内存中专门开辟了一块区域处理标记为native的代码，它的具体做法是 Native Method Stack中登记 native方法，在Execution Engine 执行时加载native libraies。

 目前该方法使用的越来越少了，除非是与硬件有关的应用，比如通过Java程序驱动打印机或者Java系统管理生产设备，在企业级应用中已经比较少见。因为现在的异构领域间的通信很发达，比如可以使用 Socket通信，也可以使用Web Service等等，不多做介绍。



**PC寄存器**  程序寄存器(基本不存在垃圾回收)

 每个线程都有一个程序计数器，是线程私有的,就是一个指针，指向方法区中的方法字节码（用来存储指向下一条指令的地址,也即将要执行的指令代码），由执行引擎读取下一条指令，是一个非常小的内存空间，几乎可以忽略不记。

这块内存区域很小，它是当前线程所执行的字节码的行号指示器，字节码解释器通过改变这个计数器的值来选取下一条需要执行的字节码指令。

如果执行的是一个Native方法，那这个计数器是空的。

用以完成分支、循环、跳转、异常处理、线程恢复等基础功能。不会发生内存溢出(OutOfMemory=OOM)错误。

Stack Overflow栈

8种数据类型+引用类型+实例方法

java方法=栈帧

栈帧保存输入输出参数和方法内的变量

栈操作 记录出栈和入栈的操作记录

递归引起java.lang.StackOverflowError sof错误 异常继承的是Throwable

 

堆（heap） 新生代、老年代、元空间

新生代： 伊甸园区、幸存者0区、幸存者1区

老年区： 养老区

元空间

OOM异常 堆内存异常





































