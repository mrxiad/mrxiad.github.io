---
title: go语法
date: 2024-3-24 22:40:40
tags: go
categories: 
- go
- 多协程
---











# sync包

## sync.Mutex

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var (
	mutex       sync.Mutex
	shared_data int
)

func Add1() {
	for i := 0; i < 1000; i++ {
		mutex.Lock()
		shared_data++
		mutex.Unlock()
	}
}

func Add2() {
	for i := 0; i < 1000; i++ {
		mutex.Lock()
		shared_data++
		mutex.Unlock()
	}
}
func main() {
	go Add1()
	go Add2()
	time.Sleep(1 * time.Second)
	fmt.Println(shared_data)
}
```





## sync.RWMutex

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var (
	mutex       sync.RWMutex
	shared_data int
)

func Add1() {
	for i := 0; i < 1000; i++ {
		mutex.Lock()
		shared_data++
		mutex.Unlock()
	}
}

func Add2() {
	for i := 0; i < 1000; i++ {
		mutex.Lock()
		shared_data++
		mutex.Unlock()
	}
}
func main() {
	go Add1()
	go Add2()
	time.Sleep(1 * time.Second)
	fmt.Println(shared_data)
}
```





## sync.WaitGroup

```go
package main

import (
	"fmt"
	"net/http"
	"sync"
)

func main() {
	// 声明一个等待组 对一组等待任务只需要一个等待组，而不需要每一个任务都使用一个等待组
	var wg sync.WaitGroup
	// 准备一系列的网站地址的字符串切片
	var urls = []string{
		"http://www.baidu.com/",
		"https://www.mi.com/",
		"https://pkg.go.dev/",
	}
	// 遍历这些地址
	for _, url := range urls {
		// 每一个任务开始时, 将等待组增加1，也就是每一个任务加 1
		wg.Add(1)
		// 将一个匿名函数开启并发
		go func(url string) {
			// 使用defer, 表示函数完成时将等待组值减1
			defer wg.Done()
			// 使用http访问提供的地址，Get() 函数会一直阻塞直到网站响应或者超时
			_, err := http.Get(url)
			// 访问完成后, 打印地址和可能发生的错误
			fmt.Println(url, err)
			// 通过参数传递url地址
		}(url)
	}
	// 等待所有的任务完成，任务完成，Wait 就会停止阻塞
	wg.Wait()
	fmt.Println("over")
}
```





#  **runtime**

Go 语言的 goroutine 是由 运行时（runtime）调度和管理的。它负责管理包括内存分配、垃圾回收、

栈处理、goroutine、channel、切片（slice）、map 和反射（reflection）等等。





## runtime.Gosched()

让出当前协程的 CPU 时间片给其他协程。当前协程等待时间片未来继续执行。

```go
package main
import (
	"fmt"
	"runtime"
)
func main() {
	go func(s string) {
		for i := 0; i < 2; i++ {
			fmt.Println(s)
		}
	}("world")
	// 主协程
	for i := 0; i < 2; i++ {
		runtime.Gosched() //主协程释放CPU时间片，此时上面的协程得以执行
		fmt.Println("hello") //CPU时间片回来后继续执行
	}
}
#结果
world
world
hello
hello
```





## runtime.Goexit()

退出当前协程，但是 defer 语句会照常执行。

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

func main() {
	go func() {
		defer fmt.Println("A.defer")
		func() {
			defer fmt.Println("B.defer")
			runtime.Goexit() // 结束当前协程
			defer fmt.Println("C.defer")
			fmt.Println("B")
		}()
		fmt.Println("A")
	}()
	time.Sleep(time.Second) // 睡一会儿，不让主协程很快结束
	fmt.Println("main")
}
#结果
B.defer
A.defer
main
```





## runtime.GOMAXPROCS()

Golang 默认所有任务都运行在一个 cpu 核里，如果要在 goroutine 中使用多核，可以使用runtime.GOMAXPROCS 函数修改，当参数小于 1 时使用默认值。

Go运行时的调度器使用 GOMAXPROCS 参数来指定需要使用多少个 OS 线程来同时执行 Go 代码。**默认值是机器上的 CPU 核心数**。例如在一个 8 核心的机器上，调度器会把 Go 代码同时调度到 8 个 OS 线程上（ GOMAXPROCS 是m:n调度中的n）。

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

func a() {
	for i := 1; i < 10; i++ {
		fmt.Println("A:", i)
	}
}
func b() {
	for i := 1; i < 10; i++ {
		fmt.Println("B:", i)

	}
}
func main() {
	runtime.GOMAXPROCS(runtime.NumCPU()) //可以通过调整核心数看看执行效果
	go a()
	go b()
	time.Sleep(time.Second * 2)
	fmt.Println("over")
}
```





## runtime.NumCPU()

返回当前系统的 CPU 核数量。





## runtime.GOROOT()

获取 goroot 目录。





## runtime.GOOS

查看目标操作系统。





## runtime.GC

会让运行时系统进行一次强制性的垃圾收集。

强制的垃圾回收：不管怎样，都要进行的垃圾回收。非强制的垃圾回收：只会在一定条件下进行的垃圾回收（即运行时，系统自上次垃圾回收之后新申请的堆内存的单元（也成为单元增量）达到指定的数值）。



# runtime.LockOSThread , runtime.UnlockOSThread

runtime.LockOSThread调用会使调用他的 Goroutine 与当前运行它的M锁定到一起。

runtime.UnlockOSThread调用会解除这样的锁定。



# runtime.NumGoroutine

返回正在执行和排队的任务总数。

runtime.NumGoroutine函数在被调用后，会返回系统中的处于特定状态的 Goroutine 的数量。这里的特定状态是指Grunnable\Gruning\Gsyscall\Gwaition。处于这些状态的Groutine即被看做是活跃的或者说正在被调度。

注意：垃圾回收所在Groutine的状态也处于这个范围内的话，也会被纳入该计数器。