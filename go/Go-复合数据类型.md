[TOC]

### 复合数据类型

复合数据类型包括：数组，slice、map和struct。

数组是同构元素组成的、结构体是异构元素组成的。数组和结构体具有固定的内存大小。slice和map是动态的。

#### 1、数组

数组是一个由固定大小的同一类型的元素组成的序列，一个数组可以由0-多个元素组成。与之对应的slice可以动态的增长和收缩序列。

##### 1.1、创建数组

`var 变量名[数组长度]数据类型` 在没有显式初始化的情况下每个元素都会初始化为元素对应的零值

```go
var a[3]int //定义一个长度为3的int类型的数组 默认初始化为0
```

##### 1.2、获取数组的长度

使用内置的`len`函数

```go
len(a) // 获取数组a的长度
```

##### 1.3 显式初始化

显示初始化对应位置的数据为显示初始化的值，没有进行显示初始化的为对应类型的零值。

```go
var a[3]int =[3]int{1,2,3} // 对应的a[0]=1 a[1]=2 a[2]=3
var b[3]int = [3]int{1,2} // 对应的b[0]=1 b[1]=2  b[2]= 0 b[2] 没有对应的显式初始化值 初始化为int的0值
```

##### 1.4、初始化推导长度

如果出现数组的长度使用`...`进行代替 则表示根据初始化值的个数作为数组的长度

```go
c:=[...]{1,2,3} // c的长度为3
```

##### 1.5 索引初始化

可以指定一个索引和对应值列表进行初始化

```GO
type Currency int

// 定义索引
const(
	USD Currency=iota // 美元
    EUR // 欧元
    GBP // 英镑
    RMB // 人名币
)

symbol:=[...]{USD:"$",EUR:"€",GBP:"￡",RMB:"￥"}
fmt.Println(symbol[RMB])// ￥

```

##### 1.6 指定初始化位置

初始化索引的位置顺序是无关紧要的，而且没有用到的索引可以省略。

```go
r:=[...]intP{99:-1} // r包含了100个元素最后一个元素初始化为-1 其他默认初始化为0
```

##### 1.7 数据比较

如果两个数组中元素一一相等那么这两个数组是相等的。可以使用`==`进行判断。

```go
a:=[2]int{1,2}
b:=[...]int{1,2}
c:=[2]int{1,3}
fmt.Println(a==b)//true
fmt.Println(a==c)//false
fmt.Println(b==c)//false
```

数组长度和类型都相同的元素是同一个类型，不同长度或者类型的数组不是同一个类型，不能进行比较。

```go
a:=[2]int{1,2}
b:=[2]float32{1.0,2.0}
c:=[3]int{1,2,3}
fmt.Println(a==b)//compile error
fmt.Println(a==c)// compile error
```

##### 1.8 数组作为参数

在Go语言中函数参数变量接收的是一个复制的副本，并不是原始调用的变量。因此对一个函数传递大数据是非常低效的，而且在函数中对数组的修改都是修改在复制的数组上不会对原始的数组产生任何影响。

如果想对原始数组进行修改，可以显式传递衣蛾数组指针，这样就能直接反馈给调用者。

```go
func zero(ptr *[32]int){
    for i:=range ptr{
        ptr[i] = 0
    }
}
```



#### 2、slice

Slice(切片)代表变长的序列，序列中每个元素都有相同的类型。一个slice类型一般写作[]T 其中T代表slice中元素的类型。slice与数组很像但是没有固定长度。

slice与数组之间有着紧密的联系，slice底层引用了一个数组对象。slice提供了访问数组子序列（包括全部）的功能。

组成:

指针、长度、容量。

指针指向第一个slice元素对应的底层数组的元素地址，但是不一定是底层数组的第一个元素。长度对应slice元素的个数，长度不能超过容量。容量一般是slice从看是位置到底层数组结尾的位置。内置的len和cap分别返回slice的长度和容量。

多个slice之间可以共享底层的数据并且引用的数组部分区间可能会重叠。例如对slice进行切片获取新的slice 底层的数据是共享的

```go
func test06() {
	months := [...]string{
		"",
		"1月",
		"2月",
		"3月",
		"4月",
		"5月",
		"6月",
		"7月",
		"8月",
		"9月",
		"10月",
		"11月",
		"12月",
	}
	q2 := months[4:7]
	summer := months[6:9]
	fmt.Println(&months[6])           // 0xc000040af0
	fmt.Println(&q2[2])               //0xc000040af0
	fmt.Println(&summer[0])           //0xc000040af0
	fmt.Println(&q2[2] == &summer[0]) // true

}
```

容量一般是slice**从看是位置到底层数组结尾的位置** 使用cap查看

```go
fmt.Println(cap(months)) // 13
fmt.Println(cap(summer)) // 7
fmt.Println(cap(q2)) // 9
```

months是分配的13个数组大小的空间 summer从6月开始 也就是从底层数组为6开始 len(底层数组)-6 = 13-6 = 7 q2同理。

如果一个切片的超出cap(s) 的上限【s为被切骗的slice】，就会导致一个panic异常



长度是切片中元素存在个数的数量。超出len(s)意味着扩展了slice，因为新的slice长度会变大【这个扩展的长度是在slice切片的基础上进行的 不能超过cap(s)的大小】

```
fmt.Println(len(months)) // 13
fmt.Println(len(summer)) // 3
fmt.Println(len(q2)) // 3
```

```go
s1 := summer[:5]
for i, i2 := range s1 {
	fmt.Println(i,"========",i2)
}

/*
0 ======== 6月
1 ======== 7月
2 ======== 8月
3 ======== 9月
4 ======== 10月
*/
```

结果比来的summer还要大 引用了原始的months里的数据，扩大了slice



```go
	s1 := summer[:10] // 大于cap(summer)=7
// panic: runtime error: slice bounds out of range [:10] with capacity 7
	for i, i2 := range s1 {
		fmt.Println(i,"========",i2)
	}

```

slice切片直接修改的是底层的共享数据

```go
	summer[0] = "我修改了"
	for i, i2 := range months {
		fmt.Println(i,"========",i2)
	}
/*
0 ========
1 ======== 1月
2 ======== 2月
3 ======== 3月
4 ======== 4月
5 ======== 5月
6 ======== 我修改了
7 ======== 7月
8 ======== 8月
9 ======== 9月
10 ======== 10月
11 ======== 11月
12 ======== 12月
*/
```

和数组不同的是slice之间不能进行比较，不过可以使用bytes.Equal韩家墅进行判断两个字节型的是否相等 其他类型必须展开进行逐个元素的比较。唯一合法的比较是nil

```go
if s==nil{...}
```

但是如果需要判断一个slice是否为空应该使用长度是否为0进行判断而不是用nil。



##### 2.1、make函数创建slice

内置的make函数可以创建一个指定元素类型、长度和容量的slice。容量部分可以省略，省略的情况下容量等于长度。

```go
make([]T,len)
make([]T,len,cap)
```

##### 2.2 append函数追加元素

```go
	s:=make([]int,3)
	s = append(s, 4)
	for i, i2 := range s {
		fmt.Println(i, "========", i2)
	}
/*
0 ======== 0
1 ======== 0
2 ======== 0
3 ======== 4
*/
```

追加元素会在之前的基础上进行扩容。

每次扩容时直接翻倍。



#### 3、map

在Go语言中一个map就是一个哈希表的引用，map类型可以写作map[K]V,map中所有的key都是相同的类型且必须支持==操作。V对应的value数据没有任何限制。

##### 3.1、创建map

* make创建map

````go
ages:=make(map[string]int)
````

* 字面量创建

```go
ages:=map[string]int{
    "alice": 31,
    "charlie": 34,
}
```

* 创建空map

```go
ages:=map[string]int{}
```

##### 3.2 访问

通过key进行访问

```go
fmt.Println(ages["alice"]) //31
```

##### 3.3 删除

使用内置的delete函数删除

```go
delete(ages,"alice") // 移除alice
```

上述的操作都是安全的，即使找不到元素会返回value类型对应的零值。

支持赋值语句

```go
ages["bob"]+=1
ages["bob"]++
```

但是map中的元素并不是一个变量，无法获取地址

```go
&ages["bob"] // compile error
```

禁止map元素取地址的原因是随着map的增大 map可能会重新分配空间，从而导致之前的地址无效。

##### 3.4 遍历

使用for range进行遍历

```go
for k,v range ages{
    ......
}
```

map的迭代顺序是不确定的。如果需要按照顺序进行遍历的话 可以先将key存入一个slice进行排序后再通过key获取value

map类型的零值是nil，也就是没有引用任何哈希表。

map在未进行初始化的时候不能进行插入操作，会引发panic错误

```go
var m map[string]int
fmt.Println(m == nil) // true
//m["a"]=1 // panic
ages:=map[string]int{}
fmt.Println(ages == nil) // false
ages["na"]=10
fmt.Println(ages)
```

##### 3.5 判断key是否存在

为了区分返回的值是结果为0 还是key不存在返回的0值，go会返回一个结果值和一个状态值,利用状态值进行判断是否存在值

```go
age,ok = ages["blob"]
if !ok{
    // key不存在
}
```

和slice一样map也不能直接进行==的比较，唯一例外的是和nil进行比较。如果要判断两个map是否相同 需要遍历进行实现。



#### 4、结构体

结构体是一种聚合的数据类型，是由0-N个任意类型的值聚合的实体，每个值称为结构体的成员。

##### 4.1、结构体声明

声明一个学生结构体 有姓名、年龄、成绩三个成员。

```go
type Student struct {
	StudentName string
	Age         int
	Score       int
}
var stu1 Student // 声明一个类型为Student的stu1变量
```

可以通过点操作符进行访问并进行赋值

```go
var stu1 Student
stu1.StudentName = "张三"
stu1.Age = 10
stu1.Score = 99
// 也可以声明时进行初始化赋值
stu := Student{"张三", 10, 100}
```

也可以对成员进行取地址然后通过指针进行访问

```go
sc := &stu.Score
fmt.Println(sc,"====",*sc)
// 0xc0000044d8 ==== 100

```

结构体成员的顺序不同的话就是不同的类型，例如交换Age和Score就是不同的类型了。

##### 4.2 成员可见性

如果一耳光结构体成员的名字是大写开头的就是可导出的，也就是可以用点符号`.`进行操作，否则就不能进行导出，一个结构体中可以同时包含导出和不可导出的成员。

结构体的成员也可以是一个结构体。但是一个命名为S的结构体不能包含S类型的成员，但是可以包含指针类型为`*S`的指针成员，这可以让我们创建递归的数据结构例如、树或者链表。

定义一个二叉树

```go
type tree struct{
	value int
	left,right *tree
}
```

##### 4.3 函数传参

由于Go语言是值传递，因此如果需要修改就必须传递指针。



##### 4.4 结构体比较

如果结构体的==全部成员==都是可以比较的，那么结构体也是可以比较的。



##### 4.5 匿名成员

```go
type Point struct {
	Radio int
}

type Wheel struct {
	Point // 匿名嵌入 直接点操作Point里面的成员
	name string
}
func main(){
    var w Wheel
	w.Radio=3 //直接操作Point内部的Radio成员 无需点Ponint
	w.name = "wheel"
}
```

#### 5、JSON

在GO语言中对JSON、XML、ASN.1等标准格式的编码和解码都有良好的支持。JSON对象可以和map与结构体进行转换。

##### 5.1、结构体转JSON

```go
type Movie struct {
	Title  string
	Year   int  `json:"released"`
	Color  bool `json:"color,omitempty"`
	Actors []string
}

var movies = []Movie{
	{Title: "Casablanca", Year: 1942, Color: false,
		Actors: []string{"Humphrey Bogart", "Ingrid Bergman"}},
	{Title: "Cool Hand Luke", Year: 1967, Color: true,
		Actors: []string{"Paul Newman"}},
	{Title: "Bullitt", Year: 1968, Color: true,
		Actors: []string{"Steve McQueen", "Jacqueline Bisset"}},
}
```

```go
//data, err := json.Marshal(movies)  // 不进行格式化
	data, err := json.MarshalIndent(movies,"","\t") // 进行格式化
	if err != nil {
		log.Fatalf("JSON marshaing failed %s", err)
	} else {
		fmt.Println(string(data))
	}
```

使用tag可以改变key的值

```json
[
        {
                "Title": "Casablanca",
                "released": 1942,
                "Actors": [
                        "Humphrey Bogart",
                        "Ingrid Bergman"
                ]
        },
        {
                "Title": "Cool Hand Luke",
                "released": 1967,
                "color": true,
                "Actors": [
                        "Paul Newman"
                ]
        },
        {
                "Title": "Bullitt",
                "released": 1968,
                "color": true,
                "Actors": [
                        "Steve McQueen",
                        "Jacqueline Bisset"
                ]
        }
]

```

omitempty选项表示当Go语言结构体成员为空或者零值时不产生JSON对象。

##### 5.2 JOSN转结构体

```go
	data:=`[
        {
                "Title": "Casablanca",
                "released": 1942,
                "Actors": [
                        "Humphrey Bogart",
                        "Ingrid Bergman"
                ]
        },
        {
                "Title": "Cool Hand Luke",
                "released": 1967,
                "color": true,
                "Actors": [
                        "Paul Newman"
                ]
        },
        {
                "Title": "Bullitt",
                "released": 1968,
                "color": true,
                "Actors": [
                        "Steve McQueen",
                        "Jacqueline Bisset"
                ]
        }
]
`
	err := json.Unmarshal([]byte(data), &movies)
	if err != nil {
		log.Fatalf("err %s",err)
	}else {
		fmt.Println(movies)
		fmt.Println(movies[0].Title)
	}
```



