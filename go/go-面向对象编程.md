[TOC]



## 面向对象编程

### 1、类型系统

类型系统是指一个语言的类型体系结构。一个典型的类型系统通常包括如下的基本内容。

基础类型 int、float等

复合类型 数组、结构体、指针等

可以指向任意对象的类型（Any类型）

值语义和引用语义

面向对象

接口。

在Java中存在两套类型系统，分别是值类型和对象类型，值类型包byte、int等值类型，对象类型是继承自Object的对象类型。这些类型可以定义成员变量和成员方法。而对于Go语言来说大多数都是值类型并且都可以包含对应的操作方法。在需要的时候你可以给任何类型新增方法（包括内置类型）。实现某个接口时无需从该接口继承，只需要实现该接口中的所有方法就被认为实现了该接口。任何类型都可以被Any类型引用，Any类型就是空接口`interface{}`

#### 1.1 为类型添加方法

```go
type Integer int
// 为Integer类增加一个Less方法
func(a Integer)Less(b Integer)bool{
    return a<b
}

func main(){
    // 可以直接声明
    var a Integer = 1
    res := a.Less(2)
    fmt.Println(res) // true
}
```

在Java和C++中隐藏了this指针。

```java
class Person{
    public String name;
    public int age;

    public void say(){
        // 直接使用了name和age 实际上是this.name和this.age
        System.out.println("name:"+ name +" age:"+age);
    }
    public static void main(String[] args) {
        Person p = new Person();
        p.age = 10;
        p.name = "w";
        p.say();
    }
}
```

在Python中并没有省略，而是使用了self固定的关键字,它和this的指针作用是完全一样的。

```python
class Person():
    def __init__(self,name,age):
        self.name = name
        self.age = int(age)
    
    def say(self):
        print(str.format("name:%s  age:%d"%(self.name,self.age)))
    
    
if __name__=='__main__':
    p = Person('w',10)
    p.say()
```

而在Go中没有隐藏this指针而且传递的不需要非要是指针，也不用非要叫this或者其他特定的名称。

```go
package main

import "fmt"

type Person struct {
	Name string
	Age  int
}

func (p Person) Say() {
	fmt.Printf("name:%s  age:%d\n", p.Name, p.Age)
}

func main() {
	p1 := Person{
		Name: "w",
		Age:  10,
	}
	p1.Say()
}
```

Go语言中面向对象最为直观，也无需支付额外的成本，如果要求对象必须以指针传递这时候会有一个额外的成本，因为如果对象很小的时候使用指针传递并不划算。只有需要修改某个对象的时候才必须使用指针，因为Go语言是值传递的，想要修改传递的值必须传递指针。

特别注意 **使用接口编程时，如果实现接口的是指针，那么不能把结构体实例值 赋值给接口。**

例如

```go
// 定义一个接口
type IRouter interface {
	// handle之前的处理
	PreHandle(request IRequest)
	// 业务处理
	Handle(request IRequest)
	// handle 之后的处理
	PostHandle(request IRequest)
}
// 实现接口
type BaseRouter struct {
}

func (b *BaseRouter) PreHandle(request ziface.IRequest) {

}

func (b *BaseRouter) Handle(request ziface.IRequest) {

}

func (b *BaseRouter) PostHandle(request ziface.IRequest) {

}

func main(){
    var ir IRouter
   
    var ir ziface.IRouter
	ir = BaseRouter{} // 报错 cannot use BaseRouter literal (type BaseRouter) as type ziface.IRouter in assignment:
    //	BaseRouter does not implement ziface.IRouter (Handle method has pointer receiver)

	// 必须取地址 不会报错
    ir = &BaseRouter{}
    
    
}

```







#### 1.2 值语义和引用语义

值引用对变量的修改只会对自身有效，而引用语义则会修改对应引用的所有变量。例如

```go

func main() {
	p1 := Person{
		Name: "w",
		Age:  10,
	}
	p2 := p1
	p2.Age = 20
	p2.Say() // name:w  age:20
	p1.Say() // name:w  age:10
	fmt.Println("----------------------------")
	p3 := &p1
	p3.Name = "l"
	p3.Say() // name:l  age:10
	p2.Say() // name:w  age:20
	p1.Say() // name:l  age:10
}
```

在Go语言中类型的值语义被表现的非常纯粹，在C语言中函数接受一个数组的时候是被认为是引用类型的，但是在结构体中定义的时候又被认为是值类型的，但是在Go语言中数组就是单纯的值语义的。其中有几个特例分别是：数组切片，map，channel和接口。

#### 1.3 结构体相关操作

在Go语言中结构体与其他的面向对象的语言的类class是一个概念，但是放弃了很多属性例如继承，只是保留了组合这个基本的属性。其实严格上来说，组合并不能算是面向对象的特性。在C语言这种面向过程的语言中依旧存在。

结构体的定义

```go
package main

//Animal 结构体.
type Animal struct {
	name string
	age  int
}

```

结构体可以直接进行组合达到继承的目的

```go
// Dog 结构体 组合Animal结构体中的属性
type Dog struct {
	Animal
	sound string
}
```

结构体的初始化操作

```go
dog1 := new(Dog) // 默认都是对应的基本属性的零值
	fmt.Println(dog1.age, dog1.name, dog1.sound)
	dog2 := &Dog{} // 同样
	_ = dog2

	dog3 := &Dog{
		Animal: Animal{
			name: "w",
			age:  10,
		},
		sound: "sound",
	}
	fmt.Println(dog3.name)

```

构造函数

```go
// NewDog 构造函数
func NewDog(name string, age int, sound string) *Dog {
	return &Dog{
		Animal{
			name: name,
			age:  age,
		},
		sound,
	}
}
```

```go
// Student 结构体
type Student struct {
	// Person的组合
	Person
	Score  float32
	Gender byte
}

// Say function 继承Person的Say方法并添加新的功能
func (s *Student) Say() {
	gender := ""
	s.Person.Say()
	if s.Gender == 0 {
		gender = "男"
	} else {
		gender = "女"
	}
	fmt.Printf("Gender：%s，Score；%f", gender, s.Score)
}

```

### 2、接口

#### 2.1 接口赋值

接口赋值在Go中分为两种情况：一种是将对象实例赋值给接口，另一种是将另一个接口赋值给该接口。

**将对象赋值给接口**

只要该对象实现了该接口中的所有方法就可以将这个对象赋值给该接口。

a、定义接口

```go
// LessAdder 定义复合接口
type LessAdder interface {
	Less(b Integer) bool
	Add(b Integer)
}
```

b、对象实现这个接口中的所有方法

```go
type Integer int

// Less 判断小于的方法.
func (a Integer) Less(b Integer) bool {
	return a < b
}

// Add 方法。
func (a *Integer) Add(b Integer) {
	*a += b
}
```

现在有个问题是，如果我们有一个`Integer`对象实例，我们是传递对象还是对象指针呢？

```go
var a Integer = 1
var b LessAdder = a // 错误
var c LessAdder = &a // 正确
```

为什么呢？

因为我们知道如果直接进行值拷贝，那么对于`Add`方法的实现就不能达到预期。对于Less方法可以生成`func (a *Integer) Less(b Integer) bool`方法，这个方法与预期的原方法是一样的。如果方法的接受者都是进行值拷贝，那么上面的两个均可以成功赋值。

**将接口赋值给接口**

在Go语言中只要两个接口拥有相同的方法列表（次序不同不影响），那么它们就是等同的，可以进行相互赋值。

例如我们定义了下面的两个接口



```go
package one

// ReadWriter 接口
type ReadWriter interface {
	Read(buf []byte) (n int, err error)
	Write(buf []byte) (n int, err error)
}

```



```go
package two

type IStraem interface {
	Write(bs []byte) (n int, err error)
    Read(bs []byte) (n int, err error)
}

```

这两个接口在不同的包中且定义方法的次序不同，但是定义的方法列表相同。在Go语言中这两个接口并没有本质的区别。任何实现了one.ReadWriter就均实现了two.IStraem 两者的任何对象均可以相互赋值。当然接口的赋值中的方法并不要求**一一对应** 满足被赋值的接口中方法是赋值接口中方法的子集即可。

```go
type Writer interface {
	Write(buf []byte) (n int, err error)
}
```

例如Writer接口，只要实现了one.ReadWriter或two.IStraem均可以给Writer接口进行赋值。但是反过来是不可以的。



**接口查询**

如果想要让上面的Writer接口赋值给two.IStraem怎么办呢？就需要使用接口查询，使用接口查询查询Writer接口是否实现了two.IStream中的方法。

```go
	var file1 one.ReadWriter = new(os.File)
	var file2 one.Writer = file1
	if file3,ok :=file2.(two.IStraem);ok{
		_=file3
		fmt.Println("可以进行接口转换")
	}
```

在Go语言中可以询问接口能否转为任何的类型。

```go
if t1,ok:=a.(type);ok{
    // do something
}
```

当然还可以进行更加直接了当的查询接口指向的对象实例的类型。

```go
var v1 interface{} ="你好"
	switch v:=v1.(type){
	case int:
		fmt.Printf("%d",v)
	case string:
		fmt.Printf("%s\n",v)
	default:
		fmt.Println("unkonw")
	}
```

**接口组合**

接口与结构体一样都可以进行组合。

