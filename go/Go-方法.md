### 方法

#### 1、方法声明

在函数声明时，在名字前面加上一个变量 就变成了一个方法。这个附加的参数会讲该函数附加到这种类型上，即相当于定义了一个独占的方法。

下面通过一个两点之间计算欧式距离的方法 来比较普通的函数和方法之间的区别

```go
package main

import (
	"math"
)

type Point struct {
	x, y float64
}

// 计算欧式距离
func Distance(p1, p2 Point) (distance float64) {
	//dis = sqrt((x1-x2)^2+(y1-y2)^2)
	x := math.Pow(p1.x-p2.x, 2)
	y := math.Pow(p1.y-p2.y, 2)
	distance = math.Sqrt((x + y))
	return distance
}

// 相对比于普通函数前面增加了函数接受者的类型
// 可与普通函数重名
func (p Point) Distance(q Point) (distance float64) {
	x := math.Pow(p.x-q.x, 2)
	y := math.Pow(p.y-q.y, 2)
	distance = math.Sqrt(x + y)
	return distance
}

func main() {
	p1 := Point{x: 4, y: 0}
	p2 := Point{x: 0, y: 3}
	println(Distance(p1, p2)) // 5.000000e+000
	println(p1.Distance(p2))  // 5.000000e+000
}
```



上面于函数不同的是在函数名前面增加了方法接收器，在Go语言中不会向其他面向对象语言那样使用this或者self作为接收器而是可以自定义选择接收器的名字。命名建议为其使用类型的第一个字母。

`p1.Distance(p2)`称为方法选择器。

#### 2、基于指针对象的方法

当调用一个函数时会对每一个参数进行值拷贝，如果一个函数需要更新变量或者函数中的参数实在太大可以通过使用指针避免进行这种默认的拷贝。对应到我们这里更新接收器的对象方法，当这个接受者变量本身比较大时，我们就可以使用其指针而不是对象声明方法。

```go
// 给予指针对象的方法 传递普通参数
func (p *Point) Distance2(q Point) (distance float64) {
	x := math.Pow(p.x-q.x, 2)
	y := math.Pow(p.y-q.y, 2)
	distance = math.Sqrt(x + y)
	return distance
}

// 基于指针对象的方法 传递指针
func (p *Point) Distance3(q *Point) (distance float64) {
	x := math.Pow(p.x-q.x, 2)
	y := math.Pow(p.y-q.y, 2)
	distance = math.Sqrt(x + y)
	return distance
}
```

###### NIl也是合法的接收器类型

就像函数允许nil指针作为参数一样，方法理论上也可以用nil指针作为接收器尤其当nil对于对象来说是合法的零值时，例如map或者slice。

例如

```go
// 定义一个链表
type IntList struct{
    Value int
    Tail *IntList
}
//求链表和

func (list *IntList)Sum()int{
    if list==nil{ // 判断是否到链尾
        return 0
    }
    // 递归求和
    return list.Value+list.Tail.Sum()
}

```

#### 3、通过嵌入结构体来扩展类型

```go
type Point struct{
    X,Y float64
}
type Rec struct{
    Point
    Area float64
}
```

通过嵌入Point类型也可以直接访问Rec.X,Rec.Y字段 不需要再调用的是否指出Point

与传统的类的继承不同的是，这里的Point和Rec并不是继承的关系 Point也无法通过“多态”的方法获得Rec的Area属性。但是Point和Area存在“has a”的关系



#### 4、方法值和方法表达式

我们通过选择一个方法并且再通过一个表达式里执行，比如常见的p.Distance()形式，实际上可以分成两步执行也是可以的。p.Distance叫做选择其，选择器可以返回一个方法值  可以使用一个变量进行接受  执行这个变量调用传入参数即可

```go
type P struct {
	X, Y float32
}
func (p1 *P) distance(p2 P) float64 {
	// 欧式距离
	xDis := math.Pow(float64((p1.X - p2.X)), 2)
	yDis := math.Pow(float64(p1.Y-p2.Y), 2)
	return math.Sqrt(xDis + yDis)
}
func main() {
	p1 := P{
		X: 2,
		Y: 3,
	}
	p2 := P{
		X: 3,
		Y: 2,
	}
	// 直接将方法赋值给变量
	f := p1.distance
	// 正常调用
	println(f(p2))
}
```

### 封装

一个对象的变量或者方法如果调用方是不可见的话就一般定义为封装。封装有时候也被称为信息隐藏。Go使用首字母大小写来判断是否对外可见。大写可见，小写不可见。

基于名字的手段再语言中最小的封装单元是package而不是其他类型语言的类型。一个struct类型字段对于同一个包中的所有代码都可见，无论是再一个函数还是方法中。

封装优点：

1、调用方不能直接修改对象的变量值，只需要关注少量的变量即可。

2、隐藏实现细节

