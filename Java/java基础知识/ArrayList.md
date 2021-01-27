# ArrayList介绍

## ArrayList简介

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

ArrayList的底层是数组队列，相当于动态数组。与 Java 中的数组相比，它的容量能动态增长。在添加大量元素前，应用程序可以使用`ensureCapacity`操作来增加 ArrayList 实例的容量。这可以减少递增式再分配的数量。它继承于 **AbstractList**，实现了 **List**, **RandomAccess**, **Cloneable**, **java.io.Serializable** 这些接口。

 ArrayList 继承了**AbstractList**，实现了**List**。它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能。

ArrayList 实现了**RandomAccess 接口**，即提供了随机访问功能。RandomAccess 是 Java 中用来被 List 实现，为 List 提供**快速访问功能**的。在 ArrayList 中，我们即可以通过元素的序号快速获取元素对象，这就是快速随机访问。

ArrayList 实现了**Cloneable 接口**，即覆盖了函数 clone()，**能被克隆**。

　　ArrayList 实现**java.io.Serializable 接口**，这意味着ArrayList**支持序列化**，**能通过序列化去传输**。

　　和 Vector 不同，**ArrayList 中的操作不是线程安全的**！所以，建议在单线程中才使用 ArrayList，而在多线程中可以选择 Vector 或者 CopyOnWriteArrayList。

> arraylist的默认初始大小是10 也可以自己指定数组的大小 

**联系：** 看两者源代码可以发现`copyOf()`内部调用了`System.arraycopy()`方法 **区别：**

1. arraycopy()需要目标数组，将原数组拷贝到你自己定义的数组里，而且可以选择拷贝的起点和长度以及放入新数组中的位置
2. copyOf()是系统自动在内部新建一个数组，并返回该数组。

## 源码阅读

```
 * Resizable-array implementation of the {@code List} interface.  Implements
 * all optional list operations, and permits all elements, including
 * {@code null}.  In addition to implementing the {@code List} interface,
 * this class provides methods to manipulate the size of the array that is
 * used internally to store the list.  (This class is roughly equivalent to
 * {@code Vector}, except that it is unsynchronized.)
```

可变数组 实现了List的所有方法 可以允许存放所有类型的元素包括NULL 
除了实现List接口 这个类提供了方法去操作数组的大小 可以实现数组的内部排序。（这个类大体上等于Vector 除了它是非线程安全的）


```
* <p>The {@code size}, {@code isEmpty}, {@code get}, {@code set},
 * {@code iterator}, and {@code listIterator} operations run in constant time.  The {@code add} operation runs in <i>amortized constant time</i>,
 * that is, adding n elements requires O(n) time.  All of the other operations run in linear time (roughly speaking).  The constant factor is low compared
 * to that for the {@code LinkedList} implementation.
```
//    list.size();
//    list.isEmpty();
//    list.get();
//    list.set(index, element);
//    list.iterator();
//    list.listIterator();
上述的操作的时间复杂度是O(1)的 add() 方法是根据添加的数据量而定O(1) 或者O(N) 总的来说 所有的方法的时间复杂度是线性的 实现的复杂度是低于LinkedList的

```
<p>Each {@code ArrayList} instance has capacity.  The capacity is the size of the array used to store the elements in the list.  It is always at least as large as the list size.  As elements are added to an ArrayList, its capacity grows automatically.  The details of the growth policy are not specified beyond the fact that adding an element has constant amortized time cost.

```

每一个ArrayList的实例对象都有容量，这个容量是这个数组被用来存储这个列表的元素的。它始终至少要与这个list的容量一样大。作为元素被加入进入一个ArrayList，ArrayList的容量是自动增加的。详细的增加策略是没有超出规定的大小那么增加一个元素的时间复杂度就是常数级。

```
 * <p>An application can increase the capacity of an {@code ArrayList} instance before adding a large number of elements using the {@code ensureCapacity}
 * operation.  This may reduce the amount of incremental reallocation.
```

一个应用可以增加一个实例的容量在添加很多数量的元素之前通过使用ensureCapacity 方法，这将会减少重新分配的次数。

如果你需要在ArrayList中添加许多元素,那么你通过使用ensureCapacity方法去指定底层开辟数组的大小，减少因为数量过大而重新分配数组容量的次数。


```
* <p><strong>Note that this implementation is not synchronized.</strong>
 * If multiple threads access an {@code ArrayList} instance concurrently,
 * and at least one of the threads modifies the list structurally, it
 * <i>must</i> be synchronized externally.  (A structural modification is
 * any operation that adds or deletes one or more elements, or explicitly
 * resizes the backing array; merely setting the value of an element is not
 * a structural modification.)  This is typically accomplished by
 * synchronizing on some object that naturally encapsulates the list.
```
注意，ArrayList是没有实现synchronized的。如果多线程同时访问一个ArrayList实例而且至少有一个线程在结构上修改它，它必须在外部加上synchronized.(一个结构上的修改指的是任何添加或者删除一个或者多个元素的操作，或者明确的重新设置list的大小，如果仅仅只是修改一个元素的值则不是结构上的修改)。这是典型的通过同步方法添加进list

```
 * If no such object exists, the list should be "wrapped" using the
 * {@link Collections#synchronizedList Collections.synchronizedList}
 * method.  This is best done at creation time, to prevent accidental
 * unsynchronized access to the list:<pre>
 *   List list = Collections.synchronizedList(new ArrayList(...));</pre>
```

如果没有这样的对象存在，这个list应该通过使用
Collections.synchronizedList方法包装
包装 

```java
public static void test01() {
        ArrayList<String> list = new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            new Thread(() -> {
                // list.add(UUID.randomUUID().toString().substring(0, 8));
                for (int j = 0; j < 30; j++) {
                    list.add(UUID.randomUUID().toString().substring(0, 8));
                }
                System.out.println(list);
            }, String.valueOf(i)).start();
        }
    }
```

报`java.util.ConcurrentModificationException`异常

```java
// 线程安全 使用Vector
    public static void test04(){
        List<String> list = new Vector<>();
        for (int i = 0; i < 3; i++) {
            new Thread(() -> {
                for (int j = 0; j < 50; j++) {
                    list.add(UUID.randomUUID().toString().substring(0, 8));
                }
                System.out.println(list);
            }, String.valueOf(i)).start();
        }
    }

```





```java
// 线程安全01 使用Collection
    public static void test02() {
        List<String> list = Collections.synchronizedList(new ArrayList<String>());

        for (int i = 0; i < 3; i++) {
            new Thread(()->{
                for (int j = 0; j < 50; j++) {
                    list.add(UUID.randomUUID().toString().substring(0,8));
                }
                System.out.println(list);
            },String.valueOf(i)).start();
        }
    }

```

```java

//线程安全使用CopyOnWriteArrayList();
    public static void test03() {
        List<String> list = new CopyOnWriteArrayList<>();
        for (int i = 0; i < 3; i++) {
            new Thread(() -> {
                for (int j = 0; j < 50; j++) {
                    list.add(UUID.randomUUID().toString().substring(0, 8));
                }
                System.out.println(list);
            }, String.valueOf(i)).start();
        }
    }

```

```
 * <p id="fail-fast">
 * The iterators returned by this class's {@link #iterator() iterator} and
 * {@link #listIterator(int) listIterator} methods are <em>fail-fast</em>:
 * if the list is structurally modified at any time after the iterator is
 * created, in any way except through the iterator's own
 * {@link ListIterator#remove() remove} or
 * {@link ListIterator#add(Object) add} methods, the iterator will throw a
 * {@link ConcurrentModificationException}.  Thus, in the face of
 * concurrent modification, the iterator fails quickly and cleanly, rather
 * than risking arbitrary, non-deterministic behavior at an undetermined
 * time in the future.
```
这个迭代器返回通过他的类的迭代方法(iterator和listIterator)是具有快速失败机制的，如果这个list在迭代器创建之后存在结构上修改的在任何时间就会抛出异常 抛出（CurrentModificationException） 这个迭代器失败非常快速和干净。但是这种机制是不能被保证的，通常来说也就是不是同步的方法很难被保证，因此快速失败机制只能被用来检测bug。




```java
 public boolean add(E e) {
        // 开始的size = 0
        ensureCapacityInternal(size + 1);  // Increments modCount!! 跳转进入下面的方法
        elementData[size++] = e;
        return true;
    }

/********* ensureCapacityInternal(size + 1) start**********************/
// minCapacity = 1 == size+1
 private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

// 判断list的最小长度是多少
  private static int calculateCapacity(Object[] elementData, int minCapacity) {
      // 判断是不是第一次增加值，如果是 就判断自己制定的和默认最小值10 那个大 返回大的
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        // 其他情况下返回最小值
        return minCapacity;
    }
// modCount
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
        // 如果容器能够放下数据就添加数据 扩容数组
            grow(minCapacity);
    }



    private void grow(int minCapacity) {
        // 获取原有数组的长度
        int oldCapacity = elementData.length;
        // 扩容原有长度的一半 oldCapacity >> 1
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        // 如果新的容量仍然比允许的最小值小 数组长度就设置为最小值
        // 也就是说明无需进行扩容操作
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        // 如果扩容的长度大于MAX_ARRAY_SIZE 就调用hugeCapacity进行扩容
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        // 利用Arrays.copyOf()方法扩容数组
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
/************ ensureCapacityInternal(size + 1) end***************************/

elementData[size++] = e;
进行元素赋值
返回true

//初始容量大小为10
 private static final int DEFAULT_CAPACITY = 10;

// ArrayList 最大的值 会调用hugeCapacity() 方法的大小为
Integer.MAX_VALUE - 8
// 如果超过了这个数值 溢出就抛出OutOfMemoryError 否者就设置为最大的数据为
//Integer.MAX_VALUE
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

```
