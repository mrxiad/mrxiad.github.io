---
title: go操作rabbitmq
date: 2024-4-9 20:33:12
tags: rabbitmq
categories: 
- rabbitmq
---





# 简单案例(默认交换机)



> **使用默认交换机的时候要用队列name作为key**



- **生产者** 通过指定路由键来发送消息，这里的路由键是队列名称。
- **消费者** 必须监听相同的路由键（即队列名称），以接收生产者发送的消息。

## 生产者

```go
package main

import (
	"fmt"
	"log"
	"os"
	"strings"

	"github.com/streadway/amqp"
)

func main() {
	// 1. 尝试连接RabbitMQ，建立连接
	// 该连接抽象了套接字连接，并为我们处理协议版本协商和认证等。
	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	if err != nil {
		fmt.Printf("connect to RabbitMQ failed, err:%v\n", err)
		return
	}
	defer conn.Close()

	// 2. 接下来，我们创建一个通道，大多数API都是用过该通道操作的。
	ch, err := conn.Channel()
	if err != nil {
		fmt.Printf("open a channel failed, err:%v\n", err)
		return
	}
	defer ch.Close()

	// 3. 要发送，我们必须创建要发送到的队列。
	q, err := ch.QueueDeclare(
		"task_queue", // name
		true,         // 持久的
		false,        // delete when unused
		false,        // 独有的
		false,        // no-wait
		nil,          // arguments
	)
	if err != nil {
		fmt.Printf("declare a queue failed, err:%v\n", err)
		return
	}

	// 4. 然后我们可以将消息发布到创建的队列
	body := bodyFrom(os.Args)
	err = ch.Publish(
		"",     // exchange
        q.Name, // routing key(这里是队列名称)
		false,  // 立即
		false,  // 强制
		amqp.Publishing{
			DeliveryMode: amqp.Persistent, // 持久
			ContentType:  "text/plain",
			Body:         []byte(body),
		})
	if err != nil {
		fmt.Printf("publish a message failed, err:%v\n", err)
		return
	}
	log.Printf(" [x] Sent %s", body)
}

// bodyFrom 从命令行获取将要发送的消息内容
func bodyFrom(args []string) string {
	var s string
	if (len(args) < 2) || os.Args[1] == "" {
		s = "hello"
	} else {
		s = strings.Join(args[1:], " ")
	}
	return s
}
```



## 消费者

```go
package main

import (
	"bytes"
	"fmt"
	"log"
	"time"

	"github.com/streadway/amqp"
)

func main() {
	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	if err != nil {
		fmt.Printf("connect to RabbitMQ failed, err:%v\n", err)
		return
	}
	defer conn.Close()

	ch, err := conn.Channel()
	if err != nil {
		fmt.Printf("open a channel failed, err:%v\n", err)
		return
	}
	defer ch.Close()

	// 创建一个queue
	q, err := ch.QueueDeclare(
		"task_queue", // name
		true,         // 创建为持久队列
		false,        // delete when unused
		false,        // exclusive
		false,        // no-wait
		nil,          // arguments
	)
	err = ch.Qos(
		1,     // prefetch count
		0,     // prefetch size
		false, // global
	)
	if err != nil {
		fmt.Printf("ch.Qos() failed, err:%v\n", err)
		return
	}

	// 立即返回一个Delivery的通道
	msgs, err := ch.Consume(
		q.Name, // queue
		"",     // consumer
		false,  // 注意这里传false,关闭自动消息确认
		false,  // exclusive
		false,  // no-local
		false,  // no-wait
		nil,    // args
	)
	if err != nil {
		fmt.Printf("ch.Consume failed, err:%v\n", err)
		return
	}

	// 开启循环不断地消费消息
	forever := make(chan bool)
	go func() {
		for d := range msgs {
			log.Printf("Received a message: %s", d.Body)
			dotCount := bytes.Count(d.Body, []byte("."))
			t := time.Duration(dotCount)
			time.Sleep(t * time.Second)
			log.Printf("Done")
			d.Ack(false) // 手动传递消息确认
		}
	}()

	log.Printf(" [*] Waiting for messages. To exit press CTRL+C")
	<-forever
}
```



> 这段代码将消息发布到默认交换机，使用队列名称作为路由键，不要求消息必须立即被消费，也不要求消息一定要被路由到队列。消息被标记为持久的，并以纯文本格式发送。









# 交换机（fanout模式）

> 消费者可以创建交换机，防止 启动消费者的时候没有交换机导致的错误

## 生产者

```go
package main

import (
        "log"
        "os"
        "strings"

        "github.com/streadway/amqp"
)

func failOnError(err error, msg string) {
        if err != nil {
                log.Fatalf("%s: %s", msg, err)
        }
}

func main() {
        conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
        failOnError(err, "Failed to connect to RabbitMQ")
        defer conn.Close()

        ch, err := conn.Channel()
        failOnError(err, "Failed to open a channel")
        defer ch.Close()

    	//创建一个交换机
        err = ch.ExchangeDeclare(
                "logs",   // name
                "fanout", // type
                true,     // durable
                false,    // auto-deleted
                false,    // internal
                false,    // no-wait
                nil,      // arguments
        )
        failOnError(err, "Failed to declare an exchange")

        body := bodyFrom(os.Args)
    	
    	//发布到交换机
        err = ch.Publish(
                "logs", // exchange
                "",     // routing key
                false,  // mandatory
                false,  // immediate
                amqp.Publishing{
                        ContentType: "text/plain",
                        Body:        []byte(body),
                })
        failOnError(err, "Failed to publish a message")

        log.Printf(" [x] Sent %s", body)
}

func bodyFrom(args []string) string {
        var s string
        if (len(args) < 2) || os.Args[1] == "" {
                s = "hello"
        } else {
                s = strings.Join(args[1:], " ")
        }
        return s
}
```



## 消费者

```go
package main

import (
        "log"

        "github.com/streadway/amqp"
)

func failOnError(err error, msg string) {
        if err != nil {
                log.Fatalf("%s: %s", msg, err)
        }
}

func main() {
        conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
        failOnError(err, "Failed to connect to RabbitMQ")
        defer conn.Close()

        ch, err := conn.Channel()
        failOnError(err, "Failed to open a channel")
        defer ch.Close()

    	//创建交换机
        err = ch.ExchangeDeclare(
                "logs",   // name
                "fanout", // type
                true,     // durable
                false,    // auto-deleted
                false,    // internal
                false,    // no-wait
                nil,      // arguments
        )
        failOnError(err, "Failed to declare an exchange")
		
    	//创建队列
        q, err := ch.QueueDeclare(
                "",    // name
                false, // durable
                false, // delete when unused
                true,  // exclusive
                false, // no-wait
                nil,   // arguments
        )
        failOnError(err, "Failed to declare a queue")

    	//绑定队列到交换机
        err = ch.QueueBind(
                q.Name, // queue name
                "",     // routing key
                "logs", // exchange
                false,
                nil,
        )
        failOnError(err, "Failed to bind a queue")

    	//获取消息channel
        msgs, err := ch.Consume(
                q.Name, // queue
                "",     // consumer
                true,   // auto-ack
                false,  // exclusive
                false,  // no-local
                false,  // no-wait
                nil,    // args
        )
        failOnError(err, "Failed to register a consumer")

        forever := make(chan bool)

        go func() {
                for d := range msgs {
                        log.Printf(" [x] %s", d.Body)
                }
        }()

        log.Printf(" [*] Waiting for logs. To exit press CTRL+C")
        <-forever
}
```





# 路由（direct模式）

绑定可以采用额外的`routing_key`参数。绑定密钥的含义取决于交换器的类型。我们以前使用的`fanout`交换器只是忽略了这个值

```go
err = ch.QueueBind(
  q.Name,    // queue name
  "black",   // routing key
  "logs",    // exchange
  false,
  nil)
```





## 绑定类型

### 直连交换器

一个key仅仅和一个queue绑定



### 多重绑定

多个key可以和一个queue绑定



## example

### 生产者

```go
package main

import (
        "log"
        "os"
        "strings"

        "github.com/streadway/amqp"
)

func failOnError(err error, msg string) {
        if err != nil {
                log.Fatalf("%s: %s", msg, err)
        }
}

func main() {
    	//创建连接
        conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
        failOnError(err, "Failed to connect to RabbitMQ")
        defer conn.Close()
	
        ch, err := conn.Channel()
        failOnError(err, "Failed to open a channel")
        defer ch.Close()
		
    	//创建交换机
        err = ch.ExchangeDeclare(
                "logs_direct", // name
                "direct",      // type
                true,          // durable
                false,         // auto-deleted
                false,         // internal
                false,         // no-wait
                nil,           // arguments
        )
        failOnError(err, "Failed to declare an exchange")
		
    	//获取参数
        body := bodyFrom(os.Args)
        err = ch.Publish(
                "logs_direct",         // exchange
                severityFrom(os.Args), // routing key
                false, // mandatory
                false, // immediate
                amqp.Publishing{
                        ContentType: "text/plain",
                        Body:        []byte(body),
                })
        failOnError(err, "Failed to publish a message")

        log.Printf(" [x] Sent %s", body)
}

func bodyFrom(args []string) string {
        var s string
        if (len(args) < 3) || os.Args[2] == "" {
                s = "hello"
        } else {
                s = strings.Join(args[2:], " ")
        }
        return s
}

func severityFrom(args []string) string {
        var s string
        if (len(args) < 2) || os.Args[1] == "" {
                s = "info"
        } else {
                s = os.Args[1]
        }
        return s
}
```





### 消费者

```go
package main

import (
        "log"
        "os"

        "github.com/streadway/amqp"
)

func failOnError(err error, msg string) {
        if err != nil {
                log.Fatalf("%s: %s", msg, err)
        }
}

func main() {
        conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
        failOnError(err, "Failed to connect to RabbitMQ")
        defer conn.Close()

        ch, err := conn.Channel()
        failOnError(err, "Failed to open a channel")
        defer ch.Close()

    	//创建交换机
        err = ch.ExchangeDeclare(
                "logs_direct", // name
                "direct",      // type
                true,          // durable
                false,         // auto-deleted
                false,         // internal
                false,         // no-wait
                nil,           // arguments
        )
        failOnError(err, "Failed to declare an exchange")
		
    	//创建队列
        q, err := ch.QueueDeclare(
                "",    // name
                false, // durable
                false, // delete when unused
                true,  // exclusive
                false, // no-wait
                nil,   // arguments
        )
        failOnError(err, "Failed to declare a queue")

        if len(os.Args) < 2 {
                log.Printf("Usage: %s [info] [warning] [error]", os.Args[0])
                os.Exit(0)
        }
    
    	//绑定队列到交换机
        for _, s := range os.Args[1:] {
                log.Printf("Binding queue %s to exchange %s with routing key %s",
                        q.Name, "logs_direct", s)
                err = ch.QueueBind(
                        q.Name,        // queue name
                        s,             // routing key
                        "logs_direct", // exchange
                        false,
                        nil)
                failOnError(err, "Failed to bind a queue")
        }

    	//获取msg channel
        msgs, err := ch.Consume(
                q.Name, // queue
                "",     // consumer
                true,   // auto ack
                false,  // exclusive
                false,  // no local
                false,  // no wait
                nil,    // args
        )
        failOnError(err, "Failed to register a consumer")

        forever := make(chan bool)

        go func() {
                for d := range msgs {
                        log.Printf(" [x] %s", d.Body)
                }
        }()

        log.Printf(" [*] Waiting for logs. To exit press CTRL+C")
        <-forever
}
```







# 主题(topic模式)

## 绑定类型

发送到`topic`交换器的消息不能具有随意的`routing_key`——它必须是单词列表，以点分隔。这些词可以是任何东西，但通常它们指定与消息相关的某些功能。一些有效的`routing_key`示例：“`stock.usd.nyse`”，“`nyse.vmw`”，“`quick.orange.rabbit`”。`routing_key`中可以包含任意多个单词，最多255个字节。



绑定键也必须采用相同的形式。`topic`交换器背后的逻辑类似于`direct`交换器——用特定路由键发送的消息将传递到所有匹配绑定键绑定的队列。但是，绑定键有两个重要的特殊情况：

- `*`（星号）可以代替一个单词。
- `＃`（井号）可以替代零个或多个单词。



> topic交换器功能强大，可以像其他交换器一样运行。
>
> 当队列用“`#`”（井号）绑定键绑定时，它将接收所有消息，而与路由键无关，就像在`fanout`交换器中一样。
>
> 当在绑定中不使用特殊字符“`*`”（星号）和“`#`”（井号）时，topic交换器的行为就像`direct`交换器一样。



## example

### 生产者

```go
package main

import (
        "log"
        "os"
        "strings"

        "github.com/streadway/amqp"
)

func failOnError(err error, msg string) {
        if err != nil {
                log.Fatalf("%s: %s", msg, err)
        }
}

func main() {
        conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
        failOnError(err, "Failed to connect to RabbitMQ")
        defer conn.Close()

        ch, err := conn.Channel()
        failOnError(err, "Failed to open a channel")
        defer ch.Close()

        err = ch.ExchangeDeclare(
                "logs_topic", // name
                "topic",      // type
                true,         // durable
                false,        // auto-deleted
                false,        // internal
                false,        // no-wait
                nil,          // arguments
        )
        failOnError(err, "Failed to declare an exchange")

        body := bodyFrom(os.Args)
        err = ch.Publish(
                "logs_topic",          // exchange
                severityFrom(os.Args), // routing key
                false, // mandatory
                false, // immediate
                amqp.Publishing{
                        ContentType: "text/plain",
                        Body:        []byte(body),
                })
        failOnError(err, "Failed to publish a message")

        log.Printf(" [x] Sent %s", body)
}

func bodyFrom(args []string) string {
        var s string
        if (len(args) < 3) || os.Args[2] == "" {
                s = "hello"
        } else {
                s = strings.Join(args[2:], " ")
        }
        return s
}

func severityFrom(args []string) string {
        var s string
        if (len(args) < 2) || os.Args[1] == "" {
                s = "anonymous.info"
        } else {
                s = os.Args[1]
        }
        return s
}
```



### 消费者

```go
package main

import (
	"log"
	"os"

	"github.com/streadway/amqp"
)

func failOnError(err error, msg string) {
	if err != nil {
		log.Fatalf("%s: %s", msg, err)
	}
}

func main() {
	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	failOnError(err, "Failed to connect to RabbitMQ")
	defer conn.Close()

	ch, err := conn.Channel()
	failOnError(err, "Failed to open a channel")
	defer ch.Close()

	err = ch.ExchangeDeclare(
		"logs_topic", // name
		"topic",      // type
		true,         // durable
		false,        // auto-deleted
		false,        // internal
		false,        // no-wait
		nil,          // arguments
	)
	failOnError(err, "Failed to declare an exchange")

	q, err := ch.QueueDeclare(
		"",    // name
		false, // durable
		false, // delete when unused
		true,  // exclusive
		false, // no-wait
		nil,   // arguments
	)
	failOnError(err, "Failed to declare a queue")

	if len(os.Args) < 2 {
		log.Printf("Usage: %s [binding_key]...", os.Args[0])
		os.Exit(0)
	}
	// 绑定topic
	for _, s := range os.Args[1:] {
		log.Printf("Binding queue %s to exchange %s with routing key %s",
			q.Name, "logs_topic", s)
		err = ch.QueueBind(
			q.Name,       // queue name
			s,            // routing key
			"logs_topic", // exchange
			false,
			nil)
		failOnError(err, "Failed to bind a queue")
	}

	msgs, err := ch.Consume(
		q.Name, // queue
		"",     // consumer
		true,   // auto ack
		false,  // exclusive
		false,  // no local
		false,  // no wait
		nil,    // args
	)
	failOnError(err, "Failed to register a consumer")

	forever := make(chan bool)

	go func() {
		for d := range msgs {
			log.Printf(" [x] %s", d.Body)
		}
	}()

	log.Printf(" [*] Waiting for logs. To exit press CTRL+C")
	<-forever
}
```





# RPC

使用默认交换机

通常，通过RabbitMQ进行RPC很容易。客户端发送请求消息，服务器发送响应消息。为了接收响应，我们需要发送带有“回调”队列地址的请求。我们可以使用默认队列。

```go
q, err := ch.QueueDeclare(
  "",    // 不指定队列名，默认使用随机生成的队列名
  false, // durable
  false, // delete when unused
  true,  // exclusive
  false, // noWait
  nil,   // arguments
)

err = ch.Publish(
  "",          // exchange
  "rpc_queue", // routing key
  false,       // mandatory
  false,       // immediate
  amqp.Publishing{
    ContentType:   "text/plain",
    CorrelationId: corrId,
    ReplyTo:       q.Name,  // 在这里指定callback队列名，也是在这个队列等回复
    Body:          []byte(strconv.Itoa(n)),
})
```

我们的RPC工作流程如下：

- 客户端启动时，它将创建一个匿名排他回调队列。
- 对于RPC请求，客户端发送一条消息，该消息具有两个属性：`reply_to`（设置为回调队列）和`correlation_id`（设置为每个请求的唯一值）。
- 该请求被发送到`rpc_queue`队列。
- RPC工作程序（又名：服务器）正在等待该队列上的请求。当出现请求时，它会完成计算工作并把结果作为消息使用`replay_to`字段中的队列发回给客户端。
- 客户端等待回调队列上的数据。出现消息时，它将检查`correlation_id`属性。如果它与请求中的值匹配，则将响应返回给应用程序。



## example

### 服务器

```go
package main

import (
	"github.com/streadway/amqp"
	"log"
	"strconv"
)

func failOnError(err error, msg string) {
	if err != nil {
		log.Fatalf("%s: %s", msg, err)
	}
}

func fib(n int) int {
	if n == 0 {
		return 0
	} else if n == 1 {
		return 1
	} else {
		return fib(n-1) + fib(n-2)
	}
}

func main() {
	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	failOnError(err, "Failed to connect to RabbitMQ")
	defer conn.Close()

	ch, err := conn.Channel()
	failOnError(err, "Failed to open a channel")
	defer ch.Close()

	q, err := ch.QueueDeclare(
		"rpc_queue", // name
		false,       // durable
		false,       // delete when unused
		false,       // exclusive
		false,       // no-wait
		nil,         // arguments
	)
	failOnError(err, "Failed to declare a queue")

	err = ch.Qos(
		1,     // prefetch count
		0,     // prefetch size
		false, // global
	)
	failOnError(err, "Failed to set QoS")

	msgs, err := ch.Consume(
		q.Name, // queue
		"",     // consumer
		false,  // auto-ack
		false,  // exclusive
		false,  // no-local
		false,  // no-wait
		nil,    // args
	)
	failOnError(err, "Failed to register a consumer")

	forever := make(chan bool)

	//异步处理消息
	go func() {
		for d := range msgs {
			n, err := strconv.Atoi(string(d.Body))
			failOnError(err, "Failed to convert body to integer")

			log.Printf(" [.] fib(%d)", n)
			response := fib(n) //计算斐波那契数列

			//返回结果
			err = ch.Publish(
				"",        // exchange
				d.ReplyTo, //	routing key
				false,     // mandatory
				false,     // immediate
				amqp.Publishing{
					ContentType:   "text/plain",
					CorrelationId: d.CorrelationId,
					Body:          []byte(strconv.Itoa(response)),
				})
			failOnError(err, "Failed to publish a message")

			d.Ack(false) //手动确认消息
		}
	}()

	log.Printf(" [*] Awaiting RPC requests")
	<-forever
}
```





### 客户端

```go
package main

import (
	"log"
	"math/rand"
	"os"
	"strconv"
	"strings"
	"time"

	"github.com/streadway/amqp"
)

func failOnError(err error, msg string) {
	if err != nil {
		log.Fatalf("%s: %s", msg, err)
	}
}

func randomString(l int) string {
	bytes := make([]byte, l)
	for i := 0; i < l; i++ {
		bytes[i] = byte(randInt(65, 90))
	}
	return string(bytes)
}

func randInt(min int, max int) int {
	return min + rand.Intn(max-min)
}

func fibonacciRPC(n int) (res int, err error) {
	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	failOnError(err, "Failed to connect to RabbitMQ")
	defer conn.Close()

	ch, err := conn.Channel()
	failOnError(err, "Failed to open a channel")
	defer ch.Close()

	q, err := ch.QueueDeclare(
		"",    // name
		false, // durable
		false, // delete when unused
		true,  // exclusive
		false, // noWait
		nil,   // arguments
	)
	failOnError(err, "Failed to declare a queue")

	msgs, err := ch.Consume(
		q.Name, // queue
		"",     // consumer
		true,   // auto-ack
		false,  // exclusive
		false,  // no-local
		false,  // no-wait
		nil,    // args
	)
	failOnError(err, "Failed to register a consumer")

	corrId := randomString(32) //随机生成一个32位的字符串

	err = ch.Publish(
		"",          // exchange
		"rpc_queue", // routing key
		false,       // mandatory
		false,       // immediate
		amqp.Publishing{
			ContentType:   "text/plain",
			CorrelationId: corrId, //关联ID
			ReplyTo:       q.Name, //回调队列
			Body:          []byte(strconv.Itoa(n)),
		})
	failOnError(err, "Failed to publish a message")

	for d := range msgs {
		if corrId == d.CorrelationId {
			res, err = strconv.Atoi(string(d.Body))
			failOnError(err, "Failed to convert body to integer")
			break
		}
	}

	return
}

func main() {
	rand.Seed(time.Now().UTC().UnixNano())

	n := bodyFrom(os.Args)

	log.Printf(" [x] Requesting fib(%d)", n)
	res, err := fibonacciRPC(n)
	failOnError(err, "Failed to handle RPC request")

	log.Printf(" [.] Got %d", res)
}

func bodyFrom(args []string) int {
	var s string
	if (len(args) < 2) || os.Args[1] == "" {
		s = "30"
	} else {
		s = strings.Join(args[1:], " ")
	}
	n, err := strconv.Atoi(s)
	failOnError(err, "Failed to convert arg to integer")
	return n
}
```





# 总结

1. 声明队列是幂等的——**仅当队列不存在时才创建**。消息内容是一个字节数组，因此你可以在此处编码任何内容。
2. 







## 各种参数的含义

### `ExchangeDeclare` 参数

当你声明一个交换器时，你定义了一个消息传递的环节，它决定了消息如何根据路由键和绑定键被路由到队列中。

- **name**: 交换器的名称，这在后续操作中用于指向这个特定的交换器。
- **type**: 交换器的类型，决定了消息路由的行为。常见类型有`direct`, `topic`, `fanout`, 和`headers`。`topic`类型允许你根据多重条件（路由键的模式匹配）来路由消息。
- **durable**: 如果设置为`true`，交换器将在服务器重启后继续存在。这有助于确保消息不会因为服务器重启而丢失。
- **auto-delete**: 如果设置为`true`，当最后一个绑定到交换器上的队列被删除后，交换器也会自动删除。
- **internal**: 如果设置为`true`，交换器不能被客户端直接用于消息发布，只能被其他交换器用于交换器到交换器的绑定。
- **no-wait**: 如果设置为`true`，服务器将不会响应方法。客户端不会等待声明完成的确认。
- **arguments**: 一些交换器类型特有的参数，用于扩展AMQP的功能。

### `Publish` 参数

用于发布消息到交换器。

- **exchange**: 要发布到的交换器名称。消息会被路由到与此交换器绑定的队列。
- **routing key**: 用于路由消息的键。根据交换器类型的不同，这个键可以用于不同的匹配规则。
- **mandatory**: 如果设置为`true`，当消息无法路由到队列时，它将返回给发送者。
- **immediate**: 这是一个不再被使用的参数，用于指示如果没有消费者立即可用来处理消息，就将消息返回给发送者。建议不要使用它，因为它可能不被所有服务器支持。
- **Publishing**: 消息的内容和属性，比如`ContentType`和`Body`。

### `QueueDeclare` 参数

声明一个队列，如果它不存在，则创建。

- **name**: 队列的名称。如果留空，服务器将为队列分配一个随机名称。
- **durable**: 如果设置为`true`，队列将在服务器重启后继续存在。
- **delete when unused**: 如果设置为`true`，当没有任何消费者时，队列会被自动删除。
- **exclusive**: 如果设置为`true`，队列将只对首次声明它的连接可见，并在连接关闭时自动删除。
- **no-wait**: 类似于交换器的`no-wait`，如果设置为`true`，服务器不会对这个操作发送确认。
- **arguments**: 一些队列特有的参数，用于扩展AMQP的功能。

### `Consume` 参数

用于接收消息的消费者设置。

- **queue**: 从哪个队列接收消息。
- **consumer**: 消费者标签，用于区分多个消费者。
- **auto ack**: 如果设置为`true`，服务器会自动确认收到的消息，这意味着一旦消息被送达，它就会从队列中移除。如果设置为`false`，消费者需要显式地发送acknowledgment来确认已经处理了消息。
- **exclusive**: 如果设置为`true`，这个消费者将成为队列的唯一消费者。
- **no local**: 如果设置为`true`，服务器将不会将消息发送给发布它们的连接。
- **no wait**: 类似于其他`no-wait`，如果设置为`true`，不等待服务器的响应。
- **args**: 消费者特有的参数。





## 持久性的含义

- **交换器的持久性（Durable Exchange）**：当一个交换器被声明为持久的时，它指的是交换器的定义和配置信息在服务器重启后仍然存在。这样，你就不需要在每次服务器启动时重新声明交换器的配置。但这与交换器内的消息无关，因为交换器不存储消息。
- **队列的持久性（Durable Queue）**：声明一个队列为持久的，意味着队列的定义将在服务器重启后仍然存在。更重要的是，只有当队列是持久的，并且消息也被标记为持久的（即在发布时将`delivery_mode`属性设置为2），这些消息才会在服务器重启后保持在队列中。需要注意的是，即使队列是持久的，如果消息没有被标记为持久的，那么在服务器重启时这些消息还是会丢失。



## 保证消息持久

1. **声明一个持久的队列**：在创建队列时，将 `durable` 参数设置为 `true`。这表示队列会在服务器重启后继续存在，并且能够存储持久化消息。
2. **发送消息时设置消息为持久**：在**发布消息**时，需要将消息的 `delivery_mode` 属性设置为 `2`（表示消息是持久的）。这确保了消息即使在服务器重启的情况下也不会丢失，前提是它们存储在持久的队列中。





## 控制消息流向消费者

默认情况下，RabbitMQ将按顺序将每个消息发送给下一个消费者。平均而言，每个消费者都会收到相同数量的消息。这种分发消息的方式称为**轮询**。

在一个有两个`worker`的情况下，当所有的奇数消息都是重消息而偶数消息都是轻消息时，一个`worker`将持续忙碌，而另一个`worker`几乎不做任何工作。嗯，RabbitMQ对此一无所知，仍然会均匀地发送消息。

设置**预取计数（prefetch count）**是一种控制消息流向消费者（workers）的有效方式，特别是在处理涉及多个消费者的任务分发时。这个设置影响了 RabbitMQ 如何平衡任务负载，确保工作不会被过载给某个特定的消费者，而是更公平地分配给所有可用的消费者。

```go
err := ch.Qos(
  1,     // prefetch count
  0,     // prefetch size
  false, // global
)
if err != nil {
  log.Fatalf("Failed to set QoS: %s", err)
}
```

- **prefetch count**: 控制 RabbitMQ 发送给单个消费者的消息数量，直到该消费者确认（ack）一些消息之前。设置为 `1` 意味着 RabbitMQ 在消费者确认处理完当前消息之前，不会给它发送新的消息，这样可以确保工作负载在多个消费者之间公平分配。
- **prefetch size**: 控制 RabbitMQ 发送给消费者的消息总体大小的限制。设置为 `0` 表示不对消息体大小做限制。
- **global**: 指示预取设置是应用于整个通道上的所有消费者（`true`），还是仅仅应用于当前命令调用的消费者（`false`）。对于大多数应用场景，将其设置为 `false` 表示这个限制是针对每个消费者的。