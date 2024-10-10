> RabbitMQ文档整理和代码测试

# RabbitMQ架构介绍

> RabbitMQ是一个开源的消息代理和队列服务器，用来通过普通协议在不同的应用之间共享数据(跨平台跨语言)。RabbitMQ是使用Erlang语言编写，并且基于AMQP协议实现。
>

## 2.1 优势：

可靠性(Reliablity)： 使用了一些机制来保证可靠性，比如持久化、传输确认、发布确认。

灵活的路由(Flexible Routing)： 在消息进入队列之前，通过Exchange来路由消息。对于典型的路由功能，Rabbit已经提供了一些内置的Exchange来实现。针对更复杂的路由功能，可以将多个Exchange绑定在一起，也通过插件机制实现自己的Exchange。

- 高可用(Highly Avaliable Queues)： 队列可以在集群中的机器上进行镜像，使得在部分节点出问题的情况下队列仍然可用。

- 多种协议(Multi-protocol)： 支持多种消息队列协议，如STOMP、MQTT等。

- 多种语言客户端(Many Clients)： 几乎支持所有常用语言，比如Java、.NET、Ruby等。

- 管理界面(Management UI)： 提供了易用的用户界面，使得用户可以监控和管理消息Broker的许多方面。

- 跟踪机制(Tracing)： 如果消息异常，RabbitMQ提供了消息的跟踪机制，使用者可以找出发生了什么。

- 插件机制(Plugin System)：提供了许多插件，来从多方面进行扩展，也可以编辑自己的插件。


## 2.2 整体架构

<img src="./assets/image-20241010150621374.png" alt="image-20241010150621374" style="zoom:50%;" />

## 2.3 消息流转

<img src="./assets/image-20241010163304024.png" alt="image-20241010163304024" style="zoom:50%;" />

## 2.4 组件功能

<img src="./assets/image-20241010150639023.png" alt="image-20241010150639023" style="zoom:50%;" />

**Broker**： 标识消息队列服务器实体.

**Virtual Host**： 虚拟主机。标识一批交换机、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个vhost本质上就是一个mini版的RabbitMQ服务器，拥有自己的队列、交换器、绑定和权限机制。vhost是AMQP概念的基础，必须在链接时指定，RabbitMQ默认的vhost是 /。

**Exchange**：交换器， 用来接收生产者发送的消息并将这些消息路由给服务器中的队列。

**Queue**： 消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。

**Banding**： 绑定，用于消息队列和交换机之间的关联。一个绑定就是基于路由键将交换机和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。

**Channel**： 信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内地虚拟链接，AMQP命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说，建立和销毁TCP都是非常昂贵的开销，所以引入了信道的概念，以复用一条TCP连接。

**Connection**： 网络连接，比如一个TCP连接。

**Publisher**： 消息的生产者，也是一个向交换器发布消息的客户端应用程序。

**Consumer**： 消息的消费者，表示一个从一个消息队列中取得消息的客户端应用程序。

**Message**： 消息，消息是不具名的，它是由消息头和消息体组成。消息体是不透明的，而消息头则是由一系列的可选属性组成，这些属性包括routing-key(路由键)、priority(优先级)、delivery-mode(消息可能需要持久性存储[消息的路由模式])等

# 交换机类型

> Exchange分发消息时，根据类型的不同分发策略有区别。目前共四种类型：direct、fanout、topic、headers(headers匹配AMQP消息的header而不是路由键(Routing-key)，此外headers交换器和direct交换器完全一致，但是性能差了很多，目前几乎用不到了。所以直接看另外三种类型
>

## 3.1 direct 直接模式类型

生产者和消费者，具有相同的交换机名称（Exchange）、交换机类型和相同的路由规则或叫密匙（routingKey），那么消费者即可成功获取到消息。

### 3.1.1 运行流程

<img src="./assets/image-20241010150707366.png" alt="image-20241010150707366" style="zoom: 50%;" />

消息中的路由键(routing_key)如果和Binding中的binding key一致，交换器就将消息发到对应的队列中。路由键与队列名完全匹配。

### 3.1.2 管理界面操作

#### 3.1.2.1 Direct类型交换机

<img src="./assets/image-20241010150741884.png" alt="image-20241010150741884" style="zoom:50%;" />

#### 3.1.2.2 创建队列

<img src="./assets/image-20241010150805940.png" alt="image-20241010150805940" style="zoom:50%;" />

#### 3.1.2.3 路由绑定

通过交换机绑定

<img src="./assets/image-20241010150821381.png" alt="image-20241010150821381" style="zoom:50%;" />

通过队列绑定

<img src="./assets/image-20241010150839366.png" alt="image-20241010150839366" style="zoom:50%;" />

#### 3.1.2.4 查看发送消息

通过路由规则发送消息

<img src="./assets/image-20241010150853305.png" alt="image-20241010150853305" style="zoom:50%;" />

由该交换机发出的消息，在消息队列中直接标注为ack已应答的消息

### 3.1.3 通过代码实现

生产者

```go
func TestExchangeDirectPublish(t *testing.T) {
    conn, err := amqp.Dial("amqp://root:123456@192.168.31.202:5672/")
    if err != nil {
        t.Error(err)
    }
    exchangeName := "exchange.direct"
    queueName := "queue.direct"
    routingKey := "routing.direct"
    defer conn.Close()
    ch, err := conn.Channel()
    handleError(err, t)
    defer ch.Close()
    err = ch.ExchangeDeclare(exchangeName, ExchangeTypeDirect, true, false,
                             false, false, nil)
    handleError(err, t)
    q, err := ch.QueueDeclare(queueName, true, false, false, false, nil)
    handleError(err, t)
    err = ch.QueueBind(q.Name, routingKey, exchangeName, false, nil)
    handleError(err, t)
    err = ch.Publish(exchangeName, routingKey, false, false, amqp.Publishing{
        Type: "application/json",
        Body: []byte("测试direct模式"),
    })
    handleError(err, t)
    t.Logf("发送direct消息完成")
}
```

//订阅消费者

```go
func TestExchangeDirectConsume(t *testing.T) {
    conn, err := amqp.Dial("amqp://root:123456@192.168.31.202:5672/")
    handleError(err, t)
    queueName := "queue.direct"
    forever := make(chan bool)
    defer conn.Close()
    ch, err := conn.Channel()
    handleError(err, t)
    defer ch.Close()
    message, err := ch.Consume(queueName, "queue_direct", true, false, false,
                               false, nil)
    handleError(err, t)
    for msg := range message {
        t.Log(string(msg.Body))
    }
    <-forever
}
```

#### 3.1.3.1 代码执行结果

```go
# 生产者
=== RUN TestExchangeDirectPublish
exchange_direct_test.go:37: 发送direct消息完成
--- PASS: TestExchangeDirectPublish (0.01s)
PASS
# 消费者
=== RUN TestExchangeDirectConsume
exchange_direct_test.go:54: 测试direct模式
exchange_direct_test.go:54: 测试direct模式
exchange_direct_test.go:54: 测试direct模式
```

#### 3.1.3.2 界面效果

交换机

<img src="./assets/image-20241010151011851.png" alt="image-20241010151011851" style="zoom: 67%;" />

队列

<img src="./assets/image-20241010151034652.png" alt="image-20241010151034652" style="zoom: 67%;" />

## 3.2 fanout 广播模式类型

生产者和消费者都不用设置routingKey,所有bind到此exchange的queue都可以接收消息

### 3.2.1 运行流程

<img src="./assets/image-20241010151048155.png" alt="image-20241010151048155" style="zoom:67%;" />

- 每个发到fanout类型交换器的消息都会分到所有绑定的队列上去。
- fanout交换器不处理该路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。
- fanout类型转发消息是最快

### 3.2.2 管理界面实现

#### 3.2.2.1 创建交换机

<img src="./assets/image-20241010151122289.png" alt="image-20241010151122289" style="zoom:67%;" />

#### 3.2.2.2 创建多个队列

<img src="./assets/image-20241010151144081.png" alt="image-20241010151144081" style="zoom:67%;" />

#### 3.2.2.3 绑定交换机

在这种模式下routing_key已经失效了。不同填写

<img src="./assets/image-20241010151201274.png" alt="image-20241010151201274" style="zoom:67%;" />

#### 3.2.2.4 发送查看新消息

<img src="./assets/image-20241010151225052.png" alt="image-20241010151225052" style="zoom:67%;" />

![image-20241010151241959](./assets/image-20241010151241959.png)

<img src="./assets/image-20241010151254859.png" alt="image-20241010151254859" style="zoom:67%;" />

> 该中模式下每个队列都会收到消息

### 3.2.3 代码实现

产生者发送消息

```go
func TestExchangeFanoutPublisher(t *testing.T) {
    exchangeName := "test.fanout"
    queueNames := []string{"test.fanout.queue", "test.fanout.queue1"}
    conn, err := amqp.Dial("amqp://root:123456@192.168.31.202:5672/vhost")
    handlerError(err, t)
    ch, err := conn.Channel()
    handlerError(err, t)
    defer ch.Close()
    err = ch.ExchangeDeclare(exchangeName, ExchangeTypeFanout, true, false,
                             false, false, nil)
    handlerError(err, t)
    for _, name := range queueNames {
        q, err := ch.QueueDeclare(name, true, false, false, false, nil)
        handlerError(err, t)
        err = ch.QueueBind(q.Name, "", exchangeName, false, nil)
        handlerError(err, t)
    }
    body, err := json.Marshal(map[string]interface{}{"name": "laozhu", "age":
                                                     20})
    handlerError(err, t)
    err = ch.Publish(exchangeName, "", false, false, amqp.Publishing{
        Type: "application/json",
        Body: body,
    })
    handlerError(err, t)
    t.Log("广播模式发送完成")
}
```

订阅者 队列test.fanout.queue和test.fanout.queue1

```go
func TestExchangeFanoutConsume(t *testing.T) {
    queueNames := []string{"test.fanout.queue", "test.fanout.queue1"}
    conn, err := amqp.Dial("amqp://root:123456@192.168.31.202:5672/vhost")
    defer conn.Close()
    handlerError(err, t)
    ch, err := conn.Channel()
    handlerError(err, t)
    defer ch.Close()
    wg := sync.WaitGroup{}
    for _, name := range queueNames {
        wg.Add(1)
        consume, err := ch.Consume(name, "", true, false, false, false, nil)
        handlerError(err, t)
        go func(c <-chan amqp.Delivery) {
            defer wg.Done()
            for msg := range consume {
                t.Log("队列"+name, string(msg.Body))
            }
        }(consume)
    }
    wg.Wait()
}
```

实现效果

```go
=== RUN TestExchangeFanoutPublisher
amqp_test.go:99: 广播模式发送完成
--- PASS: TestExchangeFanoutPublisher (0.03s)
PASS
=== RUN TestExchangeFanoutConsume
amqp_test.go:120: 队列test.fanout.queue1 {"age":20,"name":"laozhu"}
amqp_test.go:120: 队列test.fanout.queue {"age":20,"name":"laozhu"}
amqp_test.go:120: 队列test.fanout.queue1 {"age":20,"name":"laozhu"}
amqp_test.go:120: 队列test.fanout.queue {"age":20,"name":"laozhu"}
amqp_test.go:120: 队列test.fanout.queue {"age":20,"name":"laozhu"}
amqp_test.go:120: 队列test.fanout.queue1 {"age":20,"name":"laozhu"}
```

<img src="./assets/image-20241010151434200.png" alt="image-20241010151434200" style="zoom:67%;" />

## 3.3 topic 主题模式类型

topic交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上

### 3.3.1 运行流程

<img src="./assets/image-20241010151448464.png" alt="image-20241010151448464" style="zoom:67%;" />

- 发送到topic类型交换机的消息的routing key不能随便设置–它必须是多个单词组成，用点分割。
- 单词可以是任意的，但它们通常指定连接到该消息的某些功能。
- routing_key最高限制是255字节。

routing key如上图 usa.news,news.weather,europe.news,europe.weather

绑定的关键字必须有相同的格式。topic交换机和direct交换的逻辑是相似的–拥有特定的路由关键字的消息将被发送到所有匹配关键字的队列。然而，绑定关键字有两个特殊的情况

- （1）* (星号) 可以代替一个完整的单词

- （2）# (井号) 可以代替零个或多个单词


### 3.3.2 管理界面操作

#### 3.3.2.1 设置交换机

<img src="./assets/image-20241010151535346.png" alt="image-20241010151535346" style="zoom:67%;" />

#### 3.3.2.2 设置队列

<img src="./assets/image-20241010151552234.png" alt="image-20241010151552234" style="zoom: 50%;" />

<img src="./assets/image-20241010151603413.png" alt="image-20241010151603413" style="zoom: 50%;" />

<img src="./assets/image-20241010151615684.png" alt="image-20241010151615684" style="zoom: 50%;" />

<img src="./assets/image-20241010151628100.png" alt="image-20241010151628100" style="zoom:50%;" />

#### 3.3.2.3 绑定交换机

<img src="./assets/image-20241010151653719.png" alt="image-20241010151653719" style="zoom:50%;" />

#### 3.3.2.4 发送消息查看结果

<img src="./assets/image-20241010151714866.png" alt="image-20241010151714866" style="zoom:50%;" />

<img src="./assets/image-20241010151726426.png" alt="image-20241010151726426" style="zoom:50%;" />

<img src="./assets/image-20241010151737093.png" alt="image-20241010151737093" style="zoom:50%;" />

<img src="./assets/image-20241010151749768.png" alt="image-20241010151749768" style="zoom:50%;" />

### 3.3.3 代码实现

#### 3.3.3.1 生产者代码

```go
func TestExchangeTopicPublisher(t *testing.T) {
    exchangeName := "test.topic"
    queueNames := []string{"europe.news", "europe.weather", "usa.news",
                           "usa.weather"}
    routingKeys := []string{"europe.*", "*.weather", "*.news", "usa.*"}
    conn, err := amqp.Dial("amqp://root:123456@192.168.31.202:5672/vhost")
    handlerError(err, t)
    ch, err := conn.Channel()
    handlerError(err, t)
    defer ch.Close()
    err = ch.ExchangeDeclare(exchangeName, ExchangeTypeTopic, true, false,
                             false, false, nil)
    handlerError(err, t)
    list := make([]amqp.Queue, 0)
    for _, name := range queueNames {
        q, err := ch.QueueDeclare(name, true, false, false, false, nil)
        handlerError(err, t)
        list = append(list, q)
    }
    // europe.news > europe.*, *.news
    err = ch.QueueBind(list[0].Name, routingKeys[0], exchangeName, false, nil)
    err = ch.QueueBind(list[0].Name, routingKeys[2], exchangeName, false, nil)
    // europe.weather > europe.*, *.weather
    err = ch.QueueBind(list[1].Name, routingKeys[0], exchangeName, false, nil)
    err = ch.QueueBind(list[1].Name, routingKeys[1], exchangeName, false, nil)
    // usa.news > usa.*, *.news
    err = ch.QueueBind(list[2].Name, routingKeys[2], exchangeName, false, nil)
    err = ch.QueueBind(list[2].Name, routingKeys[3], exchangeName, false, nil)
    // usa.weather > usa.*, *.weather
    err = ch.QueueBind(list[3].Name, routingKeys[3], exchangeName, false, nil)
    err = ch.QueueBind(list[3].Name, routingKeys[1], exchangeName, false, nil)
    handlerError(err, t)
    // 发送针对于*.news主题消息
    err = ch.Publish(exchangeName, "usa.*", false, false, amqp.Publishing{
        Type: "text/explain",
        Body: []byte("发送usa.*主题的消息"),
    })
    handlerError(err, t)
    t.Log("主题模式发送")
}
```

#### 3.3.3.2 消费者代码

```go
func TestExchangeTopicConsume(t *testing.T) {
    queueNames := []string{"europe.news", "europe.weather", "usa.news",
                           "usa.weather"}
    conn, err := amqp.Dial("amqp://root:123456@192.168.31.202:5672/vhost")
    handlerError(err, t)
    ch, err := conn.Channel()
    handlerError(err, t)
    defer ch.Close()
    wg := sync.WaitGroup{}
    for _, name := range queueNames {
        wg.Add(1)
        consume, err := ch.Consume(name, "", true, false, false, false, nil)
        handlerError(err, t)
        go func(c <-chan amqp.Delivery) {
            defer wg.Done()
            for msg := range consume {
                t.Log("队列"+name+"消息内容：", string(msg.Body))
            }
        }(consume)
    }
    wg.Wait()
}
```

# 深入话题

## 4.1 默认交换机作用

RabbitMQ默认提供了一下几种交换机,AMQP default为默认交换机

<img src="./assets/image-20241010151852425.png" alt="image-20241010151852425" style="zoom:50%;" />

### 4.1.2 默认交换机消息

默认交换机，如果消息在信道中发送时未指定交换机，那么系统就会使用””AMQP default”这个exchange，我们创建一个queue时,默认的都会有一个和新建queue同名的routingKey绑定到这个默认的exchange上去。

通过操作界面测试一下：

#### 4.1.2.1 创建队列

<img src="./assets/image-20241010151918574.png" alt="image-20241010151918574" style="zoom:50%;" />

#### 4.1.2.2 直接通过该队列发送消息

<img src="./assets/image-20241010151935606.png" alt="image-20241010151935606" style="zoom:50%;" />

#### 4.1.2.3 查看发送结果

<img src="./assets/image-20241010151954472.png" alt="image-20241010151954472" style="zoom:50%;" />

### 4.1.3 代码实现

```go
func TestNoExchangeQueue(t *testing.T) {
    conn, err := amqp.Dial("amqp://root:123456@192.168.31.202:5672/vhost")
    if err != nil {
        t.Error(err)
    }
    queueName := "default.queue"
    defer conn.Close()
    ch, err := conn.Channel()
    handlerError(err, t)
    defer ch.Close()
    err = ch.Publish("", queueName, false, false, amqp.Publishing{
        Type: "application/json",
        Body: []byte("使用默认交换机进行发送消息"),
    })
    handlerError(err, t)
}
func TestNoExchangeConsume(t *testing.T) {
    conn, err := amqp.Dial("amqp://root:123456@192.168.31.202:5672/vhost")
    if err != nil {
        t.Error(err)
    }
    queueName := "default.queue"
    defer conn.Close()
    ch, err := conn.Channel()
    handlerError(err, t)
    defer ch.Close()
    consume, err := ch.Consume(queueName, "", true, false, false, false, nil)
    handlerError(err, t)
    for msg := range consume {
        t.Log(string(msg.Body))
    }
}
```

#### 4.1.3.1 实现效果

```go
=== RUN TestNoExchangeQueue
--- PASS: TestNoExchangeQueue (0.01s)
PASS
=== RUN TestNoExchangeConsume
amqp_test.go:237: 测试发送默认交换机
amqp_test.go:237: 使用默认交换机进行发送消息
amqp_test.go:237: 使用默认交换机进行发送消息
```

## 4.2 消息的TTL问题

RabbitMQ可以对消息和队列设置消息的生存时间TTL.

目前有两种方法可以设置。

- 第一种方法是通过队列属性设置，队列中所有消息都有相同的过期时间。

- 第二种方法是对消息进行单独设置，每条消息TTL可以不同。


如果上述两种方法同时使用，则消息的过期时间以两者之间TTL较小的那个数值为准。

消息在队列的生存时间一旦超过设置的TTL值，就称为dead message， 消费者将无法再收到该消息。

### 4.2.1 管理界面设置

#### 4.2.1.1 设置队列

设置2个队列，一个队列设置了ttl，一个队列未设置

<img src="./assets/image-20241010152046610.png" alt="image-20241010152046610" style="zoom:50%;" />

设置消息在生产者发出，在队列的生存时间为10秒x-message-ttl单位为毫秒

<img src="./assets/image-20241010152106554.png" alt="image-20241010152106554" style="zoom:50%;" />

#### 4.2.1.2 设置交换机

设置交换机的通信模式为topic

<img src="./assets/image-20241010152131470.png" alt="image-20241010152131470" style="zoom:50%;" />

绑定创建的队列到交换机

<img src="./assets/image-20241010152158712.png" alt="image-20241010152158712" style="zoom:50%;" />

#### 4.2.1.3 发送消息和查看效果

<img src="./assets/image-20241010152213902.png" alt="image-20241010152213902" style="zoom:50%;" />

<img src="./assets/image-20241010152224911.png" alt="image-20241010152224911" style="zoom:50%;" />

效果如下：

<img src="./assets/image-20241010152240737.png" alt="image-20241010152240737" style="zoom:50%;" />

查看test.ttl.queue和test.ttl.queue1中都有消息

<img src="./assets/image-20241010152255831.png" alt="image-20241010152255831" style="zoom:50%;" />

看到test.ttl.queue 10秒后，消息已经自动处理了。

### 4.2.2 代码实现

```go
func TestHeaderTTl(t *testing.T) {
    conn, err := amqp.Dial("amqp://root:123456@192.168.31.202:5672/vhost")
    if err != nil {
        t.Error(err)
    }
    exchangeName := "test.ttl"
    queueName := []string{"test.ttl.queue", "test.ttl.queue1"}
    routingKey := "test.ttl.*"
    defer conn.Close()
    ch, err := conn.Channel()
    handlerError(err, t)
    defer ch.Close()
    err = ch.ExchangeDeclare(exchangeName, ExchangeTypeDirect, true, false,
                             false, false, nil)
    handlerError(err, t)
    q, err := ch.QueueDeclare(queueName[0], true, false, false, false,
                              amqp.Table{
                                  "x-message-ttl": 10000,
                              })
    handlerError(err, t)
    err = ch.QueueBind(q.Name, routingKey, exchangeName, false, nil)
    handlerError(err, t)
    q1, err := ch.QueueDeclare(queueName[1], true, false, false, false, nil)
    handlerError(err, t)
    err = ch.QueueBind(q1.Name, routingKey, exchangeName, false, nil)
    handlerError(err, t)
    err = ch.Publish(exchangeName, routingKey, false, false, amqp.Publishing{
        Expiration: "30000", // 过期时间
        Type: "application/json",
        Body: []byte("测试direct模式"),
    })
    handlerError(err, t)
}
```

多次执行后，查看设置消息存活时间和过期时间的问题

#### 4.2.2.1 查看效果

<img src="./assets/image-20241010152340034.png" alt="image-20241010152340034" style="zoom:50%;" />

## 4.3 DLX 死信队列

> 死信队列(DLX Dead-Letter-Exchange)：没有及时被消费的消息存放的队列。
>
> 利用DLX，当消息在一个队列中变成死信(dead message)之后，它能被重新publish到另一个Exchange，这个Exchange就是DLX。

消息没有被及时消费的原因：

- a.消息被拒绝（basic.reject/ basic.nack）并且不再重新投递 requeue=false
- b.TTL(time-to-live) 消息超时未消费
- c.达到最大队列长度

### 4.3.1 管理界面操作

#### 4.3.1.1 设置交换机

设置2个交换机 test.dlx.work为正常业务交换机，test.dlx.delay为处理死信交换机

<img src="./assets/image-20241010152415792.png" alt="image-20241010152415792" style="zoom:50%;" />

<img src="./assets/image-20241010152429785.png" alt="image-20241010152429785" style="zoom:50%;" />

#### 4.3.1.2 设置队列

设置2个队列

test.dlx.delay.queue为死信队列

<img src="./assets/image-20241010152445835.png" alt="image-20241010152445835" style="zoom:50%;" />

- test.dlx.work.queue为业务队列

业务队列要设置参数，dead message后能requeue(重新排队)，到死信队列。

参数如下:

```go
x-message-ttl:30000 //消息队列中的生性周期30秒，超过30秒处理
x-dead-letter-exchange:test.dlx.delay //消息重投死信交换机
x-dead-letter-routing-key:test.dlx.delay.queue //消息重投的binding key
```

<img src="./assets/image-20241010152515994.png" alt="image-20241010152515994" style="zoom:50%;" />

交换机和队列绑定

1、死信交换机绑定死信队列，routing key是队列名称

<img src="./assets/image-20241010152530923.png" alt="image-20241010152530923" style="zoom:50%;" />

2、业务交换机绑定业务队列，routing key是队列名称

<img src="./assets/image-20241010152615415.png" alt="image-20241010152615415" style="zoom:50%;" />

#### 4.3.1.3 发送消息和查看结果

发送业务消息

<img src="./assets/image-20241010152801782.png" alt="image-20241010152801782" style="zoom:50%;" />

查看接收队列

未超过30秒，业务队列已经获取了该消息，我们在test.dlx.work.queue中查看到了有一个Ready状态的消息

<img src="./assets/image-20241010152835431.png" alt="image-20241010152835431" style="zoom:50%;" />

查看死信队列

超过30秒，该消息直接投递到test.dlx.delay和test.dlx.delay.queue交换机和消息队列

<img src="./assets/image-20241010152852912.png" alt="image-20241010152852912" style="zoom:50%;" />

### 4.3.2 代码实现

```go
func TestDxlDelivery(t *testing.T) {
    conn, err := amqp.Dial("amqp://root:123456@192.168.31.202:5672/vhost")
    handlerError(err, t)
    defer conn.Close()
    ch, err := conn.Channel()
    handlerError(err, t)
    defer ch.Close()
    ttl := 1000 * 30 //消息在业务队列中的存活时间
    workExchangeName := "test.dlx.work" //业务交换机
    workQueueName := "test.dlx.work.queue" //业务队列
    delayExchangeName := "test.dlx.delay" //死信交换机
    delayQueueName := "test.dlx.delay.queue" //死信队列，接收处理死信消息
    // 死信交换机设置,队列，路由表
    err = ch.ExchangeDeclare(delayExchangeName, ExchangeTypeDirect, true, false,
                             false, false, nil)
    handlerError(err, t)
    q, err := ch.QueueDeclare(delayQueueName, true, false, false, false, nil)
    handlerError(err, t)
    err = ch.QueueBind(q.Name, q.Name, delayExchangeName, false, nil)
    handlerError(err, t)
    // 业务交换机、队列、路由表设置
    err = ch.ExchangeDeclare(workExchangeName, ExchangeTypeDirect, true, false,
                             false, false, nil)
    handlerError(err, t)
    // 工作队列设置参数
    args := amqp.Table{
        "x-message-ttl": ttl, //消息存活时间
        "x-dead-letter-exchange": delayExchangeName, //死信交换机
        "x-dead-letter-routing-key": delayQueueName, //死信队列
    }
    q1, err := ch.QueueDeclare(workQueueName, true, false, false, false, args)
    handlerError(err, t)
    err = ch.QueueBind(q1.Name, q1.Name, workExchangeName, false, nil)
    handlerError(err, t)
    //100个订单信息，每个订单超时时间都是10s
    for i := 0; i < 30; i++ {
        err := ch.Publish(workExchangeName, workQueueName, false, false,
                          amqp.Publishing{
                              Type: "text/plain",
                              Body: []byte("发送订单，订单编号:" + strconv.Itoa(i)),
                          })
        handlerError(err, t)
        time.Sleep(1 * time.Second)
    }
}
```

```go
func TestReceiveDlxMessage(t *testing.T) {
    conn, err := amqp.Dial("amqp://root:123456@192.168.31.202:5672/vhost")
    handlerError(err, t)
    defer conn.Close()
    ch, err := conn.Channel()
    handlerError(err, t)
    defer ch.Close()
    consume, err := ch.Consume("test.dlx.work.queue", "", false, false, false,
                               false, nil)
    for msg := range consume {
        t.Log(string(msg.Body))
        msg.Ack(false)
    }
}
```

#### 4.3.2.1 查看效果

<img src="./assets/image-20241010152941583.png" alt="image-20241010152941583" style="zoom:50%;" />

## 4.4 消息在channel中的状态变化

我们在消息队列列表中，如上图中Message列中的Ready Unacked Total等。

1、 Ready状态：消息生产者已经发送了消息，但没有消费者消费，此时消息状态为Ready

2、 unacked状态：消费者接收消息未应答，则消息状态为unacked

注：如果消费者端断开连接消息又变成了Ready状态

代码测试

```go
func TestDirectNoAsk(t *testing.T) {
    conn, err := amqp.Dial("amqp://root:123456@192.168.31.202:5672/vhost")
    if err != nil {
        t.Error(err)
    }
    exchangeName := "test.direct"
    queueName := "test.direct.queue"
    defer conn.Close()
    ch, err := conn.Channel()
    handlerError(err, t)
    defer ch.Close()
    err = ch.ExchangeDeclare(exchangeName, ExchangeTypeDirect, true, false,
                             false, false, nil)
    handlerError(err, t)
    q, err := ch.QueueDeclare(queueName, true, false, false, false, nil)
    handlerError(err, t)
    err = ch.QueueBind(q.Name, q.Name, exchangeName, false, nil)
    handlerError(err, t)
    // 发送10条记录
    for i := 0; i < 10; i++ {
        err := ch.Publish(exchangeName, q.Name, false, false, amqp.Publishing{
            Body: []byte("test direct queue"),
        })
        handlerError(err, t)
    }
}
```

产生10条Ready消息

<img src="./assets/image-20241010153022433.png" alt="image-20241010153022433" style="zoom:67%;" />

启动消费者

```go
func TestNoAskConsume(t *testing.T) {
    conn, err := amqp.Dial("amqp://root:123456@192.168.31.202:5672/vhost")
    handlerError(err, t)
    queueName := "test.direct.queue"
    defer conn.Close()
    ch, err := conn.Channel()
    handlerError(err, t)
    defer ch.Close()
    consume, err := ch.Consume(queueName, "", false, false, false, false, nil)
    handlerError(err, t)
    for d := range consume {
        t.Log(string(d.Body))
    }
}
```

![image-20241010153102395](./assets/image-20241010153102395.png)

由于消费者接收消息后，未做ACK处理，所有的消息都变成了 unacked 状态

切断执行消费者，消息状态又变成了Ready,如果设置了手动应答，消息被消费，应答处理后消息被删除。

### 4.4.1 消息消费处理方式

1) ACK

在消费者端通过下面代码做手动应答，这样做的目的是正确消费后给回馈，说明我正确消费了。

这时队列删除了消息。如果消费者接收消息未做应答，消息状态为unacked，占据队列空间，空间满了，就会出现不能被消费的情况。

```go
ch.Ack(d.DeliveryTag, false)
// 同上
d.Ask(false)
```

2) Nack

在消费中出现了异常，这时候我们就不能返回应答了，这时候就使用nack了。

- 直接抛弃此消息，该消息从队列删除

```go
// 第二个参数为true表示所有channel中的消息都做消费处理
// 第三个参数Requeue消息是否重新进入队列
ch.Nack(d.DeliveryTag, false, false)
```

- 消息重回队列,第三个参数Requeue设置为true

```go
ch.Nack(d.DeliveryTag, false, true)
// 同上
d.Nack(false,true)
```

3) Reject

如果消息出现异常也可以用Reject函数处理,第二个参数Requeue设置为true标识重投队列。

nack一次可以处理多个消息，而reject一次仅能处理一条

```go
d.Reject(false)
// 同上
ch.Reject(d.DeliveryTag, false)
```

## 4.5 消息的推模式和拉模式(push/pull)

> 实现RabbitMQ的消费者有两种模式，推模式（Push）和拉模式（Pull）。

- 推模式：消息中间件主动将消息推送给消费者
- 拉模式：消费者主动从消息中间件拉取消息

**推模式** 将消息提前推送给消费者，消费者必须设置一个缓冲区缓存这些消息。好处很明显，消费者总是有一堆在内存中待处理的消息，所以效率高。缺点是缓冲区可能会溢出。

**拉模式** 在消费者需要时才去消息中间件拉取消息，这段网络开销会明显增加消息延迟，降低系统吞吐量。

选择推模式还是拉模式需要考虑使用场景。

**推模式是最常用的**，但是有些情况下推模式并不适用的，比如说：由于某些限制，消费者在某个条件成立时才能消费消息



**接受消息 – 推模式**

RMQ Server主动把消息推给消费者

```go
func (ch *Channel) Consume(queue, consumer string, autoAck, exclusive, noLocal,
noWait bool, args Table) (<- chan Delivery, error)
```

- queue:队列名称。
- consumer:消费者标签，用于区分不同的消费者。
- autoAck:是否自动回复ACK，true为是，回复ACK表示高速服务器我收到消息了。建议为false，手
- 动回复，这样可控性强。
- exclusive:设置是否排他，排他表示当前队列只能给一个消费者使用。
- noLocal:如果为true，表示生产者和消费者不能是同一个connect。
- nowait：是否非阻塞，true表示是。阻塞：表示创建交换器的请求发送后，阻塞等待RMQ Server
- 返回信息。非阻塞：不会阻塞等待RMQ Server的返回信息，而RMQ Server也不会返回信息。（不
- 推荐使用）
- args：直接写nil

注意下返回值：返回一个<- chan amqp.Delivery类型，遍历返回值，有消息则往下走， 没有则阻塞。



**接受消息 – 拉模式**

消费者主动从RMQ Server拉消息

```go
func (ch *Channel) Get(queue string, autoAck bool) (msg Delivery, ok bool, err
error)
```

- queue：队列名称 。
- autoAck：是否开启自动回复。

### 4.5.1 消费者消息确认机制（关于Ack机制）

如果其中一个消费者开始一项漫长的任务而仅部分完成就挂掉，会发生什么情况？

在这种情况下，消费者挂掉，我们将丢失正在处理的消息。我们还将丢失所有发送给该特定消费者但尚未处理的消息。但是我们不想丢失任何消息。如果一个消费者挂掉，我们希望将消息交付给另一个消费者。

为了确保消息永不丢失，RabbitMQ支持 消息*确认*。消费者发送回一个确认（ACK），告知RabbitMQ特定的消息已被接收处理，并且RabbitMQ才可以删除此条消息。

在消费者从队列接收消息时，可以设置 autoAck 来指定是否自动确认。

- autoAck = true，当消费者收到消息后，会自动发送ACK，RabbitMQ然后从内存或磁盘中删除消息。
- autoAck = false，RabbitMQ 会一直等到持有消息的消费者显式地调用 Ack 命令为止。而且RabbitMQ 也不会为消息设置过期时间，判断消息是否需要重新投递的依据是消费该消息的消费者已断开连接。这时候，队列中的消息分为两个部分：一部分是等待投递给消费者的消息；一部分是已经投递，但还没收到消费者确认信号的消息。如果一直没有受到确认信号，并且消费消息的消费者断开连接，RabbitMQ 会重新安排该消息进入队列，等待投递给下一个消费者（有可能还是原来的那个消费者）。
- 推荐手动回复，尽量不要使用autoACK，因autoACK不可控。

如果设置为false后，主要使用两个函数进行手动回复：

```go
func (ch *Channel) Ack(tag uint64, multiple bool) error
func (me Delivery) Ack(multiple bool) error {
    if me.Acknowledger == nil {
        return errDeliveryNotInitialized
    }
    return me.Acknowledger.Ack(me.DeliveryTag, multiple)
}
```

简单看一眼，函数2调用了函数1，本质上两个函数没区别。 这里推荐使用第二个，因为方便。

另外说一下multiple参数。true表示回复当前信道所有未回复的ack，用于批量确认。false表示回复当前条目。

手动回复的例子

```go
func TestAsk(t *testing.T) {
    conn, err := amqp.Dial("amqp://root:123456@192.168.31.202:5672/vhost")
    handlerError(err, t)
    queueName := "queue.direct"
    forever := make(chan bool)
    defer conn.Close()
    ch, err := conn.Channel()
    handlerError(err, t)
    defer ch.Close()
    message, err := ch.Consume(
        queueName, // 队列名称
        "", // consumer
        false, // auto-ack
        false, // exclusive
        false, // no-local
        false, // no-wait
        nil, // args
    )
    handlerError(err, t)
    for msg := range message {
        defer msg.Ack(false)
        t.Log(string(msg.Body))
    }
    <-forever
}
```

ACK机制的开发注意事项

如果你设置为手动回复，而且在程序中忘记了ACK，那么后果很严重。当消费者退出时候，Message会一直重新分发。然后RabbitMQ会占用越来越多的内容，由于RabbitMQ会长时间运行，因此这个"内存泄漏"是致命的。

### 4.5.2 消费者拒绝消费

```go
func (d Delivery) Reject(requeue bool) error
```

拒绝本条消息。如果requeue为true，则RMQ会把这条消息重新加入队列，如果requeue为false，则RMQ会丢弃本条消息。

## 4.6 生产者事务机制

在使用RabbitMQ的时候，我们可以通过消息持久化操作来解决因为服务器的异常奔溃导致的消息丢失，除此之外我们还会遇到一个问题，当消息的发布者在将消息发送出去之后，消息到底有没有正确到达broker代理服务器呢？

如果不进行特殊配置的话，默认情况下发布操作是不会返回任何信息给生产者的，也就是默认情况下我们的生产者是不知道消息有没有正确到达broker的，如果在消息到达broker之前已经丢失的话，持久化操作也解决不了这个问题，因为消息根本就没到达代理服务器，你怎么进行持久化，那么这个问题该怎么解决呢？

RabbitMQ为我们提供了两种方式：

- 通过AMQP事务机制实现，这也是AMQP协议层面提供的解决方案；
- 通过将channel设置成confirm模式来实现；

### 4.6.1 事务机制介绍

> RabbitMQ中与事务机制有关的方法有三个：txSelect(), txCommit()以及txRollback()

txSelect用于将当前channel设置成transaction模式，txCommit用于提交事务，txRollback用于回滚事务

在通过txSelect开启事务之后，我们便可以发布消息给broker代理服务器了

- 如果txCommit提交成功了，则消息一定到达了broker
- 如果在txCommit执行之前broker异常崩溃或者由于其他原因抛出异常，这个时候我们便可以捕获异常通过txRollback回滚事务了。

### 4.6.2 代码实现

```go
func TestMessageTran(t *testing.T) {
    conn, err := amqp.Dial("amqp://root:123456@192.168.31.202:5672/vhost")
    handlerError(err, t)
    defer conn.Close()
    ch, err := conn.Channel()
    handlerError(err, t)
    defer ch.Close()
    // 开启事务处理
    err = ch.Tx()
    handlerError(err, t)
    queueName := "test.trans.queue"
    q, err := ch.QueueDeclare(queueName, true, false, false, false, nil)
    handlerError(err, t)
    // 发送30个消息
    for i := 0; i < 30; i++ {
        err := ch.Publish("", q.Name, false, false, amqp.Publishing{
            DeliveryMode: amqp.Persistent,
            ContentType: "text/plain",
            Body: []byte("发送消息内容"),
        })
        time.Sleep(1 * time.Second)
        // 回滚
        if err != nil {
            ch.TxRollback()
        }
    }
    // 提交
    ch.TxCommit()
}
```

## 4.7 生产者确认机制

上面我们介绍了RabbitMQ可能会遇到的一个问题，即生成者不知道消息是否真正到达broker，随后通过AMQP协议层面为我们提供了事务机制解决了这个问题，但是采用事务机制实现会降低RabbitMQ的消息吞吐量，那么有没有更加高效的解决方式呢？答案是采用Confirm模式。

- 消息的确认，是指生产者投递消息后，如果Broker收到消息，则会给我们生产者一个应答。
- 生产者进行接受应答，用来确认这条消息是否正常的发送到了Broker，这种方式也是消息的可靠性投递的核心保障

<img src="./assets/image-20241010153737904.png" alt="image-20241010153737904" style="zoom:50%;" />

**生产者确认模式实现原理：**

生产者将信道设置成confirm模式，一旦信道进入confirm模式，所有在该信道上面发布的消息都将会被指派一个唯一的ID(从1开始)，一旦消息被投递到所有匹配的队列之后，broker就会发送一个确认给生产者(包含消息的唯一ID)，这就使得生产者知道消息已经正确到达目的队列了，如果消息和队列是可持久化的，那么确认消息会在将消息写入磁盘之后发出，broker回传给生产者的确认消息中delivery-tag域包含了确认消息的序列号，此外broker也可以设置ack的multiple域，表示到这个序列号之前的所有消息都已经得到了处理；

confirm模式最大的好处在于他是异步的，一旦发布一条消息，生产者应用程序就可以在等信道返回确认的同时继续发送下一条消息，当消息最终得到确认之后，生产者应用便可以通过回调方法来处理该确认消息，如果RabbitMQ因为自身内部错误导致消息丢失，就会发送一条nack消息，生产者应用程序同样可以在回调方法中处理该nack消息；

**开启confirm模式的方法：**

生产者通过调用channel的confirmSelect方法将channel设置为confirm模式，(注意一点，已经在transaction事务模式的channel是不能再设置成confirm模式的，即这两种模式是不能共存的)，如果没有设置no-wait标志的话，broker会返回confirm.select-ok表示同意发送者将当前channel信道设置为confirm模式(从目前RabbitMQ最新版本3.6来看，如果调用了channel.confirmSelect方法，默认情况下是直接将no-wait设置成false的，也就是默认情况下broker是必须回传confirm.select-ok的，而且我也没找到我们自己能够设置no-wait标志的方法)；

**生产者实现confiem模式有三种编程方式：**

- 普通confirm模式，每发送一条消息，等待服务端confirm，这实际上是一种串行的confirm，每publish一条消息之后就等待服务端confirm，如果服务端返回false或者超时时间内未返回，客户端进行消息重传；

- 批量confirm模式，每发送一批消息之后，设置mandatory为true，等待服务端confirm，这种批量确认的模式极大的提高了confirm效率，但是如果一旦出现confirm返回false或者超时的情况，客户端需要将这一批次的消息全部重发，这会带来明显的重复消息，如果这种情况频繁发生的话，效率也会不升反降；


**mandatory**

当mandatory标志位设置为true时，如果exchange根据自身类型和消息routeKey无法找到一个符合条件的queue，那么会调用basic.return方法将消息返回给生产者（Basic.Return + Content-Header +Content-Body）；当mandatory设置为false时，出现上述情形broker会直接将消息扔掉。

**immediate**

在RabbitMQ3.0版本之后，去掉了该功能，如果执行会报错,去掉原因immediate标记会影响镜像队列性能，增加代码复杂性，并建议采用“TTL”和“DLX”等方式替代当immediate标志位设置为true时，如果exchange在将消息路由到queue(s)时发现对于的queue上么有消费者，那么这条消息不会放入队列中。当与消息routeKey关联的所有queue（一个或者多个）都没有消费者时，该消息会通过basic.return方法返还给生产者。

概括来说，mandatory标志告诉服务器至少将该消息route到一个队列中，否则将消息返还给生产者；immediate标志告诉服务器如果该消息关联的queue上有消费者，则马上将消息投递给它，如果所有queue都没有消费者，直接把消息返还给生产者，不用将消息入队列等待消费者了。

关于mandatory设置true，监听返回

```go
err = ch.Publish(
    "", // exchange
    "", // routing key
    true, // mandatory
    true, // immediate
    amqp.Publishing{
        ContentType: "text/plain",
        Body: []byte(body),
        DeliveryMode: amqp.Persistent, // 1=non-persistent, 2=persistent
    })
handlerError(err, t)
ret := ch.NotifyReturn(make(chan amqp.Return, 1))
cancel := ch.NotifyCancel(make(chan string))
select {
    case val := <-ret:
    t.Log("返回结果", val)
    case cal := <-cancel:
    t.Log(cal)
    case <-time.After(10 * time.Second):
    fmt.Println("Timed out waiting for confirmation")
}
```

```go
func TestConfirm(t *testing.T) {
    conn, err := amqp.Dial("amqp://root:123456@192.168.31.202:5672/vhost")
    handlerError(err, t)
    defer conn.Close()
    ch, err := conn.Channel()
    handlerError(err, t)
    defer ch.Close()
    // 启动确认机制
    ch.Confirm(false)
    // 发送消息
    err = ch.Publish("test.direct", "test.direct.queue", false, false,
                     amqp.Publishing{
                         DeliveryMode: amqp.Persistent,
                         ContentType: "text/plain",
                         Body: []byte("确认机制，发送消息" + strconv.Itoa(i)),
                     })
    handlerError(err, t)
    // 确认ack或nack
    ack := make(chan uint64)
    nack := make(chan uint64)
    ackRet, nackRet := ch.NotifyConfirm(ack, nack)
    select {
        case val := <-ackRet:
        fmt.Printf("ack return i is %d\n", val)
        case <-nackRet:
        fmt.Println("nack")
    }
}
```

# 实战训练

## 5.1 处理消息幂等

### 5.1.1 什么是消息幂等

在数学与计算机学中，幂等操作的特点是其任意多次执行所产生的影响均与一次执行的影响相同。

在消息领域，幂等是指Consumer重复消费某条消息时，重复消费的结果与消费一次的结果是相同的，并且多次消费并未对业务系统产生任何负面影响。

例如，在支付场景下，Consumer消费扣款消息，对一笔订单执行扣款操作，扣款金额为100元。如果因网络不稳定等原因导致扣款消息重复投递，Consumer重复消费了该扣款消息，但最终的业务结果是只扣款一次，扣费100元，且用户的扣款记录中对应的订单只有一条扣款流水，不会多次扣除费用。那么这次扣款操作是符合要求的，整个消费过程实现了消息幂等。

### 5.1.2 适用场景

在互联网应用中，尤其在网络不稳定的情况下， RabbitMQ 消息中间件的消息有可能会出现重复。如果消息重复消费会影响您的业务处理，请对消息做幂等处理。消息重复的可能原因如下：

- 发送时消息重复

当一条消息已被成功发送到服务端并完成持久化，此时出现了网络闪断或者客户端宕机，导致服务端对客户端应答失败。 如果此时Producer意识到消息发送失败并尝试再次发送消息，Consumer后续会收到两条内容相同并且Message ID也相同的消息。

- 投递时消息重复

消息消费的场景下，消息已投递到Consumer并完成业务处理，当客户端给服务端反馈应答的时候网络闪断。为了保证消息至少被消费一次，服务端将在网络恢复后再次尝试投递之前已被处理过的消息，Consumer后续会收到两条内容相同并且Message ID也相同的消息。

- 负载均衡时消息重复（包括但不限于网络抖动、服务端重启以及Consumer应用重启）

服务端或客户端重启、扩容或缩容时，会触发Rebalance，此时Consumer可能会收到重复消息。

### 5.1.3 处理方式

在投递消息中设置全局的MessageId,消费者端获取投递的消息，并在本次cache或者db中存储已MessageId。如果已处理过直接Reject掉。

- 通过数据库记录已消费的消息记录，验证数据库记录是否存在。劣势：高并发，io操作频繁，可考虑存在数仓中
- 通过缓存Redis设置Setnx操作，自带原子性。重复发送消息毕竟时偶然性事件，推荐该中方法，可以设置MessageId的存储时间。

唯一消息ID生成注意事项：**数据库自增长的ID有什么缺陷么？**

（1）、长度有限，能够预见

（2）、不安全，黑客可以猜测，可以攻击

（3）、对[数据库分布式]支持不是特别的友好

使用：替代方案

（1）、UUID 产生的字符串是无序的

（2）、雪花ID 产生的数据是有序的

**雪花算法**：

snowflake是Twitter开源的分布式ID生成算法，结果是64bit的Long类型的ID，有着全局唯一和有序递增的特点。

<img src="./assets/image-20241010154115729.png" alt="image-20241010154115729" style="zoom: 67%;" />

最高位是符号位，因为生成的 ID 总是正数，始终为0，不可用。

41位的[时间序列]，精确到毫秒级，41位的长度可以使用69年。时间位还有一个很重要的作用是可以根据时间进行排序。

10位的机器标识，10位的长度最多支持部署1024个节点。

12位的计数序列号，序列号即一系列的自增ID，可以支持同一节点同一毫秒生成多个ID序号，12位的计数序列号支持每个节点每毫秒产生4096个ID序号



生产者消息投递处：

```go
err = ch.Publish(exchangeName, routingKey, false, false, amqp.Publishing{
MessageId: unique.SnowFlaskID(1), // 通过雪花算法生成唯一ID
Type: "application/json",
Body: []byte("测试direct模式"),
})
```

在消费时查询已投递成功是否已存在

```go
type Message struct {
    Id string `json:"id" orm:"primaryKey"`
    ContentType string `json:"content_type"`
    Body []byte `json:"body"`
}
for msg := range message {
    db.Transaction(func(tx *orm.DB) error {
        // 投递的消息是否已被消费
        count := int64(0)
        u := Message{}
        db.Model(&u).Where("id=?", msg.MessageId).Count(&count)
        if count > 0 {
            msg.Reject(false) // 拒绝消息，自动删除
        }
        // 处理具体业务
        ...
        ...
        // 投递的消息未被消费
        if count == 0 {
            // 记录数据
            tx.Create(&Message{msg.MessageId, msg.ContentType, msg.Body})
            // ACK 应答
            msg.Ack(false)
        }
        // 返回 nil 提交事务
        return nil
    })
}
```

## 5.2 消息可靠性保证

消息中间一般用于异步通信、解耦、流量削峰、队列缓冲、支持高并发（限流）

由于很多时候把同步的业务变成异步化处理，在这个过程中是不可靠的发送

<img src="./assets/image-20241010154222294.png" alt="image-20241010154222294" style="zoom:50%;" />

所以保证事务一致性

<img src="./assets/image-20241010154237327.png" alt="image-20241010154237327" style="zoom:67%;" />

设置一个消息数据表：

状态: 0、初始化 1、发送中 2、成功消费 3、未被消费

正常流程：

1. 消息生产者设置为confirm模式
2. 消息生产者发送前生成本地消息记录、状态、0
3. 当生产者确定broker可用，设置 状态置为1
4. 当消息者消费后，消费者设置Ack

重发机制：

1、定义任务扫描数据表，如果状态为0，批量发送数据

2、Retrys 小于阈值设置的最大值，消息未被消费，记录状态为0

设置消息发送的Timeout时间

超时机制：

1、设置消息expiration

 2、本地消息表中状态一直为发送中

3、超时消息状态归零

# Rabbit安装部署

## 1.1 解决docker镜像源的问题

由于国内docker镜像不能访问的问题，要配置镜像源

### 1.1.1 创建或修改docker配置文件

内容如下

```go
vi /etc/docker/daemon.json
{
    "registry-mirrors":[
        "https://x9r52uz5.mirror.aliyuncs.com",
        "https://dockerhub.icu",
        "https://docker.chenby.cn",
        "https://docker.1panel.live",
        "https://docker.awsl9527.cn",
        "https://docker.anyhub.us.kg",
        "https://dhub.kubesre.xyz"
    ]
}
```

重启daemon和docker

```sh
systemctl daemon-reload
systemctl restart docker
```

### 1.1.2 修改dns服务配置

```go
vi /etc/resolv.conf
nameserver 114.114.114.114
nameserver 8.8.8.8
```

## 1.2 RabbitMQ安装

```sh
docker pull rabbitmq:management
docker run -id --name=rabbitmq -p 5672:5672 -p 15671:15671 -p 15672:15672 -p
25672:25672 -e RABBITMQ_DEFAULT_USER=root -e RABBITMQ_DEFAULT_PASS=123456
rabbitmq:management
# 5672 ampq默认端口
# 15671 RabbitMq管理界面端口
# 25672 RabbitMq https端口
```

如何后台没有开启，可进入容器内

```sh
docker exec -it rabbitmq /bin/bash
# 进入后执行下面命令
rabbitmq-plugins enable rabbitmq_management
firewall-cmd --zone=public --add-port=5672/tcp --permanent && firewall-cmd --
reload
firewall-cmd --zone=public --add-port=15672/tcp --permanent && firewall-cmd --
reload
firewall-cmd --reload
```

浏览器中仍然不能访问192.168.31.202:15672的情况，修改下面内容：

```sh
# 进入容器
docker exec -it {rabbitmq容器名称或者id} /bin/bash
#进入容器后，cd到以下路径
cd /etc/rabbitmq/conf.d/
#修改 management_agent.disable_metrics_collector = false
echo management_agent.disable_metrics_collector = false >
management_agent.disable_metrics_collector.conf
#退出容器
exit
#重启rabbitmq容器
docker restart {rabbitmq容器id或容器名称}
```

## 1.3 使用说明

下面将介绍rabbitMQ的使用及代码实现

- 使用语言：golang
- 使用包： github.com/streadway/amqp

实践部署机器地址：

```go
http://192.168.31.202:15672
账号/密码:root/123456
```

