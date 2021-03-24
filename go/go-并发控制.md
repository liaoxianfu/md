### 并发控制

对于在一些场景中需要在一个协程中等待另一个或者多个协程的信息，举例用户下单后，创建订单需要确保在通知商家确定有货，收取金额后再进行数据库订单写入的操作。此时就需要等待通知商家和收取金额两个go routine成功通知数据库写入。Go提供了三个方案分别是channel、WaitGroup和Context



channel

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

WaitGroup

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

Context

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

emptyCtx 定义

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

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
	if parent == nil {
		panic("cannot create context from nil parent")
	}
	c := newCancelCtx(parent)
	propagateCancel(parent, &c)
	return &c, func() { c.cancel(true, Canceled) }
}


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

func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
	return WithDeadline(parent, time.Now().Add(timeout))
}


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





cancelCtx

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