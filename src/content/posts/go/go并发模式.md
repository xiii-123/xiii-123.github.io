---
title: Go并发模式
published: 2025-04-21
description: 'Go并发模式'
image: ''
tags: [Go, 并发, 并发模式]
category: 'Go'
draft: false 
---

### 生产者消费者模式

最经典的模式，在Go中主要使用channel来实现。

```go
// 生产者: 生成 factor 整数倍的序列
func Producer(factor int, out chan<- int) {
    for i := 0; ; i++ {
        out <- i*factor
    }
}

// 消费者
func Consumer(in <-chan int) {
    for v := range in {
        fmt.Println(v)
    }
}
func main() {
    ch := make(chan int, 64) // 成果队列

    go Producer(3, ch) // 生成 3 的倍数的序列
    go Producer(5, ch) // 生成 5 的倍数的序列
    go Consumer(ch)    // 消费生成的队列

    // Ctrl+C 退出
    sig := make(chan os.Signal, 1)
    signal.Notify(sig, syscall.SIGINT, syscall.SIGTERM)
    fmt.Printf("quit (%v)\n", <-sig)
}
```

### 发布订阅模式
发布订阅（publish-and-subscribe）模型通常被简写为 pub/sub 模型。在这个模型中，消息生产者成为发布者（publisher），而消息消费者则成为订阅者（subscriber），生产者和消费者是 M:N 的关系。在传统生产者和消费者模型中，是将消息发送到一个队列中，而发布订阅模型则是将消息发布给一个主题。

```go
// Package pubsub implements a simple multi-topic pub-sub library.
package main

import (
	"fmt"
	"strings"
	"sync"
	"time"
)

type (
	subscriber chan interface{}         // 订阅者为一个管道
	topicFunc  func(v interface{}) bool // 主题为一个过滤器
)

// 发布者对象
type Publisher struct {
	m           sync.RWMutex             // 读写锁
	buffer      int                      // 订阅队列的缓存大小
	timeout     time.Duration            // 发布超时时间
	subscribers map[subscriber]topicFunc // 订阅者信息
}

// 构建一个发布者对象, 可以设置发布超时时间和缓存队列的长度
func NewPublisher(publishTimeout time.Duration, buffer int) *Publisher {
	return &Publisher{
		buffer:      buffer,
		timeout:     publishTimeout,
		subscribers: make(map[subscriber]topicFunc),
	}
}

// 添加一个新的订阅者，订阅全部主题
func (p *Publisher) Subscribe() chan interface{} {
	return p.SubscribeTopic(nil)
}

// 添加一个新的订阅者，订阅过滤器筛选后的主题
func (p *Publisher) SubscribeTopic(topic topicFunc) chan interface{} {
	ch := make(chan interface{}, p.buffer)
	p.m.Lock()
	p.subscribers[ch] = topic
	p.m.Unlock()
	return ch
}

// 退出订阅
func (p *Publisher) Evict(sub chan interface{}) {
	p.m.Lock()
	defer p.m.Unlock()

	delete(p.subscribers, sub)
	close(sub)
}

// 发布一个主题
func (p *Publisher) Publish(v interface{}) {
	p.m.RLock()
	defer p.m.RUnlock()

	var wg sync.WaitGroup
	for sub, topic := range p.subscribers {
		wg.Add(1)
		go p.sendTopic(sub, topic, v, &wg)
	}
	wg.Wait()
}

// 关闭发布者对象，同时关闭所有的订阅者管道。
func (p *Publisher) Close() {
	p.m.Lock()
	defer p.m.Unlock()

	for sub := range p.subscribers {
		delete(p.subscribers, sub)
		close(sub)
	}
}

// 发送主题，可以容忍一定的超时
func (p *Publisher) sendTopic(
	sub subscriber, topic topicFunc, v interface{}, wg *sync.WaitGroup,
) {
	defer wg.Done()
	if topic != nil && !topic(v) {
		return
	}

	select {
	case sub <- v:
	case <-time.After(p.timeout):
	}
}

func main() {
	p := NewPublisher(100*time.Millisecond, 10)
	defer p.Close()

	all := p.Subscribe()
	golang := p.SubscribeTopic(func(v interface{}) bool {
		if s, ok := v.(string); ok {
			return strings.Contains(s, "golang")
		}
		return false
	})

	p.Publish("hello,  world!")
	p.Publish("hello, golang!")

	go func() {
		for msg := range all {
			fmt.Println("all:", msg)
		}
	}()

	go func() {
		for msg := range golang {
			fmt.Println("golang:", msg)
		}
	}()

	// 运行一定时间后退出
	time.Sleep(3 * time.Second)
}
```

### 控制并发数量

很多用户在适应了 Go 语言强大的并发特性之后，都倾向于编写最大并发的程序，因为这样似乎可以提供最大的性能。在现实中我们行色匆匆，但有时却需要我们放慢脚步享受生活，并发的程序也是一样：有时候我们需要适当地控制并发的程度，因为这样不仅仅可给其它的应用/任务让出/预留一定的 CPU 资源，也可以适当降低功耗缓解电池的压力。

在 Go 语言自带的 godoc 程序实现中有一个 vfs 的包对应虚拟的文件系统，在 vfs 包下面有一个 gatefs 的子包，gatefs 子包的目的就是为了控制访问该虚拟文件系统的最大并发数。gatefs 包的应用很简单：

```go
import (
    "golang.org/x/tools/godoc/vfs"
    "golang.org/x/tools/godoc/vfs/gatefs"
)

func main() {
    fs := gatefs.New(vfs.OS("/path"), make(chan bool, 8))
    // ...
}
```
其中 vfs.OS("/path") 基于本地文件系统构造一个虚拟的文件系统，然后 gatefs.New 基于现有的虚拟文件系统构造一个并发受控的虚拟文件系统。并发数控制的原理在前面一节已经讲过，就是通过带缓存管道的发送和接收规则来实现最大并发阻塞：

```go
var limit = make(chan int, 3)

func main() {
    for _, w := range work {
        go func() {
            limit <- 1
            w()
            <-limit
        }()
    }
    select{}
}
```
不过 gatefs 对此做一个抽象类型 gate，增加了 enter 和 leave 方法分别对应并发代码的进入和离开。当超出并发数目限制的时候，enter 方法会阻塞直到并发数降下来为止。

```go
type gate chan bool

func (g gate) enter() { g <- true }
func (g gate) leave() { <-g }
```
gatefs 包装的新的虚拟文件系统就是将需要控制并发的方法增加了 enter 和 leave 调用而已：
```go

type gatefs struct {
    fs vfs.FileSystem
    gate
}

func (fs gatefs) Lstat(p string) (os.FileInfo, error) {
    fs.enter()
    defer fs.leave()
    return fs.fs.Lstat(p)
}
```
我们不仅可以控制最大的并发数目，而且可以通过带缓存 Channel 的使用量和最大容量比例来判断程序运行的并发率。当管道为空的时候可以认为是空闲状态，当管道满了时任务是繁忙状态，这对于后台一些低级任务的运行是有参考价值的。

### context 包
在 Go1.7 发布时，标准库增加了一个 context 包，用来简化对于处理单个请求的多个 Goroutine 之间与请求域的数据、超时和退出等操作，官方有博文对此做了专门介绍。我们可以用 context 包来重新实现前面的线程安全退出或超时的控制:

```go
func worker(ctx context.Context, wg *sync.WaitGroup) error {
    defer wg.Done()

    for {
        select {
        default:
            fmt.Println("hello")
        case <-ctx.Done():
            return ctx.Err()
        }
    }
}

func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)

    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go worker(ctx, &wg)
    }

    time.Sleep(time.Second)
    cancel()

    wg.Wait()
}
```
当并发体超时或 main 主动停止工作者 Goroutine 时，每个工作者都可以安全退出。

Go 语言是带内存自动回收特性的，因此内存一般不会泄漏。在前面素数筛的例子中，GenerateNatural 和 PrimeFilter 函数内部都启动了新的 Goroutine，当 main 函数不再使用管道时后台 Goroutine 有泄漏的风险。我们可以通过 context 包来避免这个问题，下面是改进的素数筛实现：

```go
// 返回生成自然数序列的管道: 2, 3, 4, ...
func GenerateNatural(ctx context.Context) chan int {
    ch := make(chan int)
    go func() {
        for i := 2; ; i++ {
            select {
            case <- ctx.Done():
                return
            case ch <- i:
            }
        }
    }()
    return ch
}

// 管道过滤器: 删除能被素数整除的数
func PrimeFilter(ctx context.Context, in <-chan int, prime int) chan int {
    out := make(chan int)
    go func() {
        for {
            if i := <-in; i%prime != 0 {
                select {
                case <- ctx.Done():
                    return
                case out <- i:
                }
            }
        }
    }()
    return out
}

func main() {
    // 通过 Context 控制后台 Goroutine 状态
    ctx, cancel := context.WithCancel(context.Background())

    ch := GenerateNatural(ctx) // 自然数序列: 2, 3, 4, ...
    for i := 0; i < 100; i++ {
        prime := <-ch // 新出现的素数
        fmt.Printf("%v: %v\n", i+1, prime)
        ch = PrimeFilter(ctx, ch, prime) // 基于新素数构造的过滤器
    }

    cancel()
}
```
当 main 函数完成工作前，通过调用 cancel() 来通知后台 Goroutine 退出，这样就避免了 Goroutine 的泄漏。

然而，上面这个例子只是展示了 cancel() 的基础用法，实际上这个例子会导致 Goroutine 死锁，不能正常退出。 我们可以给上面这个例子添加 sync.WaitGroup 来复现这个问题。

```go
package main

import (
    "context"
    "fmt"
    "sync"
)

// 返回生成自然数序列的管道: 2, 3, 4, ...
func GenerateNatural(ctx context.Context, wg *sync.WaitGroup) chan int {
    ch := make(chan int)
    go func() {
        defer wg.Done()
        for i := 2; ; i++ {
            select {
            case <-ctx.Done():
                return
            case ch <- i:
            }
        }
    }()
    return ch
}

// 管道过滤器: 删除能被素数整除的数
func PrimeFilter(ctx context.Context, in <-chan int, prime int, wg *sync.WaitGroup) chan int {
    out := make(chan int)
    go func() {
        defer wg.Done()
        for {
            if i := <-in; i%prime != 0 {
                select {
                case <-ctx.Done():
                    return
                case out <- i:
                }
            }
        }
    }()
    return out
}

func main() {
    wg := sync.WaitGroup{}
    // 通过 Context 控制后台 Goroutine 状态
    ctx, cancel := context.WithCancel(context.Background())
    wg.Add(1)
    ch := GenerateNatural(ctx, &wg) // 自然数序列: 2, 3, 4, ...
    for i := 0; i < 100; i++ {
        prime := <-ch // 新出现的素数
        fmt.Printf("%v: %v\n", i+1, prime)
        wg.Add(1)
        ch = PrimeFilter(ctx, ch, prime, &wg) // 基于新素数构造的过滤器
    }

    cancel()
    wg.Wait()
}
```
执行上面这个例子很容易就复现了死锁的问题，原因是素数筛中的 ctx.Done() 位于 if i := <-in; i%prime != 0 判断之内， 而这个判断可能会一直阻塞，导致 Goroutine 无法正常退出。

1. PrimeFilter 的阻塞点：

   - PrimeFilter 的 goroutine 中存在 for 循环，每次循环直接执行 i := <-in（在 select 之外），然后处理数据。

   - 若上游的 in 通道没有数据且未被关闭，i := <-in 会一直阻塞，导致后续的 select（检查 Context 取消）无法执行，从而 goroutine 无法退出。

2. Context 取消后的问题：

   - 当 main 调用 cancel() 后，所有 goroutine 应通过 ctx.Done() 退出。

   - 但 PrimeFilter 的 goroutine 卡在 i := <-in 时，无法进入检查 ctx.Done() 的逻辑，因此永远阻塞在通道读取，无法调用 wg.Done()。

3. WaitGroup 计数器未归零：
   - main 函数通过 wg.Wait() 等待所有 goroutine 完成。

   - 由于 PrimeFilter 的 goroutine 无法退出，WaitGroup 的计数器无法归零，导致 wg.Wait() 永久阻塞，引发死锁。

根本原因:
- 通道读取与 Context 监听分离：PrimeFilter 的代码将 i := <-in 放在 select 外，导致无法在等待通道数据时响应 Context 取消。

- 通道未关闭：上游 goroutine 退出时（如 Context 取消），未关闭通道，导致下游 goroutine 的 i := <-in 永久阻塞。

让我们来解决这个问题。

```go
package main

import (
    "context"
    "fmt"
    "sync"
)

// 返回生成自然数序列的管道: 2, 3, 4, ...
func GenerateNatural(ctx context.Context, wg *sync.WaitGroup) chan int {
    ch := make(chan int)
    go func() {
        defer wg.Done()
        for i := 2; ; i++ {
            select {
            case <-ctx.Done():
                return
            case ch <- i:
            }
        }
    }()
    return ch
}

// 管道过滤器: 删除能被素数整除的数
func PrimeFilter(ctx context.Context, in <-chan int, prime int, wg *sync.WaitGroup) chan int {
    out := make(chan int)
    go func() {
        defer wg.Done()
        for {
            select {
            case <-ctx.Done():
                return
            case i := <-in:
                if i%prime != 0 {
                    select {
                    case <-ctx.Done():
                        return
                    case out <- i:
                    }
                }
            }

        }
    }()
    return out
}

func main() {
    wg := sync.WaitGroup{}
    // 通过 Context 控制后台 Goroutine 状态
    ctx, cancel := context.WithCancel(context.Background())
    wg.Add(1)
    ch := GenerateNatural(ctx, &wg) // 自然数序列: 2, 3, 4, ...
    for i := 0; i < 100; i++ {
        prime := <-ch // 新出现的素数
        fmt.Printf("%v: %v\n", i+1, prime)
        wg.Add(1)
        ch = PrimeFilter(ctx, ch, prime, &wg) // 基于新素数构造的过滤器
    }

    cancel()
    wg.Wait()
}
```
如上所示，我们可以通过将 i := <-in 放入 select，在这个 select 内也执行 <-ctx.Done() 来解决阻塞导致的死锁。 不过上面这个例子并不优美，让我们换一种方式。

```go
package main

import (
    "context"
    "fmt"
    "sync"
)

// 返回生成自然数序列的管道: 2, 3, 4, ...
func GenerateNatural(ctx context.Context, wg *sync.WaitGroup) chan int {
    ch := make(chan int)
    go func() {
        defer wg.Done()
        defer close(ch)
        for i := 2; ; i++ {
            select {
            case <-ctx.Done():
                return
            case ch <- i:
            }
        }
    }()
    return ch
}

// 管道过滤器: 删除能被素数整除的数
func PrimeFilter(ctx context.Context, in <-chan int, prime int, wg *sync.WaitGroup) chan int {
    out := make(chan int)
    go func() {
        defer wg.Done()
        defer close(out)
        for i := range in {
            if i%prime != 0 {
                select {
                case <-ctx.Done():
                    return
                case out <- i:
                }
            }
        }
    }()
    return out
}

func main() {
    wg := sync.WaitGroup{}
    // 通过 Context 控制后台 Goroutine 状态
    ctx, cancel := context.WithCancel(context.Background())
    wg.Add(1)
    ch := GenerateNatural(ctx, &wg) // 自然数序列: 2, 3, 4, ...
    for i := 0; i < 100; i++ {
        prime := <-ch // 新出现的素数
        fmt.Printf("%v: %v\n", i+1, prime)
        wg.Add(1)
        ch = PrimeFilter(ctx, ch, prime, &wg) // 基于新素数构造的过滤器
    }

    cancel()
    wg.Wait()
}
```
在上面这个例子中主要有以下几点需要关注：

通过 for range 循环保证了输入管道被关闭时，循环能退出，不会出现死循环；
通过 defer close 保证了无论是输入管道被关闭，还是 ctx 被取消，只要素数筛退出，都会关闭输出管道。
至此，我们终于足够优美地解决了这个死锁问题。