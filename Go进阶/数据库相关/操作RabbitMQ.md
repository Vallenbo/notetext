这些教程涵盖了使用RabbitMQ创建消息传递应用程序的基础知识。 你需要安装RabbitMQ服务器才能完成这些教程，请参阅[安装指南](https://www.rabbitmq.com/download.html)或使用[Docker镜像](https://registry.hub.docker.com/_/rabbitmq/)。 这些教程的代码是[开源](https://github.com/rabbitmq/rabbitmq-tutorials)的，[官方网站](https://github.com/rabbitmq/rabbitmq-website)也是如此。

# RabbitMQ Go教程1—HelloWorld

## 先决条件

本教程假设RabbitMQ已安装并运行在本机上的标准端口（5672）。如果你使用不同的主机、端口或凭据，则需要调整连接设置。

### 介绍

RabbitMQ是一个消息代理：它接受并转发消息。你可以把它想象成一个邮局：当你把你想要邮寄的邮件放进一个邮箱时，你可以确定邮差先生或女士最终会把邮件送到你的收件人那里。在这个比喻中，RabbitMQ是一个邮箱、一个邮局和一个邮递员。

RabbitMQ和邮局的主要区别在于它不处理纸张，而是接受、存储和转发二进制数据块——消息。

RabbitMQ和一般的消息传递都使用一些术语。

- 生产仅意味着发送。发送消息的程序是生产者：

  ![img](./assets/producer.png)

- 队列是位于RabbitMQ内部的邮箱的名称。尽管消息通过RabbitMQ和你的应用程序流动，但它们只能存储在队列中。队列只受主机内存和磁盘限制的限制，实际上它是一个大的消息缓冲区。许多生产者可以向一个队列发送消息，而许多消费者可以尝试从一个队列接收数据。以下是我们表示队列的方式：

  ![img](./assets/queue.png)

- 消费与接收具有相似的含义。消费者是一个主要等待接收消息的程序：

  ![img](./assets/consumer.png)

请注意，生产者，消费者和代理（broker）不必位于同一主机上。实际上，在大多数应用程序中它们不是。一个应用程序既可以是生产者，也可以是消费者。

### “Hello World”

**(使用Go RabbitMQ客户端）**

在本教程的这一部分中，我们将在Go中编写两个小程序：发送单个消息的生产者和接收消息并将其打印出来的消费者。我们将忽略[Go-RabbitMQ](http://godoc.org/github.com/streadway/amqp) API中的一些细节，只关注非常简单的事情，以便开始教程。这是一个消息传递版的“Hello World”。

在下图中，“ P”是我们的生产者，“ C”是我们的消费者。中间的框是一个队列——RabbitMQ代表消费者保存的消息缓冲区。

![(P) -> [|||] -> (C)](./assets/python-one.png)

> Go RabbitMQ客户端库
>
> RabbitMQ讲多种协议。本教程使用amqp0-9-1，这是一个开放的、通用的消息传递协议。RabbitMQ有[许多不同语言的客户端](http://rabbitmq.com/devtools.html)。在本教程中，我们将使用Go amqp客户端。

首先，使用`go get`安装amqp

```bash
go get github.com/streadway/amqp
```

现在安装好amqp之后，我们就可以编写一些代码。

### 发送

![(P) -> [|||]](./assets/sending.png)

我们将消息发布者（发送者）称为 `send.go`，将消息消费者（接收者）称为`receive.go`。发布者将连接到RabbitMQ，发送一条消息，然后退出。

在`send.go`中，我们需要首先导入库：

```go
package main

import (
  "log"

  "github.com/streadway/amqp"
)
```

我们还需要一个辅助函数来检查每个amqp调用的返回值：

```go
func failOnError(err error, msg string) {
  if err != nil {
    log.Fatalf("%s: %s", msg, err)
  }
}
```

然后连接到RabbitMQ服务器

```go
// 1. 尝试连接RabbitMQ，建立连接
// 该连接抽象了套接字连接，并为我们处理协议版本协商和认证等。
conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
failOnError(err, "Failed to connect to RabbitMQ")
defer conn.Close()
```

连接抽象了socket连接，并为我们处理协议版本协商和认证等。接下来，我们创建一个通道，这是大多数用于完成任务的API所在的位置：

```go
// 2. 接下来，我们创建一个通道，大多数API都是用过该通道操作的。
ch, err := conn.Channel()
failOnError(err, "Failed to open a channel")
defer ch.Close()
```

要发送，我们必须声明要发送到的队列。然后我们可以将消息发布到队列：

```go
// 3. 声明消息要发送到的队列
q, err := ch.QueueDeclare(
  "hello", // name
  false,   // durable
  false,   // delete when unused
  false,   // exclusive
  false,   // no-wait
  nil,     // arguments
)
failOnError(err, "Failed to declare a queue")

body := "Hello World!"
// 4.将消息发布到声明的队列
err = ch.Publish(
  "",     // exchange
  q.Name, // routing key
  false,  // mandatory
  false,  // immediate
  amqp.Publishing {
    ContentType: "text/plain",
    Body:        []byte(body),
  })
failOnError(err, "Failed to publish a message")
```

声明队列是幂等的——仅当队列不存在时才创建。消息内容是一个字节数组，因此你可以在此处编码任何内容。

[点击查看完整的send.go文件](https://github.com/rabbitmq/rabbitmq-tutorials/blob/master/go/send.go)

### 接收

上面是我们的发布者。我们的消费者监听来自RabbitMQ的消息，因此与发布单个消息的发布者不同，我们将使消费者保持运行状态以监听消息并打印出来。

![[|||] -> (C)](./assets/receiving.png)

该代码（在`receive.go`中）具有与`send`相同的导入和帮助功能：

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
```

设置与发布者相同；我们打开一个连接和一个通道，并声明要消耗的队列。请注意，这与`send`发布到的队列匹配。

```go
// 建立连接
conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
failOnError(err, "Failed to connect to RabbitMQ")
defer conn.Close()

// 获取channel
ch, err := conn.Channel()
failOnError(err, "Failed to open a channel")
defer ch.Close()

// 声明队列
q, err := ch.QueueDeclare(
  "hello", // name
  false,   // durable
  false,   // delete when unused
  false,   // exclusive
  false,   // no-wait
  nil,     // arguments
)
failOnError(err, "Failed to declare a queue")
```

请注意，我们也在这里声明队列。因为我们可能在发布者之前启动使用者，所以我们希望在尝试使用队列中的消息之前确保队列存在。

我们将告诉服务器将队列中的消息传递给我们。由于它将异步地向我们发送消息，因此我们将在goroutine中从通道（由`amqp::Consume`返回）中读取消息。

```go
// 获取接收消息的Delivery通道
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
    log.Printf("Received a message: %s", d.Body)
  }
}()

log.Printf(" [*] Waiting for messages. To exit press CTRL+C")
<-forever
```

[点击完整的receive.go脚本](https://github.com/rabbitmq/rabbitmq-tutorials/blob/master/go/receive.go)

### 完整示例

现在我们可以运行两个脚本。在一个终端窗口，运行发布者：

```bash
go run send.go
```

然后，运行使用者：

```go
go run receive.go
```

消费者将打印通过RabbitMQ从发布者那里得到的消息。使用者将持续运行，等待消息（使用Ctrl-C停止它），因此请尝试从另一个终端运行发布者。

如果要检查队列，请尝试使用`rabbitmqctl list_queues`命令。

接下来该继续教程的[第二部分](https://www.liwenzhou.com/posts/Go/go_rabbitmq_tutorials_02/)并建立一个简单的任务队列。



# RabbitMQ Go教程2—任务队列

## 先决条件

本教程假设RabbitMQ已安装并运行在本机上的标准端口（5672）。如果你使用不同的主机、端口或凭据，则需要调整连接设置。

## 任务队列/工作队列

**（使用Go RabbitMQ客户端）**

![img](./assets/python-two.png)

在[第一个教程](https://www.liwenzhou.com/posts/Go/go_rabbitmq_tutorials_01/)中，我们编写程序从命名的队列发送和接收消息。在这一节中，我们将创建一个工作队列，该队列将用于在多个工人之间分配耗时的任务。

工作队列（又称任务队列）的主要思想是避免立即执行某些资源密集型任务并且不得不等待这些任务完成。相反，我们安排任务异步地同时或在当前任务之后完成。我们将任务封装为消息并将其发送到队列，在后台运行的工作进程将取出消息并最终执行任务。当你运行多个工作进程时，任务将在他们之间共享。

这个概念在Web应用中特别有用，因为在Web应用中不可能在较短的HTTP请求窗口内处理复杂的任务，（译注：例如注册时发送邮件或短信验证码等场景）。

### 准备工作

在本教程的上一部分，我们发送了一条包含“ Hello World！”的消息。现在，我们将发送代表复杂任务的字符串。我们没有实际的任务，例如调整图像大小或渲染pdf文件，所以我们通过借助`time.Sleep`函数模拟一些比较耗时的任务。我们会将一些包含`.`的字符串封装为消息发送到队列中，其中每有一个`.`就表示需要耗费1秒钟的工作，例如，`hello...`表示一个将花费三秒钟的假任务。

我们将稍微修改上一个示例中的`send.go`代码，以允许从命令行发送任意消息。该程序会将任务安排到我们的工作队列中，因此我们将其命名为`new_task.go`

```go
body := bodyFrom(os.Args)  // 从参数中获取要发送的消息正文
err = ch.Publish(
  "",           // exchange
  q.Name,       // routing key
  false,        // mandatory
  false,
  amqp.Publishing {
    DeliveryMode: amqp.Persistent,
    ContentType:  "text/plain",
    Body:         []byte(body),
  })
failOnError(err, "Failed to publish a message")
log.Printf(" [x] Sent %s", body)
```

下面是`bodyFrom`函数：

```go
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

我们以前的`receive.go`程序也需要进行一些更改：它需要为消息正文中出现的每个`.`伪造一秒钟的工作。它将从队列中弹出消息并执行任务，因此我们将其称为`worker.go`：

```go
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
    log.Printf("Received a message: %s", d.Body)
    dot_count := bytes.Count(d.Body, []byte("."))  // 数一下有几个.
    t := time.Duration(dot_count)
    time.Sleep(t * time.Second)  // 模拟耗时的任务
    log.Printf("Done")
  }
}()

log.Printf(" [*] Waiting for messages. To exit press CTRL+C")
<-forever
```

请注意，我们的假任务模拟执行时间。

然后，我们就可以打开两个终端，分别执行`new_task.go`和`worker.go`了。

```bash
# shell 1
go run worker.go
# shell 2
go run new_task.go
```

### 循环调度

使用任务队列的优点之一是能够轻松并行化工作。如果我们的工作正在积压，我们可以增加更多的工人，这样就可以轻松扩展。

首先，让我们尝试同时运行两个`worker.go`脚本。它们都将从队列中获取消息，但是究竟是怎样呢？让我们来看看。

你需要打开三个控制台。其中两个将运行`worker.go`脚本。这些控制台将成为我们的两个消费者——C1和C2。

```bash
# shell 1
go run worker.go
# => [*] Waiting for messages. To exit press CTRL+C
# shell 2
go run worker.go
# => [*] Waiting for messages. To exit press CTRL+C
```

在第三个控制台中，我们将发布新任务。启动消费者之后，你可以发布一些消息：

```bash
# shell 3
go run new_task.go msg1.
go run new_task.go msg2..
go run new_task.go msg3...
go run new_task.go msg4....
go run new_task.go msg5.....
```

然后我们在`shell1`和 `shell2` 两个窗口看到如下输出结果了：

```bash
# shell 1
go run worker.go
# => [*] Waiting for messages. To exit press CTRL+C
# => [x] Received a message: msg1.
# => [x] Received a message: msg3...
# => [x] Received a message: msg5.....
# shell 2
go run worker.go
# => [*] Waiting for messages. To exit press CTRL+C
# => [x] Received a message: msg2..
# => [x] Received a message: msg4....
```

默认情况下，RabbitMQ将按顺序将每个消息发送给下一个消费者。平均而言，每个消费者都会收到相同数量的消息。这种分发消息的方式称为轮询。使用三个或者更多`worker`试一下。

### 消息确认

work 完成任务可能需要耗费几秒钟，如果一个`worker`在任务执行过程中宕机了该怎么办呢？我们当前的代码中，RabbitMQ一旦向消费者传递了一条消息，便立即将其标记为删除。在这种情况下，如果你终止一个`worker`那么你就可能会丢失这个任务，我们还将丢失所有已经交付给这个`worker`的尚未处理的消息。

我们不想丢失任何任务，如果一个`worker`意外宕机了，那么我们希望将任务交付给其他`worker`来处理。

为了确保消息永不丢失，RabbitMQ支持 [消息*确认*](https://www.rabbitmq.com/confirms.html)。消费者发送回一个确认（acknowledgement），以告知RabbitMQ已经接收，处理了特定的消息，并且RabbitMQ可以自由删除它。

如果使用者在不发送确认的情况下死亡（其通道已关闭，连接已关闭或TCP连接丢失），RabbitMQ将了解消息未完全处理，并将对其重新排队。如果同时有其他消费者在线，它将很快将其重新分发给另一个消费者。这样，您可以确保即使工人偶尔死亡也不会丢失任何消息。

没有任何消息超时；RabbitMQ将在消费者死亡时重新传递消息。即使处理一条消息需要很长时间也没关系。

在本教程中，我们将使用手动消息确认，方法是为“auto-ack”参数传递一个`false`，然后在完成任务后，使用`d.Ack(false)`从`worker`发送一个正确的确认（这将确认一次传递）。

```go
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
```

使用这段代码，我们可以确保即使你在处理消息时使用`CTRL+C`杀死一个`worker`，也不会丢失任何内容。在`worker`死后不久，所有未确认的消息都将被重新发送。

消息确认必须在接收消息的同一通道（Channel）上发送。尝试使用不同的通道（Channel）进行消息确认将导致通道级协议异常。有关更多信息，请参阅[确认的文档指南](https://www.rabbitmq.com/confirms.html)。

> 忘记确认
>
> 忘记确认是一个常见的错误。这是一个简单的错误，但后果是严重的。当你的客户机退出时，消息将被重新传递（这看起来像随机重新传递），但是RabbitMQ将消耗越来越多的内存，因为它无法释放任何未确认的消息。
>
> 为了调试这种错误，可以使用rabbitmqctl打印messages_unacknowledged字段：
>
> ```bash
> sudo rabbitmqctl list_queues name messages_ready messages_unacknowledged
> ```
>
> 在Windows平台，去掉sudo
>
> ```bash
> rabbitmqctl.bat list_queues name messages_ready messages_unacknowledged
> ```

### 消息持久化

我们已经学会了如何确保即使消费者死亡，任务也不会丢失。但是如果RabbitMQ服务器停止运行，我们的任务仍然会丢失。

当RabbitMQ退出或崩溃时，它将忘记队列和消息，除非您告诉它不要这样做。要确保消息不会丢失，需要做两件事：我们需要将队列和消息都标记为持久的。

首先，我们需要确保队列能够在RabbitMQ节点重新启动后继续运行。为此，我们需要声明它是持久的：

```go
q, err := ch.QueueDeclare(
	"hello", // name
	true,    // 声明为持久队列
	false,   // delete when unused
	false,   // exclusive
	false,   // no-wait
	nil,     // arguments
)
```

虽然这个命令本身是正确的，但它在我们当前的设置中不起作用。这是因为我们已经定义了一个名为`hello`的队列，它不是持久的。RabbitMQ不允许你使用不同的参数重新定义现有队列，并将向任何尝试重新定义的程序返回错误。但是有一个快速的解决方法——让我们声明一个具有不同名称的队列，例如`task_queue`：

```go
q, err := ch.QueueDeclare(
	"task_queue", // name
	true,         // 声明为持久队列
	false,        // delete when unused
	false,        // exclusive
	false,        // no-wait
	nil,          // arguments
)
```

这种持久的选项更改需要同时应用于生产者代码和消费者代码。

在这一点上，我们确信即使RabbitMQ重新启动，任务队列队列也不会丢失。现在我们需要将消息标记为持久的——通过使用`amqp.Publishing`中的持久性选项`amqp.Persistent`。

```go
err = ch.Publish(
	"",     // exchange
	q.Name, // routing key
	false,  // 立即
	false,  // 强制
	amqp.Publishing{
		DeliveryMode: amqp.Persistent, // 持久（交付模式：瞬态/持久）
		ContentType:  "text/plain",
		Body:         []byte(body),
	})
```

> 有关消息持久性的说明
>
> 将消息标记为持久性并不能完全保证消息不会丢失。尽管它告诉RabbitMQ将消息保存到磁盘上，但是RabbitMQ接受了一条消息并且还没有保存它时，仍然有一个很短的时间窗口。而且，RabbitMQ并不是对每个消息都执行`fsync(2)`——它可能只是保存到缓存中，而不是真正写入磁盘。持久性保证不是很强，但是对于我们的简单任务队列来说已经足够了。如果您需要更强有力的担保，那么您可以使用[publisher confirms](https://www.rabbitmq.com/confirms.html)。

### 公平分发

你可能已经注意到调度仍然不能完全按照我们的要求工作。例如，在一个有两个`worker`的情况下，当所有的奇数消息都是重消息而偶数消息都是轻消息时，一个`worker`将持续忙碌，而另一个`worker`几乎不做任何工作。嗯，RabbitMQ对此一无所知，仍然会均匀地发送消息。

这是因为RabbitMQ只是在消息进入队列时发送消息。它不考虑消费者未确认消息的数量。只是盲目地向消费者发送信息。

![img](./assets/prefetch-count.png)

为了避免这种情况，我们可以将预取计数设置为`1`。这告诉RabbitMQ不要一次向一个`worker`发出多个消息。或者，换句话说，在处理并确认前一条消息之前，不要向`worker`发送新消息。相反，它将把它发送给下一个不忙的`worker`。

```go
err = ch.Qos(
  1,     // prefetch count
  0,     // prefetch size
  false, // global
)
```

> 关于队列大小的说明
>
> 如果所有的`worker`都很忙，你的`queue`随时可能会满。你会想继续关注这一点，也许需要增加更多的`worker`，或者有一些其他的策略。

### 完整的代码示例

我们的`new_task.go`的最终代码代入如下：

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

	// 3. 要发送，我们必须声明要发送到的队列。
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

	// 4. 然后我们可以将消息发布到声明的队列
	body := bodyFrom(os.Args)
	err = ch.Publish(
		"",     // exchange
		q.Name, // routing key
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

`work.go`内容如下：

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

	// 声明一个queue
	q, err := ch.QueueDeclare(
		"task_queue", // name
		true,         // 声明为持久队列
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

使用消息确认和预取计数，可以设置工作队列（work queue）。即使RabbitMQ重新启动，持久性选项也可以让任务继续存在。

有关`amqp.Channel`方法和消息属性的内容，可以浏览[amqp API文档](http://godoc.org/github.com/streadway/amqp)。



# RabbitMQ Go教程3—发布/订阅





## 发布/订阅

在[上一个教程](https://www.liwenzhou.com/posts/Go/go_rabbitmq_tutorials_02/)中，我们创建了一个工作队列。工作队列背后的假设是每个任务只传递给一个工人。在这一部分中，我们将做一些完全不同的事情——我们将向多个消费者传递一个消息。这就是所谓的“订阅/发布模式”。

为了说明这种模式，我们将构建一个简单的日志系统。它将由两个程序组成——第一个程序将发出日志消息，第二个程序将接收并打印它们。

在我们的日志系统中，每一个运行的接收器程序副本都会收到消息。这样，我们就可以运行一个接收器并将日志定向到磁盘；同时，我们还可以运行另一个接收器并在屏幕上查看日志。

本质上，已发布的日志消息将被广播到所有接收者。

### Exchanges（交换器）

在本教程的前面部分中，我们向队列发送消息和从队列接收消息。现在是时候在Rabbit中引入完整的消息传递模型了。

让我们快速回顾一下先前教程中介绍的内容：

- *生产者*是发送消息的用户应用程序。
- *队列*是存储消息的缓冲区。
- *消费者*是接收消息的用户应用程序。

RabbitMQ消息传递模型中的核心思想是生产者从不将任何消息直接发送到队列。实际上，生产者经常甚至根本不知道是否将消息传递到任何队列。

相反，生产者只能将消息发送到交换器。交换器是非常简单的东西。一方面，它接收来自生产者的消息，另一方面，将它们推入队列。交换器必须确切知道如何处理接收到的消息。它应该被附加到特定的队列吗？还是应该将其附加到许多队列中？或者它应该被丢弃。这些规则由交换器的类型定义。

![img](./assets/exchanges.png)

有几种交换器类型可用：`direct`, `topic`, `headers` 和 `fanout`。我们将集中讨论最后一个——`fanout`。让我们创建一个这种类型的交换器，并给它起个名字叫`logs`：

```go
err = ch.ExchangeDeclare(
  "logs",   // name
  "fanout", // type
  true,     // durable
  false,    // auto-deleted
  false,    // internal
  false,    // no-wait
  nil,      // arguments
)
```

`fanout`（扇出）交换器非常简单。正如你可能从名称中猜测的那样，它只是将接收到的所有消息广播到它知道的所有队列中。而这正是我们记录器所需要的。

> **交换器清单**
>
> 要列出服务器上的交换器，你可以执行有用的rabbitmqctl命令：
>
> ```bash
> sudo rabbitmqctl list_exchanges
> ```
>
> 在此列表中，将有一些`amq.*`交换器和一个默认的（未命名）交换器。这些是默认创建的，但是你现在不太可能需要使用它们。
>
> **默认交换器**
>
> 在本教程的前面部分中，我们还不知道交换器的存在，但仍然能够将消息发送到队列。之所以能这样做，是因为我们使用的是默认交换器，该交换器由空字符串（`""`）标识。
>
> 回想一下我们之前是怎么发布消息的：
>
> ```go
> err = ch.Publish(
>   "",     // exchange
>   q.Name, // routing key
>   false,  // mandatory
>   false,  // immediate
>   amqp.Publishing{
>     ContentType: "text/plain",
>     Body:        []byte(body),
> })
> ```
>
> 在这里，我们使用默认或无名称的交换器：消息将以`route_key`参数指定的名称路由到队列（如果存在）。

现在，我们可以改为发布到我们的命名交换器：

```go
err = ch.ExchangeDeclare(
  "logs",   // 使用命名的交换器
  "fanout", // 交换器类型
  true,     // durable
  false,    // auto-deleted
  false,    // internal
  false,    // no-wait
  nil,      // arguments
)
failOnError(err, "Failed to declare an exchange")

body := bodyFrom(os.Args)
err = ch.Publish(
  "logs", // exchange
  "",     // routing key
  false,  // mandatory
  false,  // immediate
  amqp.Publishing{
          ContentType: "text/plain",
          Body:        []byte(body),
  })
```

### 临时队列

你可能还记得，先前我们使用的是具有特定名称的队列（还记得`hello`和`task_queue`吗？）能够命名队列对我们来说至关重要——我们需要将工作人员指向同一个队列。当你想在生产者和消费者之间共享队列时，给队列一个名称非常重要。

但对于我们的记录器来说，情况并非如此。我们希望收到所有日志消息，而不仅仅是它们的一部分。我们也只对当前正在发送的消息感兴趣，而对旧消息不感兴趣。为了解决这个问题，我们需要两件事。

首先，当我们连接到Rabbit时，我们需要一个新的、空的队列。为此，我们可以创建一个随机名称的队列，或者更好的方法是让服务器为我们选择一个随机队列名称。

其次，一旦我们断开消费者的连接，队列就会自动删除。

在[amqp](http://godoc.org/github.com/streadway/amqp)客户端中，当我们传递一个空字符串作为队列名称时，我们将使用随机生成的名称创建一个非持久队列：

```go
q, err := ch.QueueDeclare(
  "",    // 空字符串作为队列名称
  false, // 非持久队列
  false, // delete when unused
  true,  // 独占队列（当前声明队列的连接关闭后即被删除）
  false, // no-wait
  nil,   // arguments
)
```

上述方法返回时，生成的队列实例包含RabbitMQ生成的随机队列名称。例如，它可能看起来像`amq.gen-JzTY20BRgKO-HjmUJj0wLg`。

当声明它的连接关闭时，该队列将被删除，因为它被声明为独占。

你可以在[队列指南](https://www.rabbitmq.com/queues.html)中了解有关`exclusive`标志和其他队列属性的更多信息。

### 绑定

![img](./assets/bindings.png)

我们已经创建了一个扇出交换器和一个队列。现在我们需要告诉交换器将消息发送到我们的队列。交换器和队列之间的关系称为*绑定*。

```go
err = ch.QueueBind(
  q.Name, // queue name
  "",     // routing key
  "logs", // exchange
  false,
  nil,
)
```

从现在开始，`logs`交换器将会把消息添加到我们的队列中。

> 列出绑定关系
>
> 你猜也猜到了，我们可以使用下面的命令列出绑定关系
>
> ```bash
> rabbitmqctl list_bindings
> ```

### 完整示例

![img](./assets/python-three-overall.png)

产生日志消息的生产程序与上一教程看起来没有太大不同。最重要的变化是我们现在希望将消息发布到`logs`交换器，而不是空的消息交换器。发送时，我们需要提供一个`routingKey`，但是对于`fanout`型交换器，它的值可以被忽略（传空字符串）。下面是`emit_log.go`脚本的代码：

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

（[emit_logs.go源码](http://github.com/rabbitmq/rabbitmq-tutorials/blob/master/go/emit_log.go)）

如你所见，在建立连接之后，我们声明了交换器。此步骤是必需的，因为禁止发布到不存在的交换器。

如果没有队列绑定到交换器，那么消息将丢失，但这对我们来说是ok的。如果没有消费者在接收，我们可以安全地丢弃该消息。

`receive_logs.go`的代码：

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

        q, err := ch.QueueDeclare(
                "",    // name
                false, // durable
                false, // delete when unused
                true,  // exclusive
                false, // no-wait
                nil,   // arguments
        )
        failOnError(err, "Failed to declare a queue")

        err = ch.QueueBind(
                q.Name, // queue name
                "",     // routing key
                "logs", // exchange
                false,
                nil,
        )
        failOnError(err, "Failed to bind a queue")

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

（[receive_logs.go源码](http://github.com/rabbitmq/rabbitmq-tutorials/blob/master/go/receive_logs.go)）

如果要将日志保存到文件，只需打开控制台并输入：

```bash
go run receive_logs.go > logs_from_rabbit.log
```

如果希望在屏幕上查看日志，请切换到一个新的终端并运行：

```bash
go run receive_logs.go
```

当然，要发出日志，请输入：

```bash
go run emit_log.go
```

使用`rabbitmqctl list_bindings`命令，你可以验证代码是否确实根据需要创建了绑定关系和队列。在运行两个`receive_logs.go`程序后，你应该看到类似以下内容：

```bash
sudo rabbitmqctl list_bindings
# => Listing bindings ...
# => logs    exchange        amq.gen-JzTY20BRgKO-HjmUJj0wLg  queue           []
# => logs    exchange        amq.gen-vso0PVvyiRIL2WoV3i48Yg  queue           []
# => ...done.
```

对结果的解释很简单：数据从`logs`交换器进入了两个由服务器分配名称的队列。这正是我们想要的。

要了解如何侦听消息的子集，让我们继续学习[教程4](https://www.liwenzhou.com/posts/Go/go_rabbitmq_tutorials_04/)。



# RabbitMQ Go教程4—路由

## 路由

**（使用Go RabbitMQ客户端）**

在[上一教程](https://www.liwenzhou.com/posts/Go/go_rabbitmq_tutorials_03/)中，我们构建了一个简单的日志记录系统。我们能够向许多接收者广播日志消息。

在本教程中，我们将向它添加一个特性-我们将使它能够只订阅消息的一个子集。例如，我们将只能将关键错误消息定向到日志文件（以节省磁盘空间），同时仍然能够在控制台上打印所有日志消息。

### 绑定

在前面的示例中，我们已经在创建绑定。你可能会想起以下代码：

```go
err = ch.QueueBind(
  q.Name, // queue name
  "",     // routing key
  "logs", // exchange
  false,
  nil)
```

绑定是交换器和队列之间的关系。这可以简单地理解为：队列对来自此交换器的消息感兴趣。

绑定可以采用额外的`routing_key`参数。为了避免与`Channel.Publish`参数混淆，我们将其称为`binding key`。这是我们如何使用键创建绑定的方法：

```go
err = ch.QueueBind(
  q.Name,    // queue name
  "black",   // routing key
  "logs",    // exchange
  false,
  nil)
```

绑定密钥的含义取决于交换器的类型。我们以前使用的`fanout`交换器只是忽略了这个值。

### 直连交换器

我们上一个教程中的日志系统将所有消息广播给所有消费者。我们希望扩展这一点，允许根据消息的严重性过滤消息。例如，我们可能希望将日志消息写入磁盘的脚本只接收严重错误，而不会在warning或info日志消息上浪费磁盘空间。

我们使用`fanout`交换器，这并没有给我们很大的灵活性——它只能进行无脑广播。

我们将使用`direct`交换器。`direct`交换器背后的路由算法很简单——消息进入其`binding key`与消息的`routing key`完全匹配的队列。

为了说明这一点，请考虑以下设置：

![direct-exchang](./assets/direct-exchange.png)

在此设置中，我们可以看到绑定了两个队列的`direct`交换器`X`。第一个队列绑定键为`orange`，第二个队列绑定为两个，一个绑定键为`black`，另一个为`green`。

在这种设置中，使用`orange`路由键发布到交换器的消息将被路由到队列`Q1`。路由键为`black`或`green`的消息将转到`Q2`。所有其他消息将被丢弃。

### 多重绑定

![direct-exchange-multiple](./assets/direct-exchange-multiple.png)

用相同的绑定键绑定多个队列是完全合法的。在我们的示例中，我们可以使用绑定键`black`在`X`和`Q1`之间添加绑定。在这种情况下，`direct`交换器的行为将类似`fanout`，并将消息广播到所有匹配的队列。带有`black`路由键的消息将同时传递给`Q1`和`Q2`。

### 发送日志

我们将在日志系统中使用这个模型。我们将发送消息到`direct`交换器，而不是`fanout`。我们将提供严重性（译注：通常我们使用日志级别划分日志信息的严重性）作为路由键。这样，接收脚本将能够选择其想要接收的日志级别。让我们首先关注发送日志。

与往常一样，我们需要首先创建一个交换器：

```go
err = ch.ExchangeDeclare(
  "logs_direct", // name
  "direct",      // type
  true,          // durable
  false,         // auto-deleted
  false,         // internal
  false,         // no-wait
  nil,           // arguments
)
```

我们已经准备好发送一条消息：

```go
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
```

为了简化问题，我们假设“严重性”可以是“info”、“warning”、“error”之一。

### 订阅

接收消息的工作方式与上一教程一样，但有一个例外——我们将为感兴趣的每种严重性（日志级别）创建一个新的绑定。

```go
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
// 建立多个绑定关系
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
```

### 完整示例

![img](./assets/python-four.png)

`emit_log_direct.go`脚本的代码：

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
                "logs_direct", // name
                "direct",      // type
                true,          // durable
                false,         // auto-deleted
                false,         // internal
                false,         // no-wait
                nil,           // arguments
        )
        failOnError(err, "Failed to declare an exchange")

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

`receive_logs_direct.go`的代码：

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
                "logs_direct", // name
                "direct",      // type
                true,          // durable
                false,         // auto-deleted
                false,         // internal
                false,         // no-wait
                nil,           // arguments
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
                log.Printf("Usage: %s [info] [warning] [error]", os.Args[0])
                os.Exit(0)
        }
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

如果你只想将“warning”和“err”（而不是“info”）级别的日志消息保存到文件中，只需打开控制台并输入：

```bash
go run receive_logs_direct.go warning error > logs_from_rabbit.log
```

如果你想在屏幕上查看所有日志消息，请打开一个新终端并执行以下操作：

```bash
go run receive_logs_direct.go info warning error
# => [*] Waiting for logs. To exit press CTRL+C
```

例如，要发出`error`日志消息，只需输入：

```bash
go run emit_log_direct.go error "Run. Run. Or it will explode."
# => [x] Sent 'error':'Run. Run. Or it will explode.'
```

（这里是（[emit_log_direct.go](https://github.com/rabbitmq/rabbitmq-tutorials/blob/master/go/emit_log_direct.go))和（[receive_logs_direct.go](https://github.com/rabbitmq/rabbitmq-tutorials/blob/master/go/receive_logs_direct.go)）的完整源码）

继续学习[教程5](https://www.liwenzhou.com/posts/Go/go_rabbitmq_tutorials_05/)，了解如何根据模式监听消息。



# RabbitMQ Go教程5—topic

## 先决条件

本教程假设RabbitMQ已安装并运行在本机上的标准端口（5672）。如果你使用不同的主机、端口或凭据，则需要调整连接设置。

## topic交换器（主题交换器）

发送到`topic`交换器的消息不能具有随意的`routing_key`——它必须是单词列表，以点分隔。这些词可以是任何东西，但通常它们指定与消息相关的某些功能。一些有效的`routing_key`示例：“`stock.usd.nyse`”，“`nyse.vmw`”，“`quick.orange.rabbit`”。`routing_key`中可以包含任意多个单词，最多255个字节。

绑定键也必须采用相同的形式。`topic`交换器背后的逻辑类似于`direct`交换器——用特定路由键发送的消息将传递到所有匹配绑定键绑定的队列。但是，绑定键有两个重要的特殊情况：

- `*`（星号）可以代替一个单词。
- `＃`（井号）可以替代零个或多个单词。

通过下面这个示例可以很容易看明白这一点：

![img](./assets/python-five.png)

在这个例子中，我们将发送一些都是描述动物的信息。将使用包含三个词（两个点）的路由密钥发送消息。路由键中的第一个单词将描述速度，第二个是颜色，第三个是种类：“`<speed>.<colour>.<species>`”。

我们创建了三个绑定关系：Q1与绑定键“`*.orange.*`”绑定，Q2与“`*.*.rabbit`”和“`lazy.＃`”绑定。

这些绑定可以总结为：

- Q1对所有橙色动物都感兴趣。
- Q2想接收有关兔子（rabbit）的一切消息，以及有关懒惰（lazy）动物的一切消息。

路由键设置为“`quick.orange.rabbit`”的消息将传递到两个队列。消息“`lazy.orange.elephant`”也将发送给他们两个。另一方面，“`quick.orange.fox`”将仅进入第一个队列，而“`lazy.brown.fox`”将仅进入第二个队列。即使“`lazy.pink.rabbit`”与两个绑定匹配（匹配Q2的两个绑定），也只会传递到第二个队列一次。 “`quick.brown.fox`”与任何绑定都不匹配，因此将被丢弃。

如果我们打破约定并发送一个或四个单词的消息，例如“`orange`”或“`quick.orange.male.rabbit`”，会发生什么？好吧，这些消息将不匹配任何绑定，并且将会丢失。

另外，“`lazy.orange.male.rabbit`”即使有四个单词，也将匹配最后一个绑定，并将其传送到第二个队列。

> topic交换器
>
> topic交换器功能强大，可以像其他交换器一样运行。
>
> 当队列用“`#`”（井号）绑定键绑定时，它将接收所有消息，而与路由键无关，就像在`fanout`交换器中一样。
>
> 当在绑定中不使用特殊字符“`*`”（星号）和“`#`”（井号）时，topic交换器的行为就像`direct`交换器一样。

## 完整示例

我们将在日志记录系统中使用`topic`交换器。我们将从一个可行的假设开始，即日志的路由键将包含两个词：“`<facility>.<severity>`”。

该代码与[上一教程](https://www.liwenzhou.com/posts/Go/go_rabbitmq_tutorials_04/)中的代码几乎相同。

`emit_log_topic.go`的代码：

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

`receive_logs_topic.go`的代码：

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

想要接收所有的日志：

```bash
go run receive_logs_topic.go "#"
```

要从“`kern`”接收所有日志：

```bahs
go run receive_logs_topic.go "kern.*"
```

或者，如果你只想接收“`critical`”日志：

```bash
go run receive_logs_topic.go "*.critical"
```

你可以创建多个绑定：

```bash
go run receive_logs_topic.go "kern.*" "*.critical"
```

并发出带有路由键“`kern.critical`”的日志：

```bash
go run emit_log_topic.go "kern.critical" "A critical kernel error"
```

你可以自己尝试玩一下这个程序。请注意，代码没有对路由键或绑定键进行任何假设，你可能希望使用两个以上的路由键参数。

（关于[emit_log_topic.go](https://github.com/rabbitmq/rabbitmq-tutorials/blob/master/go/emit_log_topic.go)和[receive_logs_topic.go](https://github.com/rabbitmq/rabbitmq-tutorials/blob/master/go/receive_logs_topic.go)的完整源代码）

接下来，我们将在[教程6](https://www.liwenzhou.com/posts/Go/go_rabbitmq_tutorials_06/)中了解如何将往返消息用作远程过程调用。



# RabbitMQ Go教程6—RPC

## 先决条件

本教程假设RabbitMQ已安装并运行在本机上的标准端口（5672）。如果你使用不同的主机、端口或凭据，则需要调整连接设置。

## 远程过程调用（RPC）

（使用Go RabbitMQ客户端）

在第二个教程中，我们学习了如何使用工作队列在多个worker之间分配耗时的任务。

但是，如果我们需要在远程计算机上运行函数并等待结果怎么办？好吧，那是一个不同的故事。这种模式通常称为*远程过程调用*或*RPC*。

在本教程中，我们将使用RabbitMQ构建一个RPC系统：客户端和可伸缩RPC服务器。由于我们没有值得分配的耗时任务，因此我们将创建一个虚拟RPC服务，该服务返回斐波那契数。

> 有关RPC的说明
>
> 尽管RPC是计算中非常常见的模式，但它经常受到批评。
>
> 当程序员不知道函数调用是本地的还是缓慢的RPC时，就会出现问题。这样的混乱会导致系统变幻莫测，并给调试增加了不必要的复杂性。滥用RPC可能会导致无法维护的意大利面条式代码而不是简化软件，
>
> 牢记这一点，请考虑以下建议：
>
> - 确定哪个函数调用是本地的，哪个是远程的。
> - 为你的系统编写文档。明确组件之间的依赖关系。
> - 处理错误情况。 当RPC服务器长时间关闭时，客户端应如何处理？

### 回调队列

通常，通过RabbitMQ进行RPC很容易。客户端发送请求消息，服务器发送响应消息。为了接收响应，我们需要发送带有“回调”队列地址的请求。我们可以使用默认队列。让我们尝试一下：

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

> 消息属性
>
> AMQP 0-9-1协议预定义了消息附带的14个属性集。除以下属性外，大多数属性很少使用：
>
> - `persistent`：将消息标记为持久性（值为`true`）或瞬态（`false`）。你可能还记得第二个教程中的此属性。
> - `content_type`：用于描述编码的mime类型。例如，对于经常使用的JSON编码，将此属性设置为`application/ json`是一个好习惯。
> - `reply_to`：常用于命名回调队列
> - `correlation_id`：有助于将RPC响应与请求相关联

### 关联ID（Correlation Id）

在上面介绍的方法中，我们建议为每个RPC请求创建一个回调队列。这是相当低效的，但是幸运的是，有一种更好的方法——让我们为每个客户端创建一个回调队列。

这就引发了一个新问题，在该队列中收到响应后，尚不清楚响应属于哪个请求。这个时候就该使用`correlation_id`这个属性了。针对每个请求我们将为其设置一个唯一值。随后，当我们在回调队列中收到消息时，我们将查看该属性，并基于这个属性将响应与请求进行匹配。如果我们看到未知的`correlation_id`值，则可以放心地丢弃该消息——它不属于我们的请求。

你可能会问，为什么我们应该忽略回调队列中的未知消息，而不是报错而失败？这是由于服务器端可能出现竞争状况。尽管可能性不大，但RPC服务器可能会在向我们发送答案之后但在发送请求的确认消息之前死亡。如果发生这种情况，重新启动的RPC服务器将再次处理该请求。这就是为什么在客户端上我们必须妥善处理重复的响应，并且理想情况下RPC应该是幂等的。

### 总结

![img](./assets/python-six.png)

我们的RPC工作流程如下：

- 客户端启动时，它将创建一个匿名排他回调队列。
- 对于RPC请求，客户端发送一条消息，该消息具有两个属性：`reply_to`（设置为回调队列）和`correlation_id`（设置为每个请求的唯一值）。
- 该请求被发送到`rpc_queue`队列。
- RPC工作程序（又名：服务器）正在等待该队列上的请求。当出现请求时，它会完成计算工作并把结果作为消息使用`replay_to`字段中的队列发回给客户端。
- 客户端等待回调队列上的数据。出现消息时，它将检查`correlation_id`属性。如果它与请求中的值匹配，则将响应返回给应用程序。

### 完整示例

斐波那契函数：

```go
func fib(n int) int {
        if n == 0 {
                return 0
        } else if n == 1 {
                return 1
        } else {
                return fib(n-1) + fib(n-2)
        }
}
```

声明我们的斐波那契函数。它仅假设有效的正整数输入。 （不要指望这种方法适用于大量用户，它可能是最慢的递归实现）。

我们的RPC服务器[rpc_server.go](https://github.com/rabbitmq/rabbitmq-tutorials/blob/master/go/rpc_server.go)的代码如下所示：

```go
package main

import (
        "log"
        "strconv"

        "github.com/streadway/amqp"
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

        go func() {
                for d := range msgs {
                        n, err := strconv.Atoi(string(d.Body))
                        failOnError(err, "Failed to convert body to integer")

                        log.Printf(" [.] fib(%d)", n)
                        response := fib(n)

                        err = ch.Publish(
                                "",        // exchange
                                d.ReplyTo, // routing key
                                false,     // mandatory
                                false,     // immediate
                                amqp.Publishing{
                                        ContentType:   "text/plain",
                                        CorrelationId: d.CorrelationId,
                                        Body:          []byte(strconv.Itoa(response)),
                                })
                        failOnError(err, "Failed to publish a message")

                        d.Ack(false)
                }
        }()

        log.Printf(" [*] Awaiting RPC requests")
        <-forever
}
```

服务器代码非常简单：

- 与往常一样，我们首先建立连接，通道并声明队列。
- 我们可能要运行多个服务器进程。为了将负载平均分配给多个服务器，我们需要在通道上设置`prefetch`设置。
- 我们使用`Channel.Consume`获取去队列，我们从队列中接收消息。然后，我们进入goroutine进行工作，并将响应发送回去。

我们的RPC客户端[rpc_client.go](https://github.com/rabbitmq/rabbitmq-tutorials/blob/master/go/rpc_client.go)的代码：

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

        corrId := randomString(32)

        err = ch.Publish(
                "",          // exchange
                "rpc_queue", // routing key
                false,       // mandatory
                false,       // immediate
                amqp.Publishing{
                        ContentType:   "text/plain",
                        CorrelationId: corrId,
                        ReplyTo:       q.Name,
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

现在是时候看看[rpc_client.go](https://github.com/rabbitmq/rabbitmq-tutorials/blob/master/go/rpc_client.go)和[rpc_server.go](https://github.com/rabbitmq/rabbitmq-tutorials/blob/master/go/rpc_server.go)的完整示例源代码了。

我们的RPC服务现已准备就绪。我们可以启动服务器：

```bash
go run rpc_server.go
# => [x] Awaiting RPC requests
```

要请求斐波那契数，请运行客户端：

```bash
go run rpc_client.go 30
# => [x] Requesting fib(30)
```

这里介绍的设计不是RPC服务的唯一可能的实现，但是它具有一些重要的优点：

- 如果RPC服务器太慢，则可以通过运行另一台RPC服务器来进行扩展。尝试在新控制台中运行另一个`rpc_server.go`。
- 在客户端，RPC只需要发送和接收一条消息。结果，RPC客户端只需要一个网络往返就可以处理单个RPC请求。

我们的代码仍然非常简单，并且不会尝试解决更复杂（但很重要）的问题，例如：

- 如果没有服务器在运行，客户端应如何反应？
- 客户端是否应该为RPC设置某种超时时间？
- 如果服务器发生故障并引发异常，是否应该将其转发给客户端？
- 在处理之前防止无效的传入消息（例如检查边界，类型）。

> 如果要进行实验，可能会发现[管理后台界面](https://www.rabbitmq.com/management.html)对于查看队列很有用。

