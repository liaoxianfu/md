## 响应式编程

（1）、什么是响应式编程？

**简称RP（Reactive Programming）**

响应式编程是一种面向数据流和变化传播的编程范式。这意味着可以在编程语言中很方便地表达静态或动态的数据流，而相关的计算模型会自动将变化的值通过数据流进行传播。

例如，在命令式编程环境中，a:=b+c表示将表达式的结果赋给a，而之后改变b或c的值不会影响a。但在响应式编程中，a的值会随着b或c的更新而更新。

（2）Java8及其之前的版本

在Java8中提供了观察者模式，一个是`Observer`接口一个是`Obserable`。

```java
public class RxDemo01 extends Observable {
    public static void main(String[] args) {
        RxDemo01 rx = new RxDemo01();
        // 添加观察者
        rx.addObserver(new Observer() {
            @Override
            public void update(Observable o, Object arg) {
                System.out.println("发生了变化");
            }
        });

        // 使用lambda表达式
        rx.addObserver((o, arg) -> {
            System.out.println("收到观察者的通知，准备发生改变");
        });
        rx.setChanged(); // 数据变化
        rx.notifyObservers(); //通知
    }
}
```

（3）Java9以及之后的版本

使用`Flow`接口，里面有很多的方法对响应编程的方法。

### 1、Reactor

在Reactor中，数据流发布者（Publisher）由两个类Flux和Mono两个类表示，一个Flux表示包含0-N个元素 而Mono表示包含0-1个元素。 Flux和Mono包含三个数据信号，分别是元素值、错误信号和完成信号.

三种信号的特点

错误信号和完成信号都是终止信号，但是不能共存。

如果没有发送任何的元素值，而是直接发送错误信号或者完成信号，表示空数据流。



调用just或者其他方法只是声明了数据流，数据流并没有发出，只有订阅之后才能触发。

（7）操作符

对数据进行一道道的加工

map

![image-20210119155609151](D:\MarkDown\Java\spring框架\WebFlux.assets\image-20210119155609151.png)

flatMap

![image-20210119154305364](D:\MarkDown\Java\spring框架\img\image-20210119154305364.png)

依赖文件

```xml
<dependencies>
    <dependency>
        <groupId>io.projectreactor</groupId>
        <artifactId>reactor-core</artifactId>
        <version>3.3.13.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>io.projectreactor</groupId>
        <artifactId>reactor-test</artifactId>
        <version>3.3.13.RELEASE</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

```java
public void test01() {
    // 基础声明
    // flux可以声明多个数据
    Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5, 6);
    // Mono只可以有一个数据
    Mono<Integer> mono = Mono.just(1);
    // 从数组中声明
    Integer[] arr = {1, 2, 3, 4, 5, 6};
    Flux<Integer> arrayFlux = Flux.fromArray(arr);
    // 从list中声明
    List<Integer> list = Arrays.asList(arr);
    Flux<Integer> listFlux = Flux.fromIterable(list);
    // 从Stream中声明
    Stream<Integer> stream = list.stream();
    Flux<Integer> fromStream = Flux.fromStream(stream);

}
```


```java
  // 这个方法不会对事件链进行任何的消费性行为，特别是没有对错误的处理，所以通常推荐使用其他的重载方法
  // subscribe()
```

```java
// 订阅并消费flux中的所有元素。
 ArrayList<Integer> arrayList = new ArrayList<>();
 flux.subscribe(i -> {
   arrayList.add(i * i);
  });
System.out.println(arrayList.toString());
```

```java
   
// Map操作
@Test
    public void test04() {
        Mono<List<Integer>> listMono = Flux.just(1, 2, 3, 4, 5).map(i -> {
            if (i > 3) {
                return i * i;
            } else {
                return i;
            }
        }).filter(i -> i > 5).collectList();
        listMono.block().forEach(System.out::println);
    }

    @Test
    public void test05() {
        Flux.just("hello-world", "你好-世界").flatMap(s -> {
            String[] split = s.split("-");
            return Flux.fromArray(split);
        }).delayElements(Duration.ofMillis(1000))
                .collectList().block().forEach(System.out::println);
    }

```

SpringWebflux执行流程和核心API，默认容器使用的是Netty，Netty是高性能，NIO框架（异步非阻塞框架）。

多路复用的方式。

SpringWebflux执行过程和SpringMVC是相似的。

在SpringWebflux核心控制器是DispatcherHandler，实现了接口WebHandler。实现里面的handle方法

```java
public interface WebHandler {
	
	Mono<Void> handle(ServerWebExchange exchange);

}

```



```java
// DispatcherHandler实现

@Override
	public Mono<Void> handle(ServerWebExchange exchange) {
		if (this.handlerMappings == null) { 
			return createNotFoundError();
		}
		return Flux.fromIterable(this.handlerMappings)
				.concatMap(mapping -> mapping.getHandler(exchange))
				.next()
				.switchIfEmpty(createNotFoundError())
				.flatMap(handler -> invokeHandler(exchange, handler))
				.flatMap(result -> handleResult(exchange, result));
	}
```

（3）SpringWebFlux里面的DispatcherHandler负责请求的处理。
HandlerMapping 根据客户端的请求找到对应的方法。

HandlerAdapter 负责请求业务的处理

HandlerResultHandler 负责响应结果的处理。

（4）SpringWebfluc实现函数式编程的两个接口： RouterFunction（路由处理）和handlerFunction（处理函数）

5、SpringWebflux（基于注解编程模型）

默认情况下使用netty服务器。

创建springboot项目，引入wen-Flux支持

```xml
  <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.12</version>
        </dependency>
    </dependencies>
```

创建entity/Student

```java
@Data
@AllArgsConstructor
public class Student {
    private int id;
    private String studentName;
    private String gender;
}

```

创建service/StudentService

```java
@Service
public class StudentService {

    private static Map<Long, Student> studentMap = new HashMap<>(4);

    static {
        studentMap.put(1L, new Student(1, "student1", "男"));
        studentMap.put(2L, new Student(2, "student2", "女"));
        studentMap.put(3L, new Student(3, "student3", "男"));
    }

    public Flux<Student> queryAllStudent() {
        return Flux.fromIterable(studentMap.values());
    }

    public Mono<Student> queryStudentById(long id) {
        Student student = studentMap.get(id);
        return Mono.justOrEmpty(student)
                .switchIfEmpty(Mono.just(new Student(-1, "unkonw", "ukonwn")));
    }

    public Mono<Void> addStudent(Mono<Student> studentMono) {
        return studentMono.doOnNext(student -> {
            long id = studentMap.size() + 1;
            studentMap.put(id, student);
        }).thenEmpty(Mono.empty());
    }

}
```

创建Controller

```java
@RestController
public class StudentController {
    @Resource
    private StudentService studentService;

    @GetMapping("student/all")
    public Flux<Student> getAllStudents() {
        return studentService.queryAllStudent();
    }

    @GetMapping("student/{id}")
    public Mono<Student> getStudentById(@PathVariable("id") long id) {
        return studentService.queryStudentById(id);
    }
}
```













