###  1、goroutine

goroutine是Go语言中轻量级线程的实现，由Go运行时(runtime)管理。使用非常简单，只需要在调用方法前面加上go关键字即可。

```go
func Add(x,y int){
    // .....
}

go Add(1,2) // 调用
```

在一个函数调用前面加上一个go关键字就会创建一个新的goroutine并发执行。函数返回时这个goroutine就自动结束了。值得注意的是，**如果这个函数有返回值时，这个返回值会被丢弃**。

goroutine具有以下的特点：

- go 的执行是非阻塞的，不会等待
- go 后面函数的返回值会被丢弃
- 调度器是无序调度的
- 所有的goroutine的权重都是一样的
- main函数会单独开辟一个goroutine 遇到其他go关键字会再创建goroutine
- Go没有暴露goroutine id给用户，所以不能在一个go routine里显式操作另一个goroutine，不过可以使用runtime包提供的一些函数访问和设置goroutine的相关信息

#### 常见API

```go
runtime.GOMAXPROCS(0) // 在新版go语言中无论填入什么数 都会返回cpu的核心数
runtime.GoExit() // 结束当前goroutine运行
runtime.Gosched() //放弃当前调度执行的机会
```



```go
// runtime.GoExit() 
func goHandler() {
	defer func() {
		fmt.Println("exit .....")
	}()
	fmt.Println("do something")
	runtime.Goexit() // 结束当前运行 不会再往下执行 但是会调用defer 且不会产生panic
	fmt.Println("next ...")
}

func GoExitDemo()  {
	go goHandler()
	time.Sleep(time.Second)
}

```





### 2、并发通信

在工程上常见的并发通信模型有两种分别是：共享数据和消息。

共享数据是指多个并发单元分贝保持对同一个数据的引用，实现对该数据的共享。被共享的数据可能有多种形式，比如内存数据块，磁盘文件、网络数据等。在实际工作中最常见的是内存。

使用共享内存的方式进行并发通信

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
)

var counter int = 0

func Count(lock *sync.Mutex) {
	lock.Lock()
	counter++
	fmt.Println(counter)
	lock.Unlock()
}

func main() {
	lock := &sync.Mutex{}
	for i := 0; i < 10; i++ {
		go Count(lock) // 并发执行
	}

	for {
		lock.Lock()
		c := counter
		lock.Unlock()
		runtime.Gosched() // 让出时间片
		if c >= 10 {
			break
		}
	}

}

```

通过上面的例子我们在10个goroutine中共享了counter，每个goroutine执行完毕后将counter的值增加1，每次对counter的操作都要使用锁将counter锁住，操作完成之后再进行释放。在主函数中不断进行检测counter的值是否到达了10，如果大于等于10就完成了所有的goroutine，主线程就可以退出了。但是这样做将会使代码变得异常臃肿且难以理解。因此Go语言提供了另一种通信模型，以消息机制而非共享内存作为通信的方式。

Go语言中提供的消息机制被成为channel。

### 3、channel

channel是Go语言在语言级别提供的goroutine之间的通信方式。可以使用channel在两个或者多个goroutine之间传递消息。channel是进程内的通信方式，因此通过channel传递对象的过程和调用函数时的传递行为比较一致，例如可以使用传递指针。如果需要跨进程通信，建议使用分布式系统的方式来解决。

```go
package main

import (
	"fmt"
	"time"
)

func Count2(ch chan int, i int) {
	time.Sleep(time.Second)
	ch <- i
	fmt.Println("counting")
}

func main() {
	chs := make([]chan int, 10)
	count := 10
	for i := 0; i < count; i++ {
		chs[i] = make(chan int)
		go Count2(chs[i], i)
	}
	sum := 0
	for _, ch := range chs {
		sum += <-ch
	}
	fmt.Println(sum)
}
```

#### 3.1 基本语法

一般channel的声明形式为

````go
var chanName chan Type
````

与一般变量声明不同的不同地方仅仅是在于类型之前增加了一个`chan`关键字。

定义一个chan可以直接使用make函数

```go
ch1 := make(chan int) // 不带缓存
ch2 := make(chan int,number int) // 带缓存
```

这样就初始化并声明了一个int类型的名为ch的channel。在channel中最常用的是写入和读取操作。

写入`ch<-value` 读取`value := <-ch`无论是读取还是写入都会导致程序阻塞，直到有其他的goroutine从这个channel中读取数据或者写入进去数据。无缓存的channel的len和cap都是0，有缓存的len代表还没有读取的元素数,cap代表整个通道的容量**。无缓存的通道既可以用于通信也可以用于两个goroutine通信。**有缓存的用于通信。

```go
// 使用chan 进行goroutine之间的通信 模拟初始化耗时的操作 比顺序操作减少1s

// 全局配置属性
var baseUrl string

// 用于判断是否
var baseUrlChan chan bool

// 用于判断初始化是否完成
var initChan chan bool

func initConfig() {
	time.Sleep(time.Second) // 模拟读取配置文件耗时
	baseUrl = "http://www.baidu.com"
	baseUrlChan <- true
}

func Config() {
	//做一些无需baseUrl的耗时
	time.Sleep(time.Second)
    // 阻塞等待initConfig()完成 加载上baseUrl
	<-baseUrlChan
	fmt.Println(baseUrl)
	time.Sleep(time.Second)
	initChan <- true
}

func AsyncChan() {
	baseUrlChan = make(chan bool)
	initChan=make(chan bool)
	go initConfig()
	go Config()
	res := <-initChan
	fmt.Println(res)
}
```



```sh
=== RUN   TestAsyncChan
--- PASS: TestAsyncChan (2.00s)
=== RUN   TestAsyncChan/test1
http://www.baidu.com
true
    --- PASS: TestAsyncChan/test1 (2.00s)
PASS
```

操作不同状态的chan会引发三种不同的行为

**panic**

- 向已经关闭的通道写入数据会引发panic 关闭后存在数据读取不会发生panic
- 最佳实践是由写入方关闭能够最大程度上避免向已经关闭的通道写入数据引发panic
- 重复关闭会导致panic

**阻塞**

- 向未初始化的通道写数据或者读数据会导致当前goroutine永久阻塞。
- 向缓存区已满的channel写入也会导致阻塞
- 通道没有数据会进行阻塞

**非阻塞**

- 读取以及关闭的通道不会引发阻塞，如果没有数据会返回对应类型的零值可以用`commma,ok:=<-ch`进行判断
- 向缓冲没有满的通道进行写入不会引起阻塞





#### 3.3 channel传递

需要注意的是channel本身就是也是一个原生类型与map之类的类型的地位是一样的，因此channel在定义之后也可以通过channel进行传递。

单项channel只能用于接受或发送数据。channel本身是即支持读取又支持写入的，但是为了限制过多的操作，因此我们在讲一个channel变量传递给函数时，可以通过将其指定为单向channel变量，从而限制该函数对这个channel的操作。

```go
var ch1 chan int // 定义一个即支持写入又支持读取的channel
var ch2 <-chan int // 只支持读取int型的channel
var ch3 chan<- int // 只支持写入的channel
```



#### 3.4 多核并行优化

在执行一些计算任务时，我们希望能够尽量的利用多核的特性尽量让任务并行化。

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

type vector []int64

var resChan chan int64

func Add(v vector) {
	var sum int64 = 0
	for _, t := range v {
		sum += t
	}
	resChan <- sum
}

func Summary() {
	totalCount := 1000000
	v := make([]int64, totalCount)
	var i int64
	for i = 0; i < int64(totalCount); i++ {
		v[i] = i
	}
	numCPU := runtime.NumCPU()
	resChan = make(chan int64, numCPU)
	proNum := totalCount / numCPU
	for j := 0; j < numCPU; j++ {
		go Add(v[j*proNum : (j+1)*proNum])
	}
	var sum int64 = 0
	for k := 0; k < numCPU; k++ {
		sum += <-resChan
	}
	//fmt.Println(sum)
	close(resChan)
}

func Summary2(){
	totalCount := 1000000
	v := make([]int64, totalCount)
	var i int64
	for i = 0; i < int64(totalCount); i++ {
		v[i] = i
	}
	var sum int64 = 0
	for _,t:=range v{
		sum+=t
	}
}


func main() {
	now1 := time.Now()
	for i := 0; i < 100000; i++ {
		Summary()
	}
	now2 := time.Now()
	fmt.Println(now2.Sub(now1).Milliseconds())

	now3 := time.Now()
	for i := 0; i < 100000; i++ {
		Summary2()
	}
	now4 := time.Now()
	fmt.Println(now4.Sub(now3).Milliseconds())
}

// 183770
// 216741

```



#### 3.5 同步锁

虽然Go语言设计者使用了channel进行并发通信，但是也仍然保留了传统的同步锁机制。在Go语言的包中提供了两种锁类型，分别是`sync.Mutex`和`sync.RWMutex`Mutex是最简单的一种锁类型，同时也比较暴力，当一个goroutine获得了Mutex时其他的goroutine就必须乖乖的等这个goroutine释放这个Mutex。而RWMutex相对友好一些，是经典的读写模型，在读锁占用的情况下会阻止写锁，但是不会阻止读锁，也就是在读锁的情况下只允许读不允许写，但是在写锁的情况下都不允许，也就是即不允许读也不允许写.对于这两种锁的Lock()和RLock()都必须对应ULock()和URLock() 。

全局唯一性操作

对于从全局的角度只需要运行一次的代码，比如全局初始化操作，Go语言提供了一个Once类型来保证全局唯一性操作。例如全局配置文件的加载

```go
package main

import (
	"fmt"
	"sync"
)

var a string
var once sync.Once
var c chan bool

func Setup() {
	fmt.Println("我被调用了")
	a = "hello world"
}

func doPint() {
	once.Do(Setup)
	println(a)
	c <- true
}

func main() {
	c = make(chan bool, 2)
	go doPint()
	go doPint()
	for i := 0; i < 2; i++ {
		<-c
	}
}
/*
我被调用了
hello world
hello world
*/
```

如果没有引入once的那么就会调用两次setUp函数，使用once操作就会只调用了一次。

### 4、WaitGroup

前面提到的goroutine和chan 一个用于并发一个用于通信，没有缓存的channel可以具有同步功能，除此之外`sync`包提供了多个goroutine同步的机制，主要是通过WaitGroup实现的。主要数据结构和操作如下

```go
type WaitGroup struct {
		noCopy noCopy
		state1 [3]uint32
}

// 添加等待信号
func (w *WaitGroup) Add(delta int)

// 释放等待信号
func (w *WaitGroup) Done()

// 等待
func (w *WaitGroup)Wait()
```

WaitGroup 用来等待多个goroutine完成。main 函数的goroutine 调用Add来设置goroutine数目，当一个goroutine结束之后调用Done() Wait用来等待所有的goroutine完成。



```go
// 创建WaitGroup
var wg sync.WaitGroup

func WaitGroupDemo(urls []string) {
	for _, url := range urls {
		// 添加一个需要等待的goroutine
		wg.Add(1)
		go func(url string) {
			// 释放
			defer wg.Done()
			resp, err := http.Get(url)
			if err != nil {
				fmt.Printf("request %s error error info: %s\n", url, err.Error())
				return
			}
			if resp.StatusCode==http.StatusOK{
				fmt.Printf("%s request success!\n",url)
			}else {
				fmt.Printf("%s request error status code is %d\n",url,resp.StatusCode)
			}
		}(url)
	}
	// 等待所有的goroutine执行完毕
	wg.Wait()
}

```



### 5 、select机制

```go
func test02() {
	ch1 := make(chan int,1)
	// ch2:=make(chan int)
	for {
		select {
		case ch1 <- 0:
		case ch1 <- 1:
		default:
			fmt.Println("error")
		}
		i := <-ch1
		fmt.Println(i)
		time.Sleep(time.Millisecond * 500)
	
	}

}
```

select的用法和switch语言十分相似，由select开始一个新的选择块，每个选择块用case进行描述。与switch不同的是select每个case语句中必须有一个IO操作。

缓冲机制

创建一个一个带有缓冲机制的channel

```go
c:=make(chan int,1024)
```

在调用make()时缓冲区的大小作为第二个参数传入即可，具有缓冲机制的channel在缓冲区没有写满之前即使没有读取方进行读取也是可以继续写入的，如何缓冲区满了就会阻塞。读取方式与非缓冲区的channel一样。

```go
c:=make(chan int,1024)

for i:= range c{
    // ....
}
```

超时机制

在前面的介绍中完全没有考虑到出错的情况，但是这个问题显然是无法忽略的，在并发编程中最需要处理的就是超时问题，也就是向channel中写入数据时发现channel以满，或者读取的时候channel为空。使用channel是要非常小心，例如

```go
i:=<-ch
```

如果出现错误导致ch中一直没有数据写入也就是永远没有人向ch中写入数据，那么这个goroutine就会永远阻塞在这里。Go中并没有提供超时的处理机制，但是我们可以利用select机制实现。

```go
func TimeOutChan() {
	timeOutChan := make(chan bool)
	resChan := make(chan int)
	// 开启一个goroutine进行超时chan操作
	go func() {
		time.Sleep(time.Second * 2)
		timeOutChan <- true
	}()
	select {
	case <-resChan:
		fmt.Println("读取数据")
	case <-timeOutChan:
		fmt.Println("超时了 进行操作")
		
	}
}
```

```go
=== RUN   TestTimeOutChan
--- PASS: TestTimeOutChan (2.00s)
=== RUN   TestTimeOutChan/test1
超时了 进行操作
    --- PASS: TestTimeOutChan/test1 (2.00s)
PASS
```



### 6 、扇入（Fan in）和扇出（Fan out）

所谓的扇入就是讲多路通道聚合到一条通道中中进行处理，Go中最简单的扇入就是使用select聚合多条通道服务。扇出就是将一条通道发散到多条通道。在Go语言中使用go关键字启动多个goroutine进行并发处理。当生产者很慢时，可以使用扇入进行聚合多个生产者满足消费者，当消费者很慢时间可以使用扇出机制来满足生产者的生产速率。

### 7 、通知退出机制

读取已关闭的通道不会阻塞也不会引起panic，而是立即返回对应通道存储类型的零值，关闭select监听的某个通道能够使select立即感受到这种通知，然后进行相应的处理，这就是所谓的通知退出机制。`context`就是利用这种机制，通知退出机制是学习`context`库的基础。

```go
func GenerateIntA(done chan struct{}) chan int {
	ch := make(chan int)
	// 启动了一个go程 主程会立即返回ch 启动的go程会一直循环运行等待done的关闭
	go func() {
	Label:
		for true {
			select {
			// 向通道写入随机数
			case ch <- rand.Int():
			case <-done:
				break Label
			}
		}
		// 收到通知后关闭ch通道
		close(ch)
	}()
	return ch
}

func ChanDemo() {
	done := make(chan struct{})
	ch := GenerateIntA(done)
	fmt.Println(<-ch)
	fmt.Println(<-ch)
	// 查询现在go程数
    fmt.Printf("Num Goroutine = %d \n", runtime.NumGoroutine())
	// 发送通知 让生产者停止生产
	close(done)
    // 可能有数据 也可能为0
	fmt.Println(<-ch)
    // 一定为 0 
	fmt.Println(<-ch)
	// 查询现在go程数 比之前的go程数据减少1个
	fmt.Printf("Num Goroutine = %d \n", runtime.NumGoroutine())
}
```



```go
// 测试代码
func TestChanDemo(t *testing.T) {
	tests := []struct {
		name string
	}{
		{"test1"},
        {"test2"},
        {"test3"},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			ChanDemo()
		})
	}
}
```



```sh
=== RUN   TestChanDemo
=== RUN   TestChanDemo/test1
5577006791947779410
8674665223082153551
Num Goroutine = 4 
6129484611666145821
0
Num Goroutine = 3 
=== RUN   TestChanDemo/test2
3916589616287113937
6334824724549167320
Num Goroutine = 4 
0
0
Num Goroutine = 3 
=== RUN   TestChanDemo/test3
1443635317331776148
894385949183117216
Num Goroutine = 4 
2775422040480279449
0
Num Goroutine = 3 
--- PASS: TestChanDemo (0.00s)
    --- PASS: TestChanDemo/test1 (0.00s)
    --- PASS: TestChanDemo/test2 (0.00s)
    --- PASS: TestChanDemo/test3 (0.00s)
PASS
```



### 8、并发范式

#### 8.1、生成器

在应用系统编程中，常见的应用场景就是调用一个统一的全局生成器服务，用于生成全局事务号、订单号、序列号等。Go对这种场景的支持非常简单。

1. 最简单带有缓冲生成器

```go
package goroutine

import "math/rand"

// GenerateIntB 最简单的生成带有缓存的生成器
func GenerateIntB() chan int {
	ch := make(chan int, 10)
	go func() {
		for true {
			ch <- rand.Int()
		}
	}()
	return ch
}

=== 测试代码 ====

package goroutine

import (
	"fmt"
	"testing"
)

func TestGenerateIntB(t *testing.T) {
	tests := []struct {
		name string

	}{
		{"test1",},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			// 获取通道
			ch := GenerateIntB()
			fmt.Println(<-ch)
			fmt.Println(<-ch)

		})
	}
}

```

```sh
=== RUN   TestGenerateIntB
--- PASS: TestGenerateIntB (0.00s)
=== RUN   TestGenerateIntB/test1
5577006791947779410
8674665223082153551
```



(2) 多个生成器

采用扇入（fan in）的方法进行随机选择一个进行读取结果。

```go
func GenerateIntC1() chan int {
	fmt.Println("use function GenerateIntC1")
	ch := make(chan int, 10)
	go func() {
		for {
			// 模拟耗时的操作
			time.Sleep(time.Second)
			randInt := rand.Int()
			fmt.Println("=== GenerateIntC1 随机产生数字", randInt)
			ch <- randInt
		}
	}()
	return ch
}

func GenerateIntC2() chan int {
	fmt.Println("use function GenerateIntC2")
	ch := make(chan int)
	go func() {
		for {
			// 模拟耗时操作
			time.Sleep(time.Second)
			randInt := rand.Int()
			fmt.Println("=== GenerateIntC2 随机产生数字", randInt)
			ch <- randInt
		}
	}()
	return ch
}

func GenerateIntC() chan int {
	ch := make(chan int)
	go func() {
		for {
			// 使用select fan in 随机增加读写源
			select {
			//  <-GenerateIntC1() 读取数据  ch<-  将读取的数据写入
			case ch <- <-GenerateIntC1():
			case ch <- <-GenerateIntC2():
			}
		}
	}()

```



#### 8.2 、管道

通道可以分为两个方向，一个是读，一个是写，假如一个函数的输入参数和输出参数是相同的chan类型性，则函数可以自己调用自己最终形成一个调用链。

```go
func PipChan(in chan int) chan int {
	out := make(chan int)
	go func() {
		// 获取数据
		for v := range in {
			// 数据+1再扔进管道
			out <- v + 1
		}
		// 必须在go程函数中关闭
		close(out)
	}()
	return out
}

func PipDemo() {
	in := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			in <- i
		}
		// 必须在go程中关闭
		close(in)
	}()

	out := PipChan(PipChan(PipChan(in)))
	for v := range out {
		fmt.Println(v)
	}

}
```



#### 8.3、固定worker的工作池

服务器编程中使用最多的就是通过线程池来提升服务的并发处理能力。在Go语言编程中一样可以轻松的构建固定数目的goroutine作为工作线程池。



```go
package goroutine

//目的: 利用goroutine分布求取求一个int类型的slice的和

// job 结构体用来存储一个goroutine需要存储的数据的slice切片 以及存放结果的chan
type job struct {
	dataList []int // 存放一部分的数据
	result   int   // 存放切片数据
}

// work 接口 抽象处理数据的方法
type work interface {
	work()
}

// 具体实现的方法
func (job *job) work() {
	sum := 0
	// 注意 数据返回的位置和值
	for _,d := range job.dataList {
		sum += d
	}
	job.result = sum
}

// InitJob 初始化job
func InitJob(data []int, dataLen int, jobChan chan job, splitCount int) {
	loopCount := dataLen / splitCount
	mod := dataLen % splitCount
	for i := 0; i < loopCount; i++ {
		d:=data[i*splitCount:(i+1)*splitCount]
		jobChan <- job{
			dataList: d,
		}
	}
	// 最后剩余的数据作为一个新的task进行处理
	if mod != 0 {
		jobChan <- job{
			dataList: data[loopCount*splitCount:],
		}
	}
	close(jobChan)
}

func DistributeJob(jobChan chan job, resChan chan job, done chan struct{}, workPool int) {
	for i := 0; i < workPool; i++ {
		// workPool处理
		go Process(jobChan, resChan, done)
	}
}

// Process 数据处理汇总结果到新的resChan
func Process(jobChan chan job, resChan chan job, done chan struct{}) {
	for job := range jobChan {
		job.work()
		resChan <- job
	}
	done <- struct{}{}
}

func CloseChannel(resChan chan job, done chan struct{}, workPool int) {
	// 阻塞等待所有的goroutine处理完毕
	for i := 0; i < workPool; i++ {
		<-done
	}
	// 关闭通道
	close(done)
	close(resChan)
}

// Result 结果处理
func Result(resChan chan job) (sum int) {
	for res := range resChan {
		sum += res.result
	}
	return sum
}

func JobDemo(count int)(sum int) {
	dataList := make([]int, count)

	for i := 0; i < count; i++ {
		dataList[i] = i + 1
	}

	workPool := 10
	jobChan := make(chan job, workPool)
	resChan := make(chan job, workPool)
	done := make(chan struct{}, workPool)

	// 可以开启一个go程 进行初始化 因为jobChan channel 进行阻塞
	go InitJob(dataList, count, jobChan, 100)
	DistributeJob(jobChan, resChan, done, 1)
	go CloseChannel(resChan, done, 1)
	sum = Result(resChan)
	return sum
}

```



进行测试

```go
package goroutine

import "testing"

func TestJobDemo(t *testing.T) {
	type args struct {
		count int
	}
	tests := []struct {
		name    string
		args    args
		wantSum int
	}{
		{"test1", struct{ count int }{count: 1000}, 500500},
		{"test2", struct{ count int }{count: 1010}, 510555},

	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			if gotSum := JobDemo(tt.args.count); gotSum != tt.wantSum {
				t.Errorf("JobDemo() = %v, want %v", gotSum, tt.wantSum)
			}
		})
	}
}


```

```sh
=== RUN   TestJobDemo
=== RUN   TestJobDemo/test1
=== RUN   TestJobDemo/test2
--- PASS: TestJobDemo (0.00s)
    --- PASS: TestJobDemo/test1 (0.00s)
    --- PASS: TestJobDemo/test2 (0.00s)
PASS

```


