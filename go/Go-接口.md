### 接口

接口类型是对其他类型行为的抽象和概括，因为接口不会和特定的实现细节绑定在一起。Go语言与其他语言不同的是满足隐式实现，也就是说没有必要对给定具体类型定义所有满足的接口类型。简单的拥有一些必须的方法就足够了。

#### 接口约定

接口类型是一种抽象类型，不会暴露出它所代表的队形的内部值得结构和支持的基础操作的集合。也就是只知道它的方法能够干什么。

#### 接口定义

```go
// 接口定义
type Square interface {
	// 定义计算面积的方法
	Area() float64
}
```

#### 接口实现

```go

type Rectangle struct {
	l, h float64
}

func (r *Rectangle) SetLang(lang float64) {
	if lang < 0 {
		panic("数据必须大于0")
	}
	r.l = lang
}

func (r *Rectangle) SetHeight(height float64) {
	if height < 0 {
		panic("数据必须大于0")
	}
	r.h = height
}

// 实现接口
func (r *Rectangle) Area() float64 {
	return r.h * r.l
}

// 调用接口
func GetArea(square Square) float64 {
	return square.Area()
}

// 定义一个圆
type Round struct {
	r float64
}

func (round *Round) setR(r float64) {
	if r < 0 {
		panic("半径必须大于0")
	}
	round.r = r
}

func (round *Round) Area() float64 {
	return math.Pi * math.Pow(round.r, 2)
}

```



```go

func main() {
	// 创建一个长方形对象
	p := new(prase.Rectangle)
	fmt.Printf("%T\n", p)
	p.SetHeight(10)
	p.SetLang(20)
	area := prase.GetArea(p)
	println(area)

	// 创建一个圆对象
	round := new(prase.Round)
	fmt.Printf("%T\n", round)
	round.SetR(12)
	// 实现了Square接口 同样可以传参
	roundArea := prase.GetArea(round)
	println(roundArea)

}
```

```go
*prase.Rectangle
+2.000000e+002
*prase.Round
+4.523893e+002
```

只要实现了接口中的全部方法就认为是实现了该接口。无论接口是否在同一个包下，是先编译还是后编译。也就是说，Go理解是否实现了接口就看这个你实现的方法与接口有没有保持一致。如果接口中的方法都实现了就认为实现了这个接口就可以转化。

#### 接口组合

一个接口可以是多个接口的组合形成工程新的接口

例如 定义读和写接口

```go
type Reader interface {
	Reade() string
}

type Writer interface {
	Write()
}
```

定义读写组合的新接口

````go
type ReadWrite interface {
	Reader
	Writer
}

// 相当于
type ReadWrite interface {
	Reade() string
	Write()
}
````

因为ReadWrite接口实现了Reader和Writer的接口 因此可以实现

ReadWrite和Reader、ReadWrite和Writer转换、也就是类似多态的效果。

```go

// 定义读接口
type Reader interface {
	Reade() string
}

// 定义写接口
type Writer interface {
	Write(name string)
}

// 读写接口聚合
type ReadWrite interface {
	Reader
	Writer
}

// 定义一个结构体
type Robbot struct {
	Name string
}

func (r *Robbot) Reade() string {

	return r.Name
}

func (r *Robbot) Write(name string)  {
	r.Name = name
}



func main(){
    	// 可以进行类似多态的转换
	var rd prase.Writer=new(prase.Robbot)
	rd.Write("demo")
	// 判断是否转换回来
	rd2,ok := rd.(prase.ReadWrite);
	if ok {
		println(rd2.Reade())
	}
    
}
```

接口也可以作为值类型，使用变量进行接收。一个接口的零值的类型和值部分都是nil。

接口值可以使用==和!=进行比较 仅当都是nil或者动态类型相同且动态值类型也能进行\== 判断 而且相等。

如果两个接口值得动态类型相同但是动态类型的值不可比较  这样进行比较就会失败并且抛出panic。

#### 接口是动态类型

一个接口类型的变量varI中可以包含任意类型的值，如果需要检测它的动态类型 需要进性断言

```go
if v,ok:=varI.(T);ok{
    ...
}
```

如果转换合法 v是T类型的值 ok是true  否则v是T类型的零值 ok为false。

类型判断接口变量的类型可以使用type-switch进行检测。

```go
func main() {

	var a string = "qqq"
	testInterfaceValue(a)

}

func testInterfaceValue(a interface{}) {
	switch v := a.(type) {
	case int:
		fmt.Println("int", v)
	case string:
		fmt.Println("string", v)
	default:
		fmt.Println("unknown")
	}

```



