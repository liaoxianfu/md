## 并发控制

对于在一些场景中需要在一个协程中等待另一个或者多个协程的信息，举例用户下单后，创建订单需要确保在通知商家确定有货，收取金额后再进行数据库订单写入的操作。此时就需要等待通知商家和收取金额两个go routine成功通知数据库写入。Go提供了三个方案分别是channel、WaitGroup和Context

### 1. channel

```go
package main

import (
	"fmt"
	"time"
)

func test01(num int, ch chan int) {
	res := 0
	for i := 1; i <= num; i++ {
		res += i
		time.Sleep(time.Millisecond * 10)
	}
	ch <- res
	fmt.Println("success")
}

func main() {
	goCount := 10
	var ch = make([]chan int, goCount)
	fmt.Println("开始")
	for i := 0; i < goCount; i++ {
		// channel 进行初始化
		ch[i] = make(chan int)
		go test01(10*i, ch[i])
	}
	for i := 0; i < goCount; i++ {
		fmt.Println(<-ch[i]) // 会阻塞
	}
	fmt.Println("end")
}

```

### 2. WaitGroup

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func test02(num int, wg *sync.WaitGroup) {
	res := 0
	for i := 1; i <= num; i++ {
		res += i
		time.Sleep(time.Millisecond * 10)
	}
	fmt.Println(res, "   success")
	wg.Done()
}

func main() {
	var wg sync.WaitGroup
	wg.Add(2)
	go test02(100, &wg)
	go test02(1000, &wg)
	wg.Wait()
}

```

### 3. Context

Context实际上只实现了接口，接口的定义

```go
type Context interface {
	// 返回一个time.Time类型的的deadline 和是否设置deadline的状态值 如果没有设置deadline 就会默认为设置一个初始的时间和ok==false	
	Deadline() (deadline time.Time, ok bool)
    // 返回一个只读的chan 需要在select-case语句中使用 case<-context.Done() 
    // 当Context关闭后 返回一个关闭的管道 仍然可读 如果goroutine收到关闭的信息但是Context还没来得及关闭 此时返回nil。
	Done() <-chan struct{}
    // 描述context关闭的原因 关闭的理由由context决定 不需要用户设置 当context关闭后返回关闭的原因 如果还没关闭就返回nil
	Err() error
	// 有一种context不是用于控制树状分布的goroutine而是用于在树状分布的goroutine之间传递信息。
    // value 就是作用此种状态的context 该返回用key值查询map中的value。
	Value(key interface{}) interface{}
}
```

#### 3.1 emptyCtx 定义

```go
type emptyCtx int

func (*emptyCtx) Deadline() (deadline time.Time, ok bool) {
	return
}

func (*emptyCtx) Done() <-chan struct{} {
	return nil
}

func (*emptyCtx) Err() error {
	return nil
}

func (*emptyCtx) Value(key interface{}) interface{} {
	return nil
}

func (e *emptyCtx) String() string {
	switch e {
	case background:
		return "context.Background"
	case todo:
		return "context.TODO"
	}
	return "unknown empty Context"
}


var (
	background = new(emptyCtx)
	todo       = new(emptyCtx)
)

// Background returns a non-nil, empty Context. It is never canceled, has no
// values, and has no deadline. It is typically used by the main function,
// initialization, and tests, and as the top-level Context for incoming
// requests.
func Background() Context {
	return background
}
```

context包中定义了一个公用的emptCtx全局变量，名为background 可以通过context.Background()方法进行获取。context包提供了四个方法创建不同类型的context，使用这四个方法时如果没有父context则需要传入background，将background作为父节点。

四个方法的定义如下：

```go
// 创建cancelCtx
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
	if parent == nil {
		panic("cannot create context from nil parent")
	}
	c := newCancelCtx(parent)
	propagateCancel(parent, &c)
	return &c, func() { c.cancel(true, Canceled) }
}

// 创建timeCtx 实现设定一个具体的截止日期
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) {
	if parent == nil {
		panic("cannot create context from nil parent")
	}
	if cur, ok := parent.Deadline(); ok && cur.Before(d) {
		// The current deadline is already sooner than the new one.
		return WithCancel(parent)
	}
	c := &timerCtx{
		cancelCtx: newCancelCtx(parent),
		deadline:  d,
	}
	propagateCancel(parent, c)
	dur := time.Until(d)
	if dur <= 0 {
		c.cancel(true, DeadlineExceeded) // deadline has already passed
		return c, func() { c.cancel(false, Canceled) }
	}
	c.mu.Lock()
	defer c.mu.Unlock()
	if c.err == nil {
		c.timer = time.AfterFunc(dur, func() {
			c.cancel(true, DeadlineExceeded)
		})
	}
	return c, func() { c.cancel(true, Canceled) }
}

// 创建timeCtx 实现超时的效果
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
	return WithDeadline(parent, time.Now().Add(timeout))
}


// 创建valueCtx 实现在协程上传递数据的效果
func WithValue(parent Context, key, val interface{}) Context {
	if parent == nil {
		panic("cannot create context from nil parent")
	}
	if key == nil {
		panic("nil key")
	}
	if !reflectlite.TypeOf(key).Comparable() {
		panic("key is not comparable")
	}
	return &valueCtx{parent, key, val}
}

```





#### 3.2 cancelCtx

```go
type cancelCtx struct {
	Context
	mu       sync.Mutex            // protects following fields
	done     chan struct{}         // created lazily, closed by first cancel call
	children map[canceler]struct{} // set to nil by the first cancel call
	err      error                 // set to non-nil by the first cancel call
}
```

children 中记录了由此context派生的所有的child 此context被cancel时会将其所有的child都cancel掉。cancelCtx和deadline和value无关 ，所以只需要实现Done和Err即可。

```go
func (c *cancelCtx) Done() <-chan struct{} {
	c.mu.Lock()
    // 有可能cancelCtx还没分配done就被cancel了所以要初始化一下
	if c.done == nil {
		c.done = make(chan struct{})
	}
	d := c.done
	c.mu.Unlock()
	return d
}
```

```go
func (c *cancelCtx) Err() error {
	c.mu.Lock()
	err := c.err
	c.mu.Unlock()
	return err
}
```

err默认值为nil 在context被cancel时指定一个error变量var Canceled=errors.New("context canceled")

cancel方法 关闭自己以及自己的后代

```go
func (c *cancelCtx) cancel(removeFromParent bool, err error) {
	if err == nil {
		panic("context: internal error: missing cancel error")
	}
	c.mu.Lock()
	if c.err != nil {
		c.mu.Unlock()
		return // already canceled
	}
	c.err = err
	if c.done == nil {
		c.done = closedchan // 还没来得及创建 再创建一个 在初始化的是否就关闭了
        /**
        var closedchan = make(chan struct{})
		func init() {
			close(closedchan)
		}
        */
	} else {
		close(c.done) // 关闭chan
	}
	for child := range c.children {
		// NOTE: acquiring the child's lock while holding parent's lock.
		child.cancel(false, err)
	}
	c.children = nil
	c.mu.Unlock()

	if removeFromParent { // 将自己从parent中删除
		removeChild(c.Context, c)
	}
}

```



```go
package main

import (
	"context"
	"fmt"
	"time"
)

func HandleRequest(ctx context.Context) {
	go WriteRedis(ctx)
	go WriteMysql(ctx)
	for {
		select {
		case <-ctx.Done():
			fmt.Println("HandleRequest Done")
			return
		default:
			fmt.Println("HandleRequest running")
			time.Sleep(time.Second * 2)
		}
	}
}

func WriteRedis(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println("WriteRedis Done")
			return
		default:
			fmt.Println("WriteRedis running")
			time.Sleep(time.Second * 2)
		}
	}
}

func WriteMysql(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println("WriteMysql Done")
			return
		default:
			fmt.Println("WriteMysql running")
			time.Sleep(time.Second * 2)
		}

	}
}

func main() {
	ctx, cancelFunc := context.WithCancel(context.Background())
	go HandleRequest(ctx)
	time.Sleep(time.Second*5)
	fmt.Println("时间到了..... ")
	cancelFunc()
	time.Sleep(time.Second*5)
}

```





```go
type timerCtx struct {
	cancelCtx
	timer *time.Timer // Under cancelCtx.mu.

	deadline time.Time
}
```

#### 3.3 timerCtx

timerCtx在cancelCtx基础上添加了定时器的功能用于标识自动cancel的最终时间，而timeer就是自动触发的定时器。由此衍生出WithDeadline()和WithTiemout()，实际上实现的原理一致语境不同而已。

* deadline 指定最后期限，比如2021：3：22 00:00:00 自动结束
* timeout 指定最长存活时间 比如context在30s后被cancel

```go
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
	return WithDeadline(parent, time.Now().Add(timeout))
}
```

实际上就timeout就是在调用WithDeadline。

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func HandleRequest(ctx context.Context) {
	go WriteRedis(ctx)
	go WriteMysql(ctx)
	for {
		select {
		case <-ctx.Done():
			fmt.Println("HandleRequest Done")
			return
		default:
			fmt.Println("HandleRequest running")
			time.Sleep(time.Second * 2)
		}
	}
}

func WriteRedis(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println("WriteRedis Done")
			return
		default:
			fmt.Println("WriteRedis running")
			time.Sleep(time.Second * 2)
		}
	}
}

func WriteMysql(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println("WriteMysql Done")
			return
		default:
			fmt.Println("WriteMysql running")
			time.Sleep(time.Second * 2)
		}

	}
}

func main() {
	//ctx, cancelFunc := context.WithCancel(context.Background())
	// 设置3s后自动取消
	ctx, cancelFunc := context.WithTimeout(context.Background(),time.Second*3)
	// 设置截止日期
	context.WithDeadline(context.Background(),time.Now().Add(time.Second*10))
	go HandleRequest(ctx)
	time.Sleep(time.Second*5)
	fmt.Println("时间到了..... ")
	cancelFunc()
	fmt.Println(ctx.Err())
	time.Sleep(time.Second*5)
}

```



#### 3.4 valueCtx 

valueCtx比较简单 就是可以传递数据

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func HandleRequest(ctx context.Context) {
	go WriteRedis(ctx)
	go WriteMysql(ctx)
	go GetParameters(ctx)
	for {
		select {
		case <-ctx.Done():
			fmt.Println("HandleRequest Done")
			return
		default:
			fmt.Println("HandleRequest running")
			time.Sleep(time.Second * 2)
		}
	}
}

func WriteRedis(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println("WriteRedis Done")
			return
		default:
			fmt.Println("WriteRedis running")
			time.Sleep(time.Second * 2)
		}
	}
}

func WriteMysql(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println("WriteMysql Done")
			return
		default:
			fmt.Println("WriteMysql running")
			time.Sleep(time.Second * 2)
		}

	}
}

func GetParameters(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println("GetParameters Done")
			return
		default:
			fmt.Println("get parameters",ctx.Value("data"))
			time.Sleep(time.Second * 2)
		}
	}
}

func main() {
	//ctx, cancelFunc := context.WithCancel(context.Background())
	// 设置3s后自动取消
	ctx, cancelFunc := context.WithTimeout(context.Background(), time.Second*3)
	//  因为valueCtx是无法自动结束的 所以需要设置一个支持cancel的父context
	ctx = context.WithValue(ctx, "data", 1)
	// 设置截止日期
	context.WithDeadline(context.Background(), time.Now().Add(time.Second*10))
	go HandleRequest(ctx)
	time.Sleep(time.Second * 5)
	fmt.Println("时间到了..... ")
	cancelFunc()
	fmt.Println(ctx.Err())
	time.Sleep(time.Second * 5)
}

```

注意 因为valueCtx是无法自动结束的 所以需要设置一个支持cancel的父context

### 4.Mutex

互斥锁是在并发编程中对共享资源进行访问的主要手段，对此Go语言提供了简单的Mutex，Mutex是一个结构体类型对外暴露Lock() 和Unlock() 两个方法，用于加锁和解锁。

#### 4.1 Mutex构成

Mutex的定义如下:

```go
// A Mutex is a mutual exclusion lock.
// The zero value for a Mutex is an unlocked mutex.
//
// A Mutex must not be copied after first use.
type Mutex struct {
	state int32
	sema  uint32
}
```

根据官方的定义我们知道 Mutex是一个互斥锁 零值是一个unlock的互斥锁，当第一次使用后禁止进行拷贝操作。`state`用来表示互斥锁的状态，比如是否被锁定等。`sema`表示信号量，协程阻塞等待该信号量，解锁的协程释放信号量从而唤醒等待信号量。

![img](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/20201218131518552.png)

* Locked  代表是否被锁定 0 没有 1 锁定
* Woken 表示是否有协程被唤醒，0 没有 1 已经唤醒正在加锁
* Starving 该Mutex是否处于饥饿状态 0 没有 1 表示饥饿状态 有协程阻塞超过了1 ms
* Waiter 表示阻塞等待的协程个数 协程解锁时根据这个值来判断是否需要释放信号量

协程之间的抢锁实际上是争夺给Locked赋值的权利，能给Locked域赋值1，就说明抢锁成功。如果没有抢到就阻塞等待`sema`信号量，等待再次解锁进行下次的争夺。

#### 4.2 加锁和解锁过程



* 简单加锁

只有一个协程进行加锁

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/20201218131710736.png)

加锁过程会去判断Locked标志位是否为0，如果是0则把Locked位置1，代表加锁成功。从上图可见，加锁成功后，只是Locked位置1，其他状态位没发生变化。



* 阻塞加锁

如果该协程需要加锁时，这个锁已经被其他协程获取加锁了

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/20210329191100.png)



从上图可看到，当协程B对一个已被占用的锁再次加锁时，Waiter计数器增加了1，此时协程B将被阻塞，直到Locked值变为0后才会被唤醒。



* 简单解锁

假定解锁时，没有其他协程阻塞，此时解锁过程如下图所示：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/20210329191350.png)

* 解锁并唤醒

假定解锁时，有1个或多个协程阻塞，此时解锁过程如下图所示：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/20210329191622.png)

协程A解锁过程分为两个步骤，一是把Locked位置0，二是查看到Waiter>0，所以释放一个信号量，唤醒一个阻塞的协程，被唤醒的协程B把Locked位置1，于是协程B获得锁。



#### 4.3 自旋

加锁时，如果当前的locked为1，说明当前该锁被其他协程持有，尝试加锁的协程并不是直接进行阻塞而是持续的进行探测LockedI是否变为0 ，这个过程称为自旋过程。自旋的过程十分短暂，如果在自旋过程中发现锁已经被释放，那么该协程就会立即获取这个锁，此时就算有其他的协程被唤醒也不能拿到锁只能再次进行阻塞。自旋的好处是当加锁失败时并不会立即进入阻塞，有一定的机会会获得锁，这样可以避免协程的切换。

**什么是自旋？**

自旋对应的是CPU的PAUSE命令，ＣＰＵ对指令什么都不做，相当于CPU 空转，对于程序而言是相当于sleep了一段时间（与sleep不同的是不需要将协程转为睡眠状态），时间非常短。自旋的过程会持续不断的探测Locked是否变为0，连续两次探测的间隔就是执行PAUSE的指令。

**自旋的条件**

加锁时，程序会自动判断是否需要进行自旋，无限制的自旋会给CPU带来巨大的压力，所以判断是否可以自选就非常重要了。自旋必须看满足以下条件：

* 自旋的次数要足够少 通常为4 即最多自旋4次（超过自旋的次数之后就会阻塞的状态）
* CPU的核数要大于1，否者自旋没有意义一次此时不可能有其他协程释放锁
* 协程调度机制中的Process数量需要大于1 比如使用GOMAXPROCS()将处理器的数量设置为1就不能自旋。
* 协程调度机制中的可运行队列必须为空否者会延迟协程调度。

**自旋的优势**

自旋的优势是充分的利用CPU尽量避免协程的切换，如果经过短时间的自旋就可以获得锁那么当前的协程可以继续运行不必阻塞。

**自旋的问题**

如果自旋过程中获得锁，那么之前被阻塞的协程就无法获得锁。如果加锁的协程特别多每次都能通过自旋获得锁那么之前阻塞的协程就会一直被阻塞进入饥饿状态。为了避免协程长时间获取不到锁，从1.8增加了一个Mutex的Starving状态，在这个状态下不会进行自旋，一旦有协程协程释放锁就会唤醒阻塞的协程并成功加锁。



#### 4.4 Mutex模式

* Normal 模式

默认情况下就是Normal模式，在该模式下如果加锁不成功不会立即进入阻塞模式而是判断是否能够自旋，如果可以进行自旋抢锁

* Starving模式

如果阻塞的协程时间超过了1 ms就会就会进入Starving模式 此时不能进行自旋。



#### 4.5 Woken模式

Woken状态用于加锁和解锁过程中的通信。也就是在同一个时刻一个进行解锁一个加锁，那么就不需要释放信号量了。



#### 4.6 重复解锁触发Panic

如果实现重复解锁不触发Panic，那么会频繁的唤醒多个协程，多个协程唤醒后会在Lock()中进行抢锁，增加Lock()的实现复杂度，引起不必要的协程切换。



#### 4.7 编程Tips

* 使用defer避免死锁

加锁后立即使用defer进行解锁

* 加锁和解锁成对出现

加锁和解锁最好出现在同一层次的代码块中。

