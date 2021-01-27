## Go语言基础（一）



### 1、变量声明

Go语言声明变量分为四种：全局变量声明、局部变量声明、匿名变量声明和init初始化函数。

#### 1.1、全局变量声明

声明格式：**var 变量名称 变量类型** 如果在声明的同时给变量赋值可以省略变量类型。

```go
var a int // 声明一个为赋值的变量 默认值为int的零值 也就是0 
var b = false // 声明的同时进行赋值 编译器会进行自动推导 这里赋值为false后 推导类型为bool
```

多个变量可以只写一个var

```go
var(
	a int
    b bool
    str string
)
```

多个类型的可以一行声明

```go
var a,b,c int
```

可以同时在一行中进行声明和赋值

```go
var a,b,c = 1,2,3
```

也可以指定为不同的类型 自动推导

```go
var a,b,c  = 1,true,"asd"
```



#### 1.2 局部变量

局部变量声明除了与**上面的全局变量声明的格式**以外，还可以使用`:=`进行声明 注意：**这种只可以用在函数体内声明**

```go
a:=1 // 与var a =1 等同
```

也可以进行多值一行赋值

```go
a,b:=1,2
```

也可以进行互换值而无需引入第三个变量

```go
a,b=b,a
```



#### 1.3 匿名变量

由于局部变量声明后必须使用，否则编译器会报错，为了解决这个问题，可以将不需要使用的局部变量赋值给匿名变量`_`

```go
func test01() (int, string, bool) {
	return 1, "hallo", true
}

func main() {
	a, _, _ = test01() // 使用匿名变量接收无需使用的其他返回值 避免报错
	fmt.Println(a)

}
```



#### 1.4 init初始化函数

init函数是一个特殊函数，执行优先级比main函数要高并且不能手动进行调用，每个源文件都有一个init函数，在main函数之前进行调用。

```go
func test01() (int, string, bool) {
	return 1, "hallo", true
}

func main() {
	a, _, _ = test01()
	fmt.Println(a)

}

func init() {
	fmt.Println("我比main函数优先级高")
}

```

运行结果：

> 我比main函数优先级高
> 1

### 2、数据类型

不同的数据类型占据的内存空间是不同的，了解不同的数据类型能够提高内存的利用率。

|      类型       |                           类型描述                           |
| :-------------: | :----------------------------------------------------------: |
|      bool       |      bool类型 值为true和false两个，占据内存空间为1字节       |
|    int/uint     | Go支持有符号和无符号两种类型，具体的大小取决于编译器的实现。<br />但是Go内置了确定字节数的int实现int8/uint8(byte),int16/uint16 int32(rune)/uint32，int64/uint64 |
| float32/float64 | Go语言是没有float类型的，长度为32,64 占据的内存空间<br />为4字节和8字节 |
|     string      |  字符串类型 所有的字节默认使用UTF-8，字符串是固定的不可变化  |
|    派生类型     | 切片（slice）、字典（map），通道（channel）、指针（pointer）等 |
|                 |                                                              |

整数（int）和浮点数（float）类型的操作 加减乘除等 不再进行赘述，与其他语言基本一致。

#### 2.1、字符串

##### 2.1.1 字符串是不可变常量

GO语言的字符串串是一个固定的值也就是进行初始化后不能再进行修改。

```go
func main() {
	str:="Hello World"
	//str[0] ='A' // 报错 无法进行修改
	for i := range str {
		fmt.Println(str[i]) // 打印的是ASCAL码值
	}
}
```



字符串不进行转义

```go
func main() {
	str:=`你好\n世界`
	fmt.Println(str)
}
```

> 你好\n世界

字符串修改

```go
func main() {
	str := `你好\n世界`
	str += "hello"
	fmt.Println(str)
	// 修改字符（例如汉字 因为汉字的值>127） 转换为rune
	str2 := []rune(str)
	// 打印对应的ascal值
	for i:=0 ; i < len(str2); i++ {
		fmt.Println(str2[i])
	}
	// 修改
	// 你 对应值为 20320
	// 修改为 好 值为 22909
	str2[0] = 22909
	fmt.Println(string(str2))
}
```



>20320
>22909
>92
>110
>19990
>30028
>104
>101
>108
>108
>111
>好好\n世界hello



##### **2.1.2、Stirngs包操作字符串**

1、判断是否包含前缀

```go
// HasPrefix tests whether the string s begins with prefix.
func HasPrefix(s, prefix string) bool {
	return len(s) >= len(prefix) && s[0:len(prefix)] == prefix
}
```

```go
str:="你好 世界 World"
hasPrefix := strings.HasPrefix(str, "你")	
```

2、判断是否包含后缀

```go
// HasSuffix tests whether the string s ends with suffix.
func HasSuffix(s, suffix string) bool {
	return len(s) >= len(suffix) && s[len(s)-len(suffix):] == suffix
}
```

```go
suffix := strings.HasSuffix(str,"ld")
fmt.Println(suffix)
```

3、判断是否包含子串（字符）

函数原型

```go
// Contains reports whether substr is within s.
// 判断子串是否
func Contains(s, substr string) bool {
	return Index(s, substr) >= 0
}

// ContainsAny reports whether any Unicode code points in chars are within s.
func ContainsAny(s, chars string) bool {
	return IndexAny(s, chars) >= 0
}

// ContainsRune reports whether the Unicode code point r is within s.
func ContainsRune(s string, r rune) bool {
	return IndexRune(s, r) >= 0
}
```



函数测试

```go
func main() {
	str := "你好 世界 World"
	hasStr := strings.Contains(str, "世界")
	fmt.Println(hasStr) // true 

	containsAny := strings.ContainsAny(str, "")
	fmt.Println(containsAny) // false

	containsRune := strings.ContainsRune(str, rune('世'))
	fmt.Println(containsRune) // true
}
```



4、索引

查询首个匹配数据

```go

// 返回第一个子串的位置 不存在返回-1
func Index(s, substr string) int {...}  


// example
s:="Hello world"
strings.Index(s, "world") // 返回子串第一个的位置 6
strings.Index(s,"www") // 不存在子串 返回 -1 
```



```go
// 查询子串中任一一个字符存在的位置
func IndexAny(s, chars string) int {...}

strings.IndexAny(s, "loooo") // 2 

```



```go
// 返回r在s的位置
func IndexRune(s string, r rune) int {...} 

strings.IndexRune(s, rune('w') //  6
```



查询最后匹配的数据

与前面的用法基本一致

```go
s := "Hello world" // 长度为 11
	println(strings.LastIndex(s, "world")) // 6
	println(strings.LastIndexAny(s, "looo")) // 9
	println(strings.LastIndexByte(s, byte('l'))) // 9
```



5、替换

Replace函数返回前n个old字符串被new字符串替代的一个string类型变量s的副本。如果old为空，默认会匹配开头，也就是将替换的new字符串加载开头 如果n<0 就不会限制被替换的副本数

```go
// 函数原型

// Replace returns a copy of the string s with the first n
// non-overlapping instances of old replaced by new.
// If old is empty, it matches at the beginning of the string
// and after each UTF-8 sequence, yielding up to k+1 replacements
// for a k-rune string.
// If n < 0, there is no limit on the number of replacements.
func Replace(s, old, new string, n int) string {...}

```

```go
s := "你好，世界，世界"
newWord := "地球"
oldWord := "世界"
replaceWord := strings.Replace(s, oldWord, newWord, 1) // 将世界替换成地球 匹配的次数为1 返回的结果为 你好，地球，世界


```



```go
func test06() {
	s := "你好，世界，世界"
	newWord := "地球"
	oldWord := "世界"
	replaceWord := strings.Replace(s, oldWord, newWord, -1) // n为-1 <0 替换所有的 
    // 你好，地球，地球
	println(replaceWord)
}
```

```go
func test07() {
	s := "你好，世界，世界"
	newWord := "地球"
	oldWord := ""
	replaceWord := strings.Replace(s, oldWord, newWord, 1) // oldWorld为空 默认将新的字符串加载开头 地球你好，世界，世界
	println(replaceWord)
} 
```



6、统计字符串出现的次数

```go
	fmt.Println(strings.Count("cheese", "e"))
	fmt.Println(strings.Count("five", "")) // before & after each rune 空字符串是统计的字符串数量加1
	// Output:
	// 3
	// 5
```

7、统计字符的数量

```go
s := "你好，世界，世界"
fmt.Printf("%s长度为%d\n",s,len([]rune(s))) // 将字符转换为int类型统计数组的长度 8
fmt.Println(utf8.RuneCountInString(s)) // 8
```



8、两种遍历方式

```go
	s:="Hello,世界"
	for i:=0;i<len(s);i++ {
		//fmt.Println(s[i])
		fmt.Printf("%c\n",s[i]) // 存在乱码 
	}

	for _, v := range s {
		//fmt.Println(v)
		fmt.Printf("%c\n",v) // 不存在乱码 
	}

```



9、大小写转换

```go
// 大小写转换
s := "hello world"
upperStr := strings.ToUpper(s)
println(upperStr)
lowerStr := strings.ToLower(upperStr)
println(lowerStr)
// Output
// HELLO WORLD
//hello world
```



10、字符串修剪

只可以替换开头和结尾 如果需要进行中间的修剪可以使用Replace函数

```go
s:="hello world"
trim := strings.Trim(s, "h")
println(trim) // ello,world


s="  he"
trim=strings.TrimSpace(s)// 修剪空格
```



11、字符串分割

```go
s:="a,b,c"
split := strings.Split(s, ",")
for _, s2 := range split {
	println(s2)
}
```

12、插入字符串

```go
s := "a b c"
fields := strings.Fields(s)
for _, field := range fields {
	println(field)
}
// a
// b
// c

strJoin := strings.Join(fields, ";") // 将为slice的字符串间隔添加字符
println(strJoin)
// a;b;c
```



2.1.3、strconv包

strconv包主要用于字符串与其他数据类型进行转换

###### 1、其他类型转字符串

int类型转换为各种进制的字符串

```go
formatInt10 := strconv.FormatInt(122, 10) // 转换为10进制的字符串
println(formatInt10) //122
formatInt8 := strconv.FormatInt(122, 8) // 转换为8进制的字符串
println(formatInt8) //172
formatInt16 := strconv.FormatInt(122, 16) // 转换为16进制的字符串
println(formatInt16) // 7a
formatInt2 := strconv.FormatInt(122, 2) // 转换为2进制
println(formatInt2) // 1111010
itoa := strconv.Itoa(122) //相当于int转换为10进制
println(itoa) // 122
```





###### 2、字符串转其他类型







