---
title: go语法
date: 2024-3-24 22:40:40
tags: go
categories: 
- go
- 定时器
---





# Timer定时器



## 创建定时器

`Timer` 用于在未来的某一时刻执行单一的时间事件。创建一个 `Timer` 实例后，你可以等待它的时间通道 (`C`) 发出信号，表示定时器已经到期。这种机制非常适合需要延迟执行的场景。

- **创建Timer**: 通过 `time.NewTimer(duration)` 创建一个定时器，`duration` 指定了从现在开始到定时器触发的时间长度。

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	timer1 := time.NewTimer(2 * time.Second)
	time.Sleep(1 * time.Second)
	time2 := time.Now()
	fmt.Println(time2)
	<-timer1.C //等待定时器到期
	time2 = time.Now()
	fmt.Println(time2)
}
```

- 通过`time.After()`函数实现

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	<-time.After(2 * time.Second)
	fmt.Println("2秒到")
}
```



## 停止定时器

`定时器.Stop()`

其返回值代表定时器有没有超时：

- true: true: 定时器超时前停止，后续不会再有事件发送；

- false: 定时器超时后停止；

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	timer4 := time.NewTimer(2 * time.Second)
	go func() {
		<-timer4.C
		fmt.Println("定时器执行了")
	}()
	b := timer4.Stop()
	if b {
		fmt.Println("timer4已经关闭")
	}
}
```



## 重置定时器

`定时器.Reset(持续时间)`





## AferFunc()

time.AfterFunc() **是异步执行的**，所以需要在函数最后sleep等待指定的协程退出，否则可能函数结束时协程还未执行

```go
package main

import (
	"log"
	"time"
)

func AfterFuncDemo() {
	log.Println("AfterFuncDemo start: ", time.Now())
	time.AfterFunc(1*time.Second, func() {				//时间，函数
		log.Println("AfterFuncDemo end: ", time.Now())
	})
	time.Sleep(2 * time.Second) // 等待协程退出
}
func main() {
	AfterFuncDemo()
}
```





# **Ticker** 定时器

**Ticker****是周期性定时器**，即周期性的触发一个事件，通过Ticker本身提供的管道将事件传递出去



- **作用**: 周期性地触发事件。

- **创建**: 使用 `time.NewTicker(duration)` 创建，`duration` 是触发周期。

- **停止**: 使用 `ticker.Stop()` 停止定时器。停止后不会再向通道发送时间，但通道不会被关闭。

- 示例

	```go
	func TickerDemo() {
	    ticker := time.NewTicker(1 * time.Second)
	    defer ticker.Stop()
	    for range ticker.C {
	        log.Println("Ticker tick.")
	    }
	}
	```

### 简单定时任务

使用 `time.Tick(duration)`

- 适用于需要定时执行任务但不需要停止定时器的场景。

- 注意: 不能停止，可能导致资源泄露。

- 示例

	```go
	for tick := range time.Tick(3 * time.Second){
	    fmt.Println(tick)
	}
	```

### 定时聚合任务

- **场景**: 模拟公交车定时发车，或在满载时提前发车。

- **实现方法**: 使用 `Ticker` 进行定时检查，并根据条件（如时间到达或达到最大乘客数）执行任务。

- 示例

	```go
	// 假设有函数 GetNewPassenger() 获取新乘客，Launch(passengers) 发车
	func TickerLaunch() {
	    ticker := time.NewTicker(5 * time.Minute)
	    maxPassenger := 30
	    passengers := make([]string, 0, maxPassenger)
	    for {
	        passenger := GetNewPassenger()
	        if passenger != "" {
	            passengers = append(passengers, passenger)
	        }
	        select {
	        case <-ticker.C:
	            Launch(passengers)
	            passengers = []string{}
	        default:
	            if len(passengers) >= maxPassenger {
	                Launch(passengers)
	                passengers = []string{}
	            }
	        }
	    }
	}
	```

