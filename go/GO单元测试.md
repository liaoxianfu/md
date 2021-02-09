## 单元测试

Go本身提供了一套轻量级的测试框架。符合规则的测试代码会在运行测试时被自动识别病执行。单元测试源文件命名需要符合**在需要测试的包下创建**`xx_test.go`测试文件。GO单元测试函数分为功能型测试函数和性能测试函数，分别为以Test和Benchmark为函数前缀并以`t *testing.T`为单一参数的函数

```go
func TestAdd(t *testing.T)
func Benchmark(t *testing.T)
```

测试工具会根据函数中的实际执行动作来得到不同的测试结果。功能测试函数会根据测试代码执行过程中是否发生错误来返回不同的结果，而性能测试函数仅仅打印测试过程中花费的时间。

举个简单的例子

创建一个go mod 项目具体如下图所展示

![20210208201616.png](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/20210208201616.png)

### 单元测试

```go
// math.go

package mymath

func Add(a, b int) (sum int) {
	sum = a + b
	return sum
}

```

```go
// math_test.go
package mymath

import "testing"

func TestAdd(t *testing.T) {
	r := Add(1, 2)
	if r != 3 {
		t.Errorf("Add(1,2) failed Go %d expected 3", r)
	}
}

```



进行测试

```sh
go test -v ./mymath

============================
=== RUN   TestAdd
--- PASS: TestAdd (0.00s)
PASS
ok      unitest/mymath  
```



出错的情况

```sh
=== RUN   TestAdd
    math_test.go:8: Add(1,2) failed Go 5 expected 3
--- FAIL: TestAdd (0.00s)
FAIL
FAIL    unitest/mymath  0.002s
FAIL

```



### 性能测试

```go
package mymath

import (
	"testing"
	"time"
)

func TestAdd(t *testing.T) {
	......
}

// BenchmarkAdd 用于测试性能
func BenchmarkAdd(b *testing.B) {
	b.StopTimer() // 暂停计时器
	// 模拟非测试函数的耗时 排除这个耗时
	time.Sleep(time.Second * 2) // 模拟一个耗时的准备工作 例如读取文件
	//开启计时器
	b.StartTimer()
	// 开始进行测试性能
	for i := 0; i < b.N; i++ {
		Add(1, 2)
	}
}

```



测试执行 (使用 `--run=none` 避免运行普通的测试函数, 因为一般不可能有函数名匹配 `none`):

```sh
go test -v  -bench="BenchmarkAdd$" -run=none  ./mymath
```



```sh
liao@liao ~/g/s/unitest> go test -v  -bench="BenchmarkAdd$" -run=none  ./mymath
goos: linux
goarch: amd64
pkg: unitest/mymath
BenchmarkAdd
BenchmarkAdd-4          1000000000               0.397 ns/op
PASS
ok      unitest/mymath  12.471s
```



从测试结果可以看出一共测试了1000000000轮 平均耗时0.397 ns 

### 性能分析

执行

```zai
 go test -v  -bench="BenchmarkAdd$" -run=none  -cpuprofile cpu.out  ./mymath 
```

执行完毕在项目的根目录生成`cpu.out`和`mymath.test`文件

![20210208204435.png](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/20210208204435.png)

可以生成cpu的profile 利用`graphviz`进行可视化的查询

```bash
go tool pprof -http=":" cpu.out
```

![20210208204558.png](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/20210208204558.png)

![20210208204614.png](https://cdn.jsdelivr.net/gh/liaoxianfu/blogimg/data/20210208204614.png)

### 测试覆盖率

```sh
go test -v   -coverprofile=cover.out ./mymath
```



假设添加一个新的函数

```go
// math.go
package mymath

func Add(a, b int) (sum int) {
	.....
}
// 添加减法函数
func Sub(a,b int)(res int){
	return a-b
}
```

```sh
liao@liao ~/g/s/unitest> go test -v   -coverprofile=cover.out ./mymath                                                                                                                                     1
=== RUN   TestAdd
--- PASS: TestAdd (0.00s)
PASS
coverage: 66.7% of statements
ok      unitest/mymath  0.002s  coverage: 66.7% of statements
```

检查测试覆盖率

```sh
liao@liao ~/g/s/unitest> go tool cover -func=cover.out                
unitest/mymath/math.go:3:       Add             100.0%
unitest/mymath/math.go:8:       Sub             0.0%
total:                          (statements)    66.7%
```





使用goland进行快捷测试

使用Goland在每个测试函数或者包的前面有绿色的选项，可以根据自己的需要进行上面的测试。