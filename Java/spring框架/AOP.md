
## Spring AOP 使用


### 1、java动态代理

java的动态代理包括两类，一类是jdk的动态代理，另一类是使用CJLIB进行代理。

首先介绍jdk的动态代理

jdk的动态代理必须实现接口，代理接口中的方法。例如 创建一个Animal接口创建eat方法。

```java
public interface Animal {
    void eat();
}
```

创建Dog类实现Animal方法

```java
public class Dog implements Animal {
    public void eat() {
        System.out.println("dog eat");
    }
}
```

使用JDk进行代理

```java
public class AnimalInvocationHandler implements InvocationHandler {
    private Object target;

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object result = null;
        System.out.println("调用之前的处理");
        result = method.invoke(target, args);
        System.out.println("执行之后的操作");
        return result;
    }

    public Object bind(Object target) {
        this.target = target;
        return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), this);
    }
}

```

被代理后所有的方法都会执行代理前后的操作

```java
public class JDKProxy {
    public static void main(String[] args) {
        Dog dog = new Dog();
        AnimalInvocationHandler invocationHandler = new AnimalInvocationHandler();
        Animal animal = (Animal) invocationHandler.bind(dog);
        animal.eat();
        animal.sound("wangwang");
    }
}
/*
运行结果
调用之前的处理
dog eat
执行之后的操作
调用之前的处理
wangwang
执行之后的操作
*/
```

使用aop进行切面编程。

引入pom文件

```xml
<build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <dependencies>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>

    </dependencies>

```

1、使用JDK动态代理

使用这种方式必须实现接口才能进行代理，因此实现`Fruit`接口，在接口中实现抽象方法`eat`

```java
public interface Fruit {
    void eat();
}
```

实现实现类`Apple`和`Banana`

```java
@Component
public class Apple implements Fruit {
    public void eat() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("吃苹果");
    }
}
@Component
public class Banana implements Fruit {
    @Override
    public void eat() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("吃香蕉");
    }
}

```

实现切面关注点逻辑

```java
@Component
@Aspect
public class FruitEatHandler {
    @Pointcut("execution(* com.liao.aop.Fruit.*(..))")
    public void eatFruit() {
    }

    private void logDate() {
        LocalDateTime now = LocalDateTime.now();
        int year = now.getYear();
        int month = now.getMonth().getValue();
        int day = now.getDayOfMonth();
        int hour = now.getHour();
        int minute = now.getMinute();
        int second = now.getSecond();
        System.out.println(String.format("log time: %d-%d-%d %d:%d:%d", year, month, day, hour, minute, second));
    }

    @Before("eatFruit()")
    public void beforeEatFruit() {
        System.out.print("start ");
        logDate();
    }

    @After("eatFruit()")
    public void afterEatFruit() {
        System.out.print("end ");
        logDate();
    }

}

```

配置applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--配置使用注解-->
    <context:component-scan base-package="com.liao.aop"/>
    <!--配置使用aop注解-->
    <aop:aspectj-autoproxy/>
</beans>
```

进行测试

```java
@ContextConfiguration(locations = "classpath:applicationContext.xml")
@RunWith(SpringJUnit4ClassRunner.class)
public class SpringAopTest {
    @Autowired
    ApplicationContext context;
    @Test
    public void test01(){
        Fruit apple = (Fruit) context.getBean("apple");
        apple.eat();
    }
}
/*
startlog time: 2021-1-17 17:51:19
吃苹果
endlog time: 2021-1-17 17:51:20
*/
```

### 2、使用CGlib进行动态代理

使用CGlib与jdk动态代理，在业务逻辑代码出基本不变，但是需要强制开启CGlib,只需要在配置文件中增加` <aop:aspectj-autoproxy proxy-target-class="true"/>`即可。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--配置使用注解-->
    <context:component-scan base-package="com.liao.aop"/>
    <!--配置使用aop注解-->
    <aop:aspectj-autoproxy/>
    <!--强制开启cglib动态代理-->
    <aop:aspectj-autoproxy proxy-target-class="true"/>
</beans>
```

进行测试

```java
    @Autowired
    Apple apple;
    @Autowired
    Banana banana;

    @Test
    public void test02() {
        apple.eat();
        System.out.println("休息一下");
        banana.eat();
    }
/*
startlog time: 2021-1-17 18:1:53
吃苹果
endlog time: 2021-1-17 18:1:54
休息一下
startlog time: 2021-1-17 18:1:54
吃香蕉
endlog time: 2021-1-17 18:1:55
*/
```

**使用cglib的好处**

使用cglib可以直接对类中的方法进行切面而jdk动态代理是不可以实现这样的功能的，如果将cglib动态代理关闭，那么上面的测试就会报错。

```java
严重: Caught exception while allowing TestExecutionListener [org.springframework.test.context.support.DependencyInjectionTestExecutionListener@3f3afe78] to prepare test instance [com.liao.aop.SpringAopTest@7c1e2a9e]
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'com.liao.aop.SpringAopTest': Unsatisfied dependency expressed through field 'apple'; nested exception is org.springframework.beans.factory.BeanNotOfRequiredTypeException: Bean named 'apple' is expected to be of type 'com.liao.aop.Apple' but was actually of type 'com.sun.proxy.$Proxy17'
....
```

### 3、spring aop的增强类型

spring aop按照增强在目标类方法中的连接点可以分为5种。分别是：

① 前置增强：表示在目标方法执行之前进行增强

②后置增强：表示目标在执行之后进行增强

③环绕增强：表示目标在执行前后进行增强

④异常抛出增强：表示再目标方法抛出异常进行增强

⑤ 引介增强：在目标中添加一些新的方法和属性



利用AspectJ相关注解进行AOP编程。AspectJ是一个面相切面编程的框架，可以生成遵循java字节码编程规范的class文件。Spring AOP与AspectJ之间的关系是：Spring使用了AspectJ一样的注解，并且使用AspectJ做切入点的匹配和解析。但是SpringAOP运行时并不依赖AspectJ的编译器或者织入器等特性。

对应的注解如下：

① 前置增强：@Before

②后置增强：@AfterReturning

③环绕增强：@Around

④异常抛出增强：@AfterThrowing

使用@Pointcut来表示切入点

在注解的使用的时候可以指定一个切入点，也可以直接指定表达式、

例如

1、被切入的对象

```java
@Component
public class Person {
    public void say(String info) {
        System.out.println(info);
    }

    public int add(int a, int b) {
        if ((a < 0)) {
            throw new RuntimeException("a 不能小于0");
        }
        return a + b;
    }
}

```



2、切入代码

```java
@Aspect
@Component
public class PersonAspect {
    //前置增强
    @Before(value = "execution(* com.liao.aop.Person.*(int,int ))")
    public void beforePointCut() {

    }

}

```

```java
@Aspect
@Component
public class PersonAspect {
    // 切入点
    @Pointcut("execution(* com.liao.aop.Person.*(..))")
    public void allPointCut() {
    }

    //前置增强
    @Before(value = "allPointCut()")
    public void beforePointCut() {

    }
}

```

这两个表达的效果都是一样的，只是如果写方法需要多个切面编程的话，使用切入点会更加的方便。

```java
 //前置增强
    @Before(value = "execution(* com.liao.aop.Person.*(int,int ))")
    public void beforePointCut(JoinPoint pointcut) {
        System.out.println("前置增强");
        // 获取参数 
        Object[] args = pointcut.getArgs(); 
        for (Object arg : args) {
            System.out.println("获取到的参数为" + arg);
        }

        Signature pointcutSignature = pointcut.getSignature();
        // 获取方法名 add
        String signature = pointcutSignature.getName();
        System.out.println(signature);
        // 获取类名 com.liao.aop.Person
        System.out.println(pointcutSignature.getDeclaringType().getName());
        // 获取短类型名 Person.add(..)
        System.out.println(pointcutSignature.toShortString());
        
    }
```



后置返回增强

```java
    @AfterReturning(value = "allPointCut()", returning = "res") // 指定返回值参数
    public void afterReturning(JoinPoint joinPoint, Object res) {
        System.out.println("后置返回增强");
        System.out.println("返回的结果为" + res);
    }

```

后置增强

```java
    @After(value = "allPointCut()")
    public void after() {
        System.out.println("后置增强");
        System.out.println("After");
    }

```

异常增强

```java
    @AfterThrowing(value = "allPointCut()", throwing = "e")
    public void afterThrowing(Exception e) {
        System.out.println("异常增强");
        System.out.println("异常" + e.getMessage());
    }
```



**注：后置增强比后置返回增强先执行**

很好理解，后置增强在函数还没有返回之前进行运行，后置返回增强是在返回之后增强。而且在异常增强之前执行。

测试代码



```java 
   @Autowired
    Person person;

    @Test
    public void test03() {
        int add = person.add(1, 2);

    }
    /* 
    返回的结果为：
    前置增强
    获取到的参数为1
    获取到的参数为2
    add
    com.liao.aop.Person
    Person.add(..)
    后置增强
    After
    后置返回增强
    返回的结果为3
     */

  @Test
    public void test04() {
        int add = person.add(-1, 2);

    }

/*
前置增强
获取到的参数为-1
获取到的参数为2
add
com.liao.aop.Person
Person.add(..)
后置增强
After
异常增强
异常a 不能小于0

java.lang.RuntimeException: a 不能小于0

	at com.liao.aop.Person.add(Person.java:19)
*/

```

环绕增强

```java
 @Around(value = "allPointCut()")
    public Object around(ProceedingJoinPoint pjp) {
        System.out.println("前置通知");
        Object o = null;
        // 获取参数
        Object[] args = pjp.getArgs();
        System.out.println(Arrays.asList(args));

        try {
            // 对数据进行处理
            o = pjp.proceed(args);
            int res = (int) o;
            if (res < 5) {
                o = 10;
            }
            System.out.println(o.getClass().getName());
            System.out.println("后置通知返回");
        } catch (Throwable throwable) {
            throwable.printStackTrace();
            System.out.println("后置通知异常");
        } finally {
            if (o != null) {
                System.out.println(o);
            }
            System.out.println("后置返回结束");
        }

        return o;
    }
```



4、execution表达式

```

```



