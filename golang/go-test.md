# go test介绍

Golang 测试文件名必须以 _test.go 结尾

包含单元测试、基准测试、子测试、Example和Main

## 1、单元测试

创建一个文件base_test.go，输入下面的内容

```go
package main

import (
  "math"
  "testing"
)

func TestAbs(t *testing.T) {
  got := math.Abs(-1)
  if got != 1 {
    t.Errorf("Abs(-1) = %f; want 1", got)

  }

}
```

```shell
go test base_test.go
```

输出

```
ok      command-line-arguments  0.004s
```



## 2、基准测试

创建一个文件bench_test.go，输入下面的内容

```go
package main

import (
  "fmt"
  "testing"
)

func BenchmarkHello(b *testing.B) {
  // skip when run with short
  if testing.Short() {
    for i := 0; i < b.N; i++ {
    	fmt.Sprintf("hello")
  	}
  }
  
}

// 使用所有的协成并行执行测试代码，总共b.N次
func BenchmarkHelloParallel(b *testing.B) {
  b.RunParallel(func(pb *testing.PB) {
    for pb.Next() {
      fmt.Sprintf("hello")
    }
  })
}

```

```shell
go test -bench=Hello -cpu=1 bench_test.go
```

输出

```
goos: linux
goarch: amd64
BenchmarkHello  15776000                79.0 ns/op
PASS
ok      command-line-arguments  2.104s
```

## 3、Example

创建一个文件example_test.go，输入下面的内容

```go
package main

import "fmt"

func ExampleHello() {
  fmt.Println("hello") // 如果输出不是hello则报错
  // Output: hello

}
```

```shell
go test example_test.go
```

输出

```
ok      command-line-arguments  0.005s
```

## 4、Subtests and Sub-benchmarks

```
func TestFoo(t *testing.T) {
    // <setup code>
    t.Run("A=1", func(t *testing.T) { ... })
    t.Run("A=2", func(t *testing.T) { ... })
    t.Run("B=1", func(t *testing.T) { ... })
    // <tear-down code>
}
go test -run ''      # Run all tests.
go test -run Foo     # Run top-level tests matching "Foo", such as "TestFooBar".
go test -run Foo/A=  # For top-level tests matching "Foo", run subtests matching "A=".
go test -run /A=1    # For all top-level tests, run subtests matching "A=1".
```

## 5、Main

```go
func TestMain(m *testing.M) {
	// call flag.Parse() here if TestMain uses flags
	os.Exit(m.Run())
}
```



## 命令详解

go test 是 go 语言自带的测试工具，其中包含的是两类，单元测试(即 功能测试) 和 性能测试

通过 `go help test` 可以看到 go test 的使用说明：

### 格式：

```shell
go test [-c] [-i] [build flags] [packages] [flags  for test binary]
```

### 参数：

-c : 编译 go test 成为可执行的二进制文件，但是不运行测试。

-i : 安装测试包依赖的 package，但是不运行测试。

关于 build flags，调用 `go help build`，这些是编译运行过程中需要使用到的参数，一般设置为空

关于 packages，调用 `go help packages`，这些是关于包的管理，一般设置为空

关于 flags for test binary，调用 `go help testflag`，这些是 go test 过程中经常使用到的参数：

-test.v : 是否输出全部的单元测试用例（不管成功或者失败），默认没有加上，所以只输出失败的单元测试用例

-test.run pattern : 只跑哪些单元测试用例

-test.bench patten : 只跑那些性能测试用例

-test.benchmem : 是否在性能测试的时候输出内存情况

-test.benchtime t : 性能测试运行的时间，默认是1s

-test.cpuprofile cpu.out : 是否输出cpu性能分析文件

-test.memprofile mem.out : 是否输出内存性能分析文件

-test.blockprofile block.out : 是否输出内部goroutine阻塞的性能分析文件

-test.memprofilerate n : 内存性能分析的时候有一个分配了多少的时候才打点记录的问题。这个参数就是设置打点的内存分配间隔，也就是 profile 中一个 sample 代表的内存大小。默认是设置为 512 * 1024 的。如果你将它设置为 1，则每分配一个内存块就会在 profile 中有个打点，那么生成的 profile 的 sample 就会非常多。如果你设置为 0，那就是不做打点了

你可以通过设置 memprofilerate=1 和 GOGC=off 来关闭内存回收，并且对每个内存块的分配进行观察

-test.blockprofilerate n : 基本同上，控制的是 goroutine 阻塞时候打点的纳秒数。默认不设置就相当于 -test.blockprofilerate=1，每一纳秒都打点记录一下

-test.parallel n : 性能测试的程序并行 cpu 数，默认等于 GOMAXPROCS

-test.timeout t : 如果测试用例运行时间超过 t，则抛出 panic

-test.cpu 1,2,4 : 程序运行在哪些 CPU 上面，使用二进制的 1 所在位代表，和 nginx 的 nginx_worker_cpu_affinity 是一个道理

-test.short : 将那些运行时间较长的测试用例运行时间缩短