### 函数



#### 1、函数声明

函数声明包括 `函数名`、`形式参数列表`、`返回值列表`(可以省略)以及`函数体`

```go
func test(p1 type1,p2 type2...)(r1 type3,r2 type4...){
    // 具体逻辑
}
```

如果一个函数没有==形式参数==可以不用写，如果没有==返回值==也不用写 

返回值可以直接指定返回的变量

```go
func sum01(x, y int)(res int){
    res = x+y
   // return res // 可以返回变量
    return  // 也可以不指定返回变量
}
// 等价于
func sum02(x,y int)int{ // 如果只有一个返回值且不指定返回变量的名称可以不用写
    res:=x+y
    return res
}
```

如果一个函数包含了返回值列表就必须以return结尾，除非函数明显无法运行到结尾处，例如在结尾调用了panic或者函数无限循环。

函数传参是实参的拷贝，对形参的修改不会改变实参，但是如果形参是引用类型 例如指针、slice、map、function、channel等类型，实参可能由于函数简介引用被修改。

如果遇到没有函数体的函数声明，表示这个函数不是用GO实现的，这样声明定义了函数标识符。

```go
func Sin(x float64) float64 // 不是用Go实现的
```

#### 2、多值返回

Go语言支持多值返回，很多函数返回一个期望的返回值另一个是函数出错时的信息。语法见上。

#### 3、错误

对于大多数函数来说永远都不能保证一直运行成功。例如进行IO操作时会经常会面临出错的问题。通常导致失败的原因不止一种，尤其是对I/O操作来说，Go提供了一个内置的error接口。error的类型可能是 nil或者non-nil。nil意味着函数运行成功，对于non-nil的error类型，我们可以调用error的Error函数或者输出函数获得字符串类型的错误信息。

##### 3.1 错误处理策略

根据情况的不同，常用的处理方式有五种。

（1）、**传播错误**

传播错误意味着函数中的某个子程序运行失败会导致函数整体失败。这时候需要将该错误返回给调用者用来分析导致函数出错的具体原因。

（2）、**重试**

如果错误是偶然发生的，或者是由于不可知的问题导致的。当然在重试的时候需要限制重试的时间间隔或者重试的次数防止无限制的重试。

（3）、**输出错误信息并结束程序**

如果发生错误后程序无法运行，就需要直接结束程序运行并输出错误信息给用户，这种策略只应在main函数中执行。对于库函数而言应该向上传递错误，除非该错误意味着程序内部包含不一致性，也就是遇到了bug，才能在库函数中结束程序。

```go
os.Exit(1)
```

（4）直接输出错误信息

（5）忽略

##### 3.2 文件结尾错误

io包保证任何文件结束都会引发同一个错误--io.EOF。通过io.EOF来判断是否读取完成。



#### 4、函数值

在Go中函数被看做为第一类值，函数同其他值一样也拥有类型可以赋值给其他变量，传递给函数，从函数返回。

```go
func add(x, y int) int {
	return x + y
}

// 作为变量进行赋值
var addFunc = add

// 函数作为参数作为参数
func AddFun(add func(int, int) int, x, y int) int {
	return add(x, y)

}

// 返回函数
func mkSub() func(x, y int) int {
	
	return func(x, y int) int {
		return x - y
	}
}
```

函数类型的零值为nil 调用值为nil的函数会引起panic错误。

函数可以与nil进行比较，但是函数之间不能进行比较，也不能作为map的key。

#### 5、匿名函数

拥有函数名的函数只能在包级的语法块中被声明，通过函数字面量我们可以饶过这一限制，在任一和表达式中表示这一函数值。函数字面量的语法和函数声明相似，区别在于func关键字后面没有函数名。函数值字面量是一种表达式，它的值被称为匿名函数。

函数字面量允许我们在使用函数时再去定义它,通过这种技巧我们可以改写string.Map的调用。

##### 5.1、捕获迭代的变量

```go
func test21() {
	rmdirs := make([]string, 10)

	for _, dir := range demoDirs() {
		println(&dir) // 地址是相同的
		rmdirs = append(rmdirs, dir)
	}
}
```



#### 6、可变参数9

参数数量可变的参数被称为可变参数，典型的例子就是fmt.Println() 函数，可以接受任意多个参数。在声明可变参数时，需要在参数列表的最后一个参数类型前加上`...`表示该函数会接受任意数量该类型的参数。

```go
func sum(vals...int)int{
    total:=0
    for _,val:range vals{
        total+=val
    }
    return total
}
```

虽然在可变参数函数内部，....int型参数的行为看起来很像切片类型，但在实际上，可变参数函数和切片作为参数的函数时不同的。

```go
func f(vals...int){} // type func(...int)
func g(vals []int]){} // type func([]int)
```

如果参数是interface{} 类型表示可以接受任意类型。

#### 7、defer函数

在普通的函数或者回复前加上defer关键字就完成了defer的语法。当defer语句执行时，跟在defer后面的函数会延迟执行，直到包含该defer语句的函数执行完毕后defer后面的函数才会别执行，不论包含defer语句的函数时通过return正常结束还是由于panic异常导致的结束，可以在一个函数中执行多条defer语句，它们的==执行顺序和声明顺序相反==

defer语句通常用于处理成对的操作，例如 打开、关闭、连接、断开等。

**for循环中使用defer的注意事项**

在循环题中使用defer语句需要特别注意，因为只有函数执行完毕之后，这些被延迟的函数才会被执行。

下面的代码会导致系统化的文件描述符被耗尽，因为在所有文件都被处理之前没有文件会被关闭。

```go
for _,fliename range filenames{
    f,err:=os.Open(filename)
    if err!=nil{
        return err
    }
    defer f.Close() // 在循环处理完所有的文件之前不会调用
    ... 
    
}
```

解决方法：

将循环体中的defer语句移至另一个函数，每次循环时调用这个函数。

```go
for _,filename :=range filenames{
    if err:=doFile(filename);err!=nil{
        return err
    }
}

func doFile(filename string)error{
    f,err:=os.Open(filename)
    if err!=nil{
        return err
    }
    defer f.Close()
}
```



#### 8、Panic异常

Go的类型系统会在编译时捕获很多错误但是有些错误只能在运行时进行检查，比如数组访问越界、空指针异常等，这些运行时会引起panic异常。一般而言当发生panic异常发生时程序会中断运行并立即执行在该goroutine中被延迟的函数（defer机制）术后程序崩溃后并输出日志信息。

并不是所有的panic异常都来自运行时，直接调用内置的panic函数也会引起panic异常，panic函数接受任何只作为参数。当某些不应该发生的场景发生时应该调用panic进行中断。

虽然Go的panic机制类似于其他语言的异常但是panic使用场景有一些不同。由于panic会引起程序的崩溃，因此panic一把用于严重的程序错误。对于大部分的漏洞应该使用Go提供的错误机制而不是panic。

#### 9、recover捕获异常

一般来说不应该对panic异常进行处理，但有时需要在异常中进行恢复例如在web服务器遇到不可预料的严重问题时，在崩溃前应该将所有连接关闭，如果不做任何处理会使得客户端一直处理等待状态。

如果再defer函数中调用了内置函数recover并且定义该defer语句的函数发生了panic异常recover会使程序从panic中回复并返回并返回panic value。导致panic异常的函数不会继续运行但是能正常价返回。在未发生panic时调用recover，recover会返回nil。

不能不加区分的恢复所有panic异常。不应该回复其他包引起的panic，公共API应该将函数作的运行失败作为error进行返回而不是panic。同样的一夜不应该回复一个由其他人开发的函数引起的panic，比如调用这传入的回调函数因为你无法确保这样做是安全的。







