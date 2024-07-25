# kafka

[Apache Kafka](https://kafka.apache.org/documentation/)

# 通信模型

**点对点模式(queue消息队列)---单通道**
消息生产者生产消息发送到queu消息队列中，然后消息消费者从queue中取出并且消费消息。一条消息被消费以后，queue中就没有了，不存在重复消费。
**发布/订阅(topic)---广播方式**
消息生产者(发布)将消息发布到topic中，同时有多个消息消费者(订阅) 消费该消息。和点对点方式不同，发布到topic的消息会被所有订阅者消费(类似于关注了微信公众号的人都能收到推送的文章)。

补充:发布订阅模式下，当发布者消息量很大时，显然单个订阅者的处理能力是不足的。实际上现实场景中是多个订阅者节点组成一个订阅组（由组内一个成员中转传递消息），负载均衡消费topic消息即分组订阅，这样订阅者很容易实现消费能力线性扩展。可以看成是一个topic下有多个Queue，每个Queue是点对点的方式，Queue之间是发布订阅方式。

# 介绍

Kafka是一个分布式数据流平台，支持部署形成集群。它提供了发布和订阅功能，使用者可以发送数据到Kafka中，也可以从Kafka中读取数据(以便进行后续的处理)。
Kafka是一种高吞吐量的分布式发布订阅消息系统，它可以处理消费者规模的网站中的所有动作流数据，具有高性能、持久化、多副本备份、横向扩展等特点。

<img src="assets/image-20230424112405381.png" alt="image-20230424112405381" style="zoom: 67%;" />

# 一、Kafka集群的架构

kafka cluster: kafka集群

**1、Broker**: 指部署了Kafka实例的**服务器节点**（有唯一的标识）。每个服务器.上有一个或多个kafka的实例，我们姑且认为每个broker对应一台服务器。每个kafka集群内的broker都有一个不重复的编号，如图中的broker-0、broker-1等....
**2、Topic**: 消息的主题（如nginx专用），kafka的数据就保存在topic。在每个broker上都可以创建多个topic。实际应用中通常是一个业务线建一个topic。
**3、Partition**: Topic的分区，每个topic可以有多个分区，分区的作用是做负载，提高kafka的吞吐量。Topic分区的角色（Follower:分区的从节点）。同一个topic的不同Partition分区数据是不重复的，partition的表现形式就是一个一个的文件夹!
**4、Replication**:备份文件，每一个分区都有多个副本。当主分区(Leader) 故障的时候会选择一个备胎(Follower)上位， 成为Leader。	在kafka中默认副本的最大数量是10个，且副本的数量不能大于Broker的数量，follower和leader在不同的机器，同一机器对同一个分区也只可能存放一个副本(包括自己)。
**5、Consumer**:消费者，即消息的消费方，是消息的出口。
**Consumer Group**:我们可以将多个消费组组成一个消费者组，在kafka的设计中同一个分区的数据只能被消费者组中的某一个消费者消费。同一个消费者组的消费者可以消费同一个topic的不同分区的数据，这也是为了提高kafka的吞吐量!

**6、Producer**: 即生产者，消息的产生者，是消息的入口。

Zookeeper：kafka集群依赖zookeeper来保存集群的的元信息，来保证系统的可用性。

## 工作流程

生产者往kafka发送数据的流程（6步）
<img src="assets/image-20230424112414352.png" alt="image-20230424112414352" style="zoom:67%;" />

1、生产者从Kafka集群获取分区leader信息
2、生产者将消息发送给leader
3、leader将消息写入本地磁盘
4、follower从leader拉取消息数据
5、follower将消息写入本地磁盘后向leader发送ACK
6、leader收到所有的follower的ACK之后向生产者发送ACK

## 选择partition的原则

某个 topic可以设置多个parition目录，生产者数据应该发往哪个分区呢？
1、partition在写入的时候可以指定需要写入的partition,如果有指定，则写入对应的partition。
2、如果没有指定partition,但是设置了数据的key,则会根据key的值hash出一个partition。
3、如果既没指定partition，又没有设置key,则会采用轮询方式，即每次取一小段时间的数据写入某个partition，下一小段的时间写入下一个partition。

# 二、kafka数据发送ack应答机制

生产者往kafka发送数据的模式（3种）：
producer在向kafka写入消息的时候，设置确认参数确定kafka接收到数据，这个参数可设置的值为0、1、all。
● 0代表producer往集群发送数据不需要等到集群的返回，不确保消息发送成功。安全性最低但是效率最高。
● 1代表producer往集群发送数据只要leader应答就可以发送下一条，只确保leader发送成功。
● all代表producer往集群发送数据需要所有的follower都完成从leader的同步才会发送下一条，确保leader发送成功和所有的副本都完成备份。安全性最高，但是效率最低。

# 三、Topic分区数据日志文件结构

对于每个Topic主题下 partition数据存储如下，Kafka集群维护了一个partition分区数据日志文件结构如下:
![image-20230424112426154](assets/image-20230424112426154.png)
如上图，假如某个Topic有三个partition分区，采用轮询方式，在同一个partition上数据是有序存储的

每个partition都是一个有序并且不可变的消息记录集合。
	当新的数据写入时，追加到partition的末尾。在每个partition中，每条消息都会被分配一个顺序的唯一标识， 这个标识被称为offset（即偏移量），Kafka只保证在同一个partition内部消息是有序的，在不同partition之间并不能保证消息有序。

Kafka可以配置一个保留期限，用来标识日志会在Kafka集群内保留多长时间。Kafka集群会保留在保留期限内所有被发布的消息，不管这些消息是否被消费过，超过保留期，这些数据将会被清空。由于Kafka会将数据进行持久化存储(即写入到硬盘上)，所以保留的数据大小可以设置为一个比较大的值。

**offset**

- offset记录着下一条将要发送给Consumer的消息的序号
- 默认Kafka将offset存储在ZooKeeper中
- 在一个分区中，消息是有顺序的方式存储着，每个在分区的消费都是有一个递增的id。这个就是偏移量offset
- 偏移量在分区中才是有意义的。在分区之间，offset是没有任何意义的



## Partition结构
Partition以文件夹的形式存在， 每个partition文件夹下面会有多组segment文件，每组segment文件又包含. index文件、.1og文件、.timeindex文件三个文件

- `.1og`文件就是实际存储message的地方
- `.index`和`.time .index`文件为索引文件，用于检索消息。

## 为什么kafka快?
虽然是写入物理磁盘，但是每条记录都是通过index索引能快速定位

# 四、Consumer组消费数据

多个消费者实例可以组成一个消费者组，并用一个标签来标识这个消费者组。一个消费者组中的不同消费者实例可以运行在不同的进程甚至不同的服务器上。如果所有的消费者实例都在同一个消费者组中，那么消息记录会被很好的均衡的发送到每个消费者实例。
如果所有的消费者实例都在不同的消费者组，那么每一条消息记录会被广播到每一个消费者实例。

![image-20230424112434869](assets/image-20230424112434869.png)
每个消费者实例可以消费多个分区,但是每个分区最多只能被消费者组中的一个实例消费。





# kafka安装教程

## 一、需要jdk1.8.0_201环境

```sh
方式一:
ubantu : apt-get install openjdk-8-jdk -y
centos : yum install openjdk-8-jdk -y

方式二: tar -zxf jdk1.8.0_201.tar# 解压文件
```

配置环境变量

```sh
vim /etc/profile # 在 profile 文件最后加上
export JAVA_HOME=/usr/local/java/jdk1.8.0_201
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tool.jar

source /etc/profile			# 使配置生效
java -v # 进行检测
```

 

## 二、使用并启动kafka内置 zookeeper

kafka 依赖 zookeeper，kafka内置zookeeper

作用：查询kafka节点及进行连接

使用内置kafka内置zk

```sh
bin/zookeeper-server-start.sh -daemon config/zookeeper.properties # 
bin/zookeeper-server-start.sh config/zookeeper.properties &  # 后台启动，不会打印日志到控制台
ss -pln | grep 2181 #查看进程端口状态
```

**zookeeper安装**

```go
tar ‐zxvf apache‐zookeeper‐3.5.8‐bin.tar.gz	# 解压文件
cp conf/zoo_sample.cfg conf/zoo.cfg	# 复制一份配置文件, 方便修改
bin/zkServer.sh start	# 启动
```



## 三、安装 kafka

```go
wget https://dlcdn.apache.org/kafka/3.6.0/kafka_2.13-3.6.0.tgz

tar -xvf kafka_2.13-3.6.0.tgz -C /usr/local/
```

修改配置文件

```sh
vim config/server.config

broker.id=0		# broker.id属性在kafka集群中必须要是唯一

listeners=PLAINTEXT://192.xxx.xx.xx:9092		# kafka部署的机器ip和提供服务的端口号,切勿设0.0.0.0可能报错

log.dir=/usr/local/data/kafka‐logs# kafka的消息存储文件

zookeeper.connect=192.xxx.xx.xx:2181# kafka 连接 zookeeper 的地址
```

**启动服务**

运行的日志打印在 logs 目录里的server.log 里

```go
方式一：bin/kafka‐server‐start.sh ‐daemon config/server.properties 
方式二：bin/kafka‐server‐start.sh config/server.properties &  # 后台启动，不会打印日志到控制台
```

**停止kafka**

```go
bin/kafka‐server‐stop.sh
```

启动成功后，可以进入zookeeper 查看kafka节点

```go
root@123:/usr/local/kafka_2.13-3.6.0# bin/zookeeper-shell.sh  192.168.0.3:2181
Connecting to 192.168.0.3:2181
Welcome to ZooKeeper!
JLine support is disabled

ls /brokers/ids
WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[0]
```



## 四、kafka使用相关命令

参数说明：

`--bootstrapver`指定连接服务器		 `--from-beginning`从最前开始读取

`-group test_group`指定 Group



**topic**----相关命令

```sh
#创建topic
bin/kafka-topics.sh --create --bootstrap-server 192.168.0.3:9092 --replication-factor 1 --partitions 1 --topic test2

#查看所有topic
bin/kafka-topics.sh --bootstrap-server 192.168.0.3:9092 --list

#查看topic具体内容
bin/kafka-console-consumer.sh --bootstrap-server 192.168.0.3:9092 --topic test2 --from-beginning

#清楚 Topic内数据
bin/kaftopics.sh --bootstrap-server 192.168.0.3:9092 --topic ehminer --delete
```

**producer生产者**----使用命令

```sh
#向指定topic中生产数据
bin/kafka-console-producer.sh --broker-list 192.168.0.3:9092 --topic test2
#例如：{"id":"1","name":"xiaoming","age":"20"}
```

**consumer消费者**----使用命令

```sh
#创建消费者组
bin/kafka-console-consumer.sh --bootstrap-server 192.168.0.3:9092 --topic test2 --group kafkatest

#消费数据，从头部开始
bin/kafka-console-consumer.sh --bootstrap-server 192.168.0.3:9092 --topic test2 --from-beginning

#消费数据，尾部开始，必需要指定分区：
/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --offset latest --partition 0

#查看消费者组
bin/kafka-consumer-groups.sh --bootstrap-server 192.168.0.3:9092 --list

#查看消费者详情
bin/kafka-consumer-groups.sh --bootstrap-server 192.168.0.3:9092 --describe  --group kafkatest
```





