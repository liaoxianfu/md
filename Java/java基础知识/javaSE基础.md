# javaSE基础

## 面向对象和面向过程的优缺点

|          | 优点                                                         | 缺点                                       |
| -------- | ------------------------------------------------------------ | ------------------------------------------ |
| 面向过程 | 性能比面向对象高，因为类的实例，开销比较大，比较消耗资源。   | 没有面向对象容易开发维护容易服用，容易扩展 |
| 面向对象 | 易维护、易复用、易扩展，由于面向对象有封装、继承、多态的特性。可以设计出低耦合的系统，使得系统更加的灵活，易于维护。 | 性能比面向过程低                           |

## java语言的特点

简单易学，面向对象，平台无关、可靠性、安全性，支持多线程，支持网络编程并且方便，解释与编译并存。

JDK、JRE、JVM三者的区别和联系

|                              | 介绍                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| JRE（java run environment）  | java 运行环境。普通用户使用JRE来运行java程勋。开发者必须安装JDK来编译、调试程序。 |
| JDK （java development Kit） | java开发套件，供java开发者使用，除了完整的JRE（java runtime environment）java运行环境，还包含了其他供开发者使用的工具包。 |
| JVM(Java Virtual Machine)    | jvm将字节码转换成特定的机器码，JVM提供了内存管理/垃圾回收和安全机制。这种机制独立硬件和操作系统，这正是java程序可以一次编写多处执行的原因。 |

1. JDK 用于开发，JRE 用于运行java程序 ；
2. JDK 和 JRE 中都包含 JVM ；
3. JVM 是 java 编程语言的核心并且具有平台独立性。

## 什么是字节码、字节码的好处

### java中的编译器和解释器

java引入了虚拟机的概念，在机器和编译器之间加入了一层抽象的虚拟的机器。这个虚拟机在任何平台都提供了编译程序的一个公共的接口。

编译程序只需要面向虚拟机即可，编译后生成虚拟机能够理解的字节码，然后由解释器来将虚拟机中的代码翻译成本地机器能够理解的二进制机器码。这也是为啥java是编译和解释型的并存。

执行过程

java源代码 a.java->编译器->a.class(字节码)->jvm->jvm解释器->机器可以执行的二进制机器码->程序运行

### 采用字节码的好处

java通过采用字节码的方式，在一定程度上解决了传统的解释型程序语言运行慢的问题，同时保留了可移植的特性，所以java的运行比较搞笑，而且由于java不针对特定的机器，因此java无需重新编译便可以在多种不同的机器上运行。

## java和c++的区别

都是面向对象的语言，但是都支持封装、继承和多态。

Java不提供指针来访问内存，程序内存更加安全。

Java是单继承的而C++是多继承的，但是Java的接口可以多继承，Java也支持实现多个接口。

Java有自动内存管理机制，无需手动释放内存。

## 字符型常量和字符串常量的区别

形式上:字符串常量是单引号引起的一个支付，字符串常量是多个字符用双引号引起来的。

含义上：字符常量代表以一个常量值是一个ASCI码 支付字符串是一个引用地址。

占内存地址： 字符常量占2个直接，字符串常量根据含有的字符数量计算。

## 重载和重写的区别

**重载：** 发生在同一个类中，方法名必须相同，参数类型不同、个数不同、顺序不同，方法返回值和访问修饰符可以不同，发生在编译时。 　　

**重写：** 发生在父子类中，方法名、参数列表必须相同，返回值范围小于等于父类，抛出的异常范围小于等于父类，访问修饰符范围大于等于父类；如果父类方法访问修饰符为 private 则子类就不能重写该方法。

|      | 在同一个类 | 方法名   | 参数                             | 返回值   |
| ---- | ---------- | -------- | -------------------------------- | -------- |
| 重载 | 是         | 必须相同 | 参数类型不同、个数不同、顺序不同 | 可以不同 |
| 重写 | 否   | 必须相同 | 所有必须相同，抛出的异常范围必须小于等于父类，访问范围大于等于父类，父类的方法是private无法重写 | 相同 |



## 构造器Constructor是否可以重写OverWrite

在讲继承的时候我们就知道父类的私有属性和构造方法并不能被继承，所以 Constructor 也就不能被 override（重写）,但是可以 overload（重载）,所以你可以看到一个类中有多个构造函数的情况。

## java面向对象的三大特性

封装、继承、多态

**封装**

封装把一个对象的属性私有化，同时提供一些可以被外界访问的属性的方法，如果属性不想被外界访问，我们大可不必提供方法给外界访问。但是如果一个类没有提供给外界访问的方法，那么这个类也没有什么意义了。

**继承**

继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。通过使用继承我们能够非常方便地复用以前的代码。 

**多态（**polymorphism**）**

所谓的多态是指程序中定义的引用变量所指向法人具体类型和通过引用变量所指向的具体类型和通过引用变量发出的方法在调用在编程时并不确定程序运行期间才能确定，即一个引用变量到底会指向哪个类的实例对象，该引用变量发出的方法到底调用哪个类的实现方法，必须在程序运行的时候才能确定。

java中有两种形式可以实现多态：继承（多个子类对同一个方法重写）和接口（实现接口并覆盖接口中的同一个方法）。

String和StringBuffer、StringBuilder的区别是什么 String为什么是不可变的

**可变性**

简单来说，String类中使用final修饰字符数组。

```java
    @Stable
    private final byte[] value;
```

所以String对象是不可变的，而StringBuilder和StringBuffer都是继承AbstractStringBuilder

```java
public final class StringBuffer 
    extends AbstractStringBuilder
```

```java
public final class StringBuilder
    extends AbstractStringBuilder
```

```java
abstract class AbstractStringBuilder implements Appendable, CharSequence
```

**线程安全性能**

String中的对象是不可变的，也是可以理解成常量，线程安全。AbstractStringBuiler是StringBuilder和StringBuffer的公共父类，定义了一些字符串的基本操作。StringBuffer对方法加了同步锁或者对调用的方法增加了同步锁，所以是线程安全的，StringBuilder并没有对方法进行加通过不锁，所以是非线程安全的。

```java
public final class StringBuilder extends AbstractStringBuilder implements java.io.Serializable, Comparable<StringBuilder>, CharSequence
{
    @HotSpotIntrinsicCandidate
    public StringBuilder() {
        super(16);
    }
    @HotSpotIntrinsicCandidate
    public StringBuilder(int capacity) {
        super(capacity);
    }
    @Override
    public StringBuilder append(CharSequence s, int start, int end) {
        super.append(s, start, end);
        return this;
    }

    @Override
    public StringBuilder append(char[] str) {
        super.append(str);
        return this;
    }
    .....
}
```

> JDK的源码中，被@HotSpotIntrinsicCandidate标注的方法，在HotSpot中都有一套高效的实现，该高效实现基于CPU指令，运行时，HotSpot维护的高效实现会替代JDK的源码实现，从而获得更高的效率。

```java
public final class StringBuffer
    extends AbstractStringBuilder
    implements java.io.Serializable, Comparable<StringBuffer>, CharSequence
{
    @Override
    public synchronized int compareTo(StringBuffer another) {
        return super.compareTo(another);
    }

    @Override
    public synchronized int length() {
        return count;
    }

    @Override
    public synchronized int capacity() {
        return super.capacity();
    }


    @Override
    public synchronized void ensureCapacity(int minimumCapacity) {
        super.ensureCapacity(minimumCapacity);
    }

    @Override
    public synchronized void trimToSize() {
        super.trimToSize();
    }


    @Override
    public synchronized void setLength(int newLength) {
        toStringCache = null;
        super.setLength(newLength);
    }


    @Override
    public synchronized char charAt(int index) {
        return super.charAt(index);
    }

    @Override
    public synchronized int codePointAt(int index) {
        return super.codePointAt(index);
    }

    @Override
    public synchronized int codePointBefore(int index) {
        return super.codePointBefore(index);
    }
    .....
}
```

**性能**
每次对String的对象进行修改就会生成一个新的String对象，然后将指针指向的String对象。StringBuffer每次丢回对StringBuffer对象本身进行操作，而不是生成新的String Buffer对象。相同情况下使用StringBuiler相比使用StringBuffer仅能获得10%~15%左右的心里能提升，但是是线程不安全的。

对于三者使用的

1. 操作少量的数据 = String
2. 单线程操作字符串缓冲区下操作大量数据 = StringBuilder
3. 多线程操作字符串缓冲区下操作大量数据 = StringBuffer

```java
package java01;

public class StringAndStringBuffer {	
    public static void main(String[] args) {
        String str1 = "hello world";
        int str1HashCode = str1.hashCode();
        System.out.println(str1HashCode);
        str1+="123";
        int str2HashCode = str1.hashCode();
        System.out.println(str2HashCode);
        // str1只要更改就会创建新的对象
        System.out.println(str1HashCode==str2HashCode);

        StringBuffer strBuffer1 = new StringBuffer("1233");
        System.out.println(strBuffer1.toString());
        int hashCode2 = strBuffer1.hashCode();
        System.out.println(hashCode2);
        strBuffer1.append("233");
        int hashCode3 = strBuffer1.hashCode();
        System.out.println(strBuffer1.toString());
        System.out.println(hashCode3);
        // StringBuffer 修改是在原有的对象进行操作的。 并且是线程安全的
        System.out.println(hashCode2==hashCode3);

        StringBuilder stringBuilder1 = new StringBuilder("123");
        System.out.println(strBuffer1.toString());
        int hashCode4 = stringBuilder1.hashCode();
        System.out.println(hashCode4);
        strBuffer1.append("str");
        System.out.println(strBuffer1);
        int hashCode5 = stringBuilder1.hashCode();
        System.out.println(hashCode5);
        // StringBuilder  修改是在原有的对象进行操作的 但是是线程不安全的。
        System.out.println(hashCode4==hashCode5);
    }
}
```

**自动装箱和拆箱**
装箱： 将基本类型用对应的引用类型进行包装

拆箱：将包装类型转换成基本数据类型。

```java
int x = 5;
Integer integer = x; // 自动装箱
System.out.println(integer); 
int a = integer; // 自动拆箱
System.out.println(a); 
```

在一个静态方法里调用一个非静态的成员是非法的。

Java程序在执行子类构造方法之前，如果没有super()调用父类的特定的构造方法，就会调用父类没有参数的构造方法，如果父类中没有无参构造方法就会导致在编译的失败。解决的方法就是在父类中添加一个无参构造方法。

## 接口和抽象类的区别

接口的默认的方法是public，所有父类方法在接口中不能有实现，在Java8中开始方法可以有默认的实现。抽象类可以有非抽象的方法。

接口中的实例变量默认是final的，而抽象类则不一定

一个类可以实现多个接口，但是最多只能实现一个抽象类。

一个类实现一个接口就要实现这个接口中的所有方法，但是抽象类则不一定。

接口不能使用new进行实例化，但是可以声明，如果声明一个接口，就必须引用一个实现该接口的对象。从设计层来说，抽象类是对类的抽象，是一种模板设计，接口是行为的抽象，是一种行为的抽象。

## 成员变量和局部变量的区别有哪些？

1、从语法形式上，成员变量属于类的，而局部变量是在方法中定义的变量或者是方法的参数。成员变量可以使用public、private、static等进行修饰，而局部变量是不能进行修饰的但局部和全局变量都可以使用 final进行修饰。

2、从变量的内存存在位置上，我们可以知道，成员变量是对象的一部分，它随着对象的创建而存在，而局部存在栈内存。

3、在变量在内存生存的时间来看，成员变量是对象的一部分，它随着对象的创建而存在，而局部变量随着方法的调用而自动消失。

4、成员变量如果没有赋值，则会自动以类型的默认值赋值。（如果被final进行修饰就必须显式赋值）；局部变量则不会自动赋值。



## 构造方法有哪些特性

1. 名字与类名相同；
2. 没有返回值，但不能用void声明构造函数；
3. 生成类的对象时自动执行，无需调用

## 静态方法和实例方法有何不同

在外部调用静态方法的时候，可以使用类名.方法名可以使用对象名.方法名进行操作，但是实例对象只能使用对象名.方法名这种方法调用。也就是静态方法可以不用实例化对象进行调用。

静态方法在访问访问本类成员时，只允许访问静态成员，也就是静态变量和静态方法，不允许访问实例成员变量和成员方法。实例方法没有限制。

对象相等和指向他们引用相等，两者不同

## 对象的相等与指向他们的引用相等，两者有何不同？

对象相等比的是内存中存放的内容是否相等，而引用相等比较的是他们指向的内存地址是否相等。

## ==与equals

==：判断的是两个对象的地址是不是相等，也就是判断两个对象是不是同一个对象（基本数据类型判断的是值是否相等，引用数据类型比较的是地址）

equals(): 它的作用是判断两个对象是否相等：一般会有两种使用情况：

情况1：类没有覆盖equals()方法。则通过equals()比较该类的两个对象，等价于等“==”比较这两个对象。

情况2: 类覆盖了equals()方法。一般我们都覆盖equals()方法来两个对象的内容相等；若它们的内容相等，则返回true（认为这两个对象相等）。

**说明**：

String中的equals方法是否被重写过的，因为object的equals方法是比较对象的内容地址，而string的equals方法比较的是对象的值。

当创建String类型的对象时，虚拟机会在常量池中查询有没有已经存在的值和要创建值相同的对象，如果有就把它赋给当前引用。如果没有就在常量池中重新创建一个String对象。

## hashCode与equals

hashCode()介绍

hashCode的作用就是获取哈希码，它实际上是一个int型的整数，这个哈希码的作用是确定该对象在哈希表中的索引位置。hashCode()定义在JDK的Object.java中，这就意味着Java的任何类中都包含hashCode()函数。

散列表存储的是键值对，它的特点是能够根据键快速检索出对应值。这其中就利用到散列码。

为什么要有**hashCode**

当你把对象加入HashSet时，HashSet会先计算对象的hashCode值来判断对象加入的位置，同时也会与其他已经加入的对象的hashCode值比较，如果没有相符合的hashCode，HashSet就会假设对象没有重复出现，但是如果发现相同的Hashcode值，这时就会调用equals()方法来检查hashCode相等的对象是否真的相等。如果两个相等，HashSet就不会让其加入，操作成功，如果不同就会重新散列到其他的位置。这样就大大减少了equals的次数，大大提高了执行的效率。

如果两个对象相等，则hashcode一定是相同的。

两个对象相等，对两个对象分别调用equals方法都返回true

两个对象有相同的hashCode的值，它们也不一定是相等的

因此，equals方法被覆盖过，则hashCode方法也必须覆盖。

hashCode的默认行为是对堆上的对象产生独特值，如果没有重写hashCode，那么该class的两个对象无论如何都不会相等。





























































