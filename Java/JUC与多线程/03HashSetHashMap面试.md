## HashSet与HashMap

HashSet**底层**是什么？

HashMap

key value 

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

HashSet**也是线程不安全的**。

![image-20191226150357090](img\image-20191226150357090.png)

ArrayList HashSet HashMap 都是线程不安全的 都会报并发修改异常

Hash Map的解决同样是concurrentHashMap

## 八锁问题：



![image-20191226152040654](img\image-20191226152040654.png)

```java
synchronized 锁的是对象
```

![image-20191226153759978](img\image-20191226153759978.png)