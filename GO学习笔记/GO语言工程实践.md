## GO语言工程实践

## 语言简单进阶

### 并发与并行

并发：多线程在一个核cpu上运行

并行：多线程在多个核cpu上运行

Go可以发挥多核优势，高效运行

### Goroutine

- 线程
    - 内核态、同一线程上可以跑多个协程、栈一般MB级别
- 协程
    - 用户态、轻量级线程、栈一般KB级别

#### Goroutine ege:

```go	
//快速打印hello x
func hello(i int) {
	println("hello " + fmt.Sprint(i))
}

func main() {
	for i := 0; i < 5; i++ {
		go func(j int) { //go即可开启协程
			hello(j)
		}(i)
	}
	time.Sleep(time.Second) //主线程睡一会等待协程输出
}

// hello 4
// hello 2
// hello 3
// hello 0
// hello 1
```

顺序并不一致，可以看出是并行执行的

#### CSP(communicating Sequential Processes) 

提倡通过**通信共享内存**而不是通过共享内存实现通信

- 通过通信有序，而通过共享内存需要加锁

#### Channel

`make(chan 元素类型,[缓存大小])`

channel是并发安全的

- 无缓冲通道 	`make(chan int)`	
- 有缓冲通道     `make(chan int,2)`

```go
//A 子协程发送0~9的数字
//B 子协程计算输出数字的平方
//主协程输出最后的平方数
package main

func main() {
	src := make(chan int)     //创建无缓冲区的协程
	dest := make(chan int, 3) //创建缓冲区大小为3的协程
	go func() {               //生产协程
		defer close(src)
		for i := 0; i < 10; i++ {
			src <- i
		}
	}()
	go func() { //消费协程
		defer close(dest)
		for i := range src {
			dest <- i * i
		}
	}()
	for i := range dest {
		//复杂操作，使用带缓冲的chan可以解决生产者消费者速度不一致的问题
		println(i)
	}
}
```

#### 并发安全Lock

使用lock机制解决临界区并发安全问题

```go
lock sync.Mutex

//在访问临界区资源时，先加锁，再解锁
lock.Lock()
x+=1
lock.Unlock()
```

#### WaitGroup

维护了一个等待组计数器,给外界提供了3个接口

- `Add(delta int)`:计数器+delta
- `Done()`:计数器-1，这个函数内部实现是通过调用`Add(-1)`实现的
- `Wait()`:阻塞等待直到计数器为0

```go	
func hello(i int) {
	println("hello " + fmt.Sprint(i))
}

func main() {
	var wg sync.WaitGroup
	wg.Add(5) //wg等待组+5
	for i := 0; i < 5; i++ {
		go func(j int) { //go即可开启协程
			defer wg.Done() //执行完后-1
			hello(j)
		}(i)
	}
	wg.Wait() //阻塞等待
}

// hello 4
// hello 2
// hello 3
// hello 0
// hello 1
```

## Go依赖管理

1. 配置文件，描述依赖		go.mod
2. 中心仓库管理依赖库        Proxy
3. 本地工具                            go get / mod

## 测试

### 单元测试

- 所有测试文件以_test.go结尾
- `func TestXxxx(*testing.T)`
- 初始化逻辑放到TestMain中

### Mock测试

### 基准测试

性能的测试

## 项目分析

### 分层结构

- 数据层：数据Model,外部数据的增删查改
- 逻辑层：业务Entity，处理核心业务的逻辑输出
- 视图层：视图view，处理和外部的交互逻辑

### 组件工具

- Gin框架
- Go Mod


