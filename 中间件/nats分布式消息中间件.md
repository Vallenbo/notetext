# nats使用

# 一、介绍

[官网](https://nats.io/)	| 	[官网下载](https://nats.io/download/)	|	[nats - github 地址](https://github.com/nats-io/nats-server/releases)	| [dockerhub链接](https://hub.docker.com/_/nats)

NATS是一个开源、轻量级、高性能的分布式消息中间件，使用Golang语言开发，实现了高可伸缩性和优雅的Publish/Subscribe模型，但他不保证消息的到达，持久性等特性，nats streaming即为解决这一问题

NATS的开发哲学认为高质量的QoS应该在客户端构建，故只建立了Request-Reply，不提供 

- 1.持久化 
- 2.事务处理 
- 3.增强的交付模式 
- 4.企业级队列。

## NATS消息传递模型

NATS支持各种消息传递模型，包括：

- 发布订阅（Publish Subscribe）
- 请求回复（Request Reply）
- 队列订阅（Queue Subscribers )

提供的功能：

- 纯粹的发布订阅模型（Pure pub-sub）

- 服务器集群（Cluster mode server）

- 自动精简订阅者（Auto-pruning of subscribers)

- 基于文本协议（Text-based protocol）

- 多服务质量保证（Multiple qualities of service - QoS）

- 发布订阅（Publish Subscribe）


nats三种工作模式：

-  pub/sub (1对多)

-  request/reply(1对多 设置超时，只要有一个回复就结束)

-  queue(1对1)


优点：

-  1.使用简单，配置简单。

-  2.速度极快，性能良好。

-  3.多语言支持，不依赖于网络位置，client端只需知道nats的节点和约定好的subject名称即可。


缺点：

-  1.对服务器稳定性要求较高，机房出现故障，导致nats server端需要重连。可能需要重启nats server。


-  2.在消息timeout后，需要在reconnection里要重新初始化连接，不方便。


# 二、安装

docker安装

2.1 docker方式安装

```sh
docker pull nats
# 启动：
docker run -d -p 4222:4222 -p 6222:6222 -p 8222:8222 --name nats-main
nats:latest
```

2.2 nats 下载安装

```sh
cd /usr/local/
tar -xzvf nats-server-v2.9.15-linux-amd64.tar.gz
mv nats-server-v2.9.15-linux-amd64 nats-server
cd nats-server
[root@testyxqy local]# tree nats-server/
nats-server/
|-- LICENSE
|-- nats-server
`-- README.md
```

2.3 nats默认配置文件

```go
# Client port of 4222 on all interfaces
port: 4222
# HTTP monitoring port
monitor_port: 8222
# This is for clustering multiple servers together.
cluster {
    # It is recommended to set a cluster name
    name: "my_cluster"
    # Route connections to be received on any interface on port 6222
    port: 6222
    # Routes are protected, so need to use them with --routes flag
    # e.g. --routes=nats-route://ruser:T0pS3cr3t@otherdockerhost:6222
    authorization {
        user: ruser
        password: T0pS3cr3t
        timeout: 2
    }
    # Routes are actively solicited and connected to from this server.
    # This Docker image has none by default, but you can pass a
    # flag to the nats-server docker image to create one to an existing server.
    routes = []
}
```

启动参数说明

```go
服务器选项:
-a, --addr <host> 绑定主机IP地址（默认是0.0.0.0）
-p, --port <port> 客户端连接NATS服务器使用的端口（默认是4222）
-n, --name <server_name> 服务器名字(默认:自动)
-P, --pid <file> 存储PID的文件
-m, --http_port <port> HTTP监听端口
-ms,--https_port <port> HTTPS监听端口
-c, --config <file> 指定配置文件
-t 测试配置文并退出
-sl,--signal <signal>[=<pid>] 向 nats-server 进程发送信号（停止、退出、重新打
开、重新加载）
<pid> 可以是 PID（例如 1）或 PID 文件的路径（例
如 /var/run/nats-server.pid）
--client_advertise <string> 向其他服务器广播的客户端 URL
日志选项:
-l, --log <file> 指定日志输出的文件
-T, --logtime 是否开启日志的时间戳（默认为true）
-s, --syslog 启用syslog作为日志方法
-r, --remote_syslog <addr> 远程日志服务器的地址（默认为udp://localhost:514）
-D, --debug 开启调试输出
-V, --trace 跟踪原始的协议
-VV 详细跟踪（也跟踪系统帐户）
-DV 调试并跟踪
-DVV 调试和详细跟踪（也跟踪系统帐户）
JetStream 选项:
-js, --jetstream 启用 JetStream 功能。
-sd, --store_dir <dir> 设置存储目录。
授权认证选项:
--user <user> 连接需要的用户名
--pass <password> 连接需要的密码
--auth <token> 连接所需的授权令牌
TLS 安全选项:
--tls 启用TLS，不验证客户端（默认为false）
--tlscert <file> 服务器证书文件
--tlskey <file> 服务器证书私钥
--tlsverify 启用TLS，每一个客户端都要认证
--tlscacert <file> 客户端证书CA用于认证
集群选项:
--routes <rurl-1, rurl-2> 请求和连接的路由
--cluster <cluster-url> 请求路由的集群 URL
--cluster_name <string> Cluster Name，如果不设置会动态生成一个
--no_advertise <bool> 不向客户端通告已知的集群信息
--cluster_advertise <string> 向其他服务器通告的集群 URL
--connect_retries <number> 连接重试次数
常规选项:
-h, --help 显示帮助消息
-v, --version 显示版本信息
--help_tls 显示TLS 帮助消息
启动参数配置说明
```

2.3 使用配置文件启动

```sh
[root@testyxqy nats-server]# ./nats-server -c nats.yml
[16228] 2023/04/18 16:46:24.250449 [INF] Starting nats-server
[16228] 2023/04/18 16:46:24.250503 [INF] Version: 2.9.15
[16228] 2023/04/18 16:46:24.250508 [INF] Git: [b91fa85]
[16228] 2023/04/18 16:46:24.250512 [INF] Cluster: my_cluster
[16228] 2023/04/18 16:46:24.250516 [INF] Name:
NAJWLAJQZ6PLSDAYCLUYF4JOU6BVERLWFBP42CZFHYEDN4WXP7RRSODT
[16228] 2023/04/18 16:46:24.250520 [INF] ID:
NAJWLAJQZ6PLSDAYCLUYF4JOU6BVERLWFBP42CZFHYEDN4WXP7RRSODT
[16228] 2023/04/18 16:46:24.250537 [INF] Using configuration file: nats.yml
[16228] 2023/04/18 16:46:24.251156 [INF] Starting http monitor on 0.0.0.0:8222
[16228] 2023/04/18 16:46:24.251264 [INF] Listening for client connections on
0.0.0.0:4222
[16228] 2023/04/18 16:46:24.251478 [INF] Server is ready
[16228] 2023/04/18 16:46:24.251525 [INF] Cluster name is my_cluster
[16228] 2023/04/18 16:46:24.251558 [INF] Listening for route connections on
0.0.0.0:6222
[root@testyxqy ~]# netstat -ntlp | grep nats
tcp6 0 0 :::4222 :::* LISTEN
16228/./nats-server
tcp6 0 0 :::8222 :::* LISTEN
16228/./nats-server
tcp6 0 0 :::6222 :::* LISTEN
16228/./nats-server
```

web管理地址：http://192.168.31.202:8000/

## pub/sub订阅

```sh
使用两个客户端进行验证。在远程Windows主机上开两个CMD命令行环境，均使用命令“C:> telnet xxx.xxx.xxx.xxx 4222”连上nats-server。为了以示区别，这里命名为客户端A和客户端B，A表示发布者，B表示订阅者。
telnet 连接显示 4222
INFO
{"server_id":"NDUMVY3NZU4MICIB53H6JYBWRL2JFSLOB3P3PEHA3ZD2TIKTVAPWIRBS","server_name":"NDUMVY3NZU4MICIB53H6JYBWRL2JFSLOB3P3PEHA3ZD2TIKTVAPWIRBS","version":"2.10.20","proto":1,"git_commit":"7140387","go":"go1.22.7","host":"0.0.0.0","port":4222,"headers":true,"max_payload":1048576,"client_id":11,"client_ip":"192.168.31.55","xkey":"XBI66WZ5NBFZMHQ6AFZWEKDBTW2Q5ZIDJGVRHX2KKXTIN6IACTDWPB7O"}
```

## 客户端测试

```sh
# 1）订阅者B运行
订阅者B使用通配符foot.*注册主题ID为90的主题，订阅成功，nats-server返回+OK消息
sub foo.* 90
+OK

# 2）发布者A运行
发布者A发布一条消息到主题foo.bar，消息有效负载的长度为5，按下回车。消息发布成功，nats-server
返回+OK消息。
pub foo.bar 5
hello
+OK

# 订阅者B显示：可以看到获得的消息
PING
MSG foo.bar 90 5
hello

# 3）发布者A继续执行
发布者A继续执行以下命令，消息发布成功，nats-server服务器返回+OK消息
pub foo.bar optional.reply.subject 5
hello
+OK

# 订阅者B显示：PING是维持连接的消息
PING
MSG foo.bar 90 optional.reply.subject 5
hello
```





# golang代码示例

引用包

```go
go get github.com/nats-io/nats.go
```

订阅者

```go
package main
import (
    "fmt"
    "github.com/nats-io/nats.go"
)
////nats-server 在管理 subject 的时候是通过’.’ 进行分割的，server 底层是使用 tree
module 分层管理 subject. 此处有两个通配符*和>。
////*可以匹配以.分割的一切。如：
////nc.Subscribe("aa.*.cc", func(m *Msg) {}) 可以匹配 aa.11.cc、aa.zngw.cc,但不能匹
配aa.11.zngw.cc
////> 需要放在通配符最后，匹配后面所有长度。如：
////nc.Subscribe("aa.>", func(m *Msg) {})，这个匹配所有 aa.开头的subject
func main() {
    // 连接Nats服务器
    nc, _ := nats.Connect("nats://192.168.31.202:4222")
    // 发布-订阅 模式，异步订阅 test1
    _, _ = nc.Subscribe("test1", func(m *nats.Msg) {
        fmt.Printf("订阅收到消息: %s\n", string(m.Data))
    })
    // 队列 模式，订阅 test2， 队列为queue, test2 发向所有队列，同一队列只有一个能收到消息
    _, _ = nc.QueueSubscribe("test2", "queue", func(msg *nats.Msg) {
        fmt.Printf("队列消息: %s\n", string(msg.Data))
    })
    nc.Subscribe("test.*.cc", func(msg *nats.Msg) {
        fmt.Printf("所有test.*.cc %s\n", string(msg.Data))
    })
    // 请求-响应， 响应 test3 消息。
    _, _ = nc.Subscribe("test3", func(m *nats.Msg) {
        fmt.Printf("接收请求消息%s\n", string(m.Data))
        _ = nc.Publish(m.Reply, []byte("thank you tony!!!"))
    })
    // 持续发送不需要关闭
    //_ = nc.Drain()
    // 关闭连接
    defer nc.Close()
    // 阻止进程结束而收不到消息
    select {}
}
```

生产者

```go
func main() {
    // 连接Nats服务器
    nc, _ := nats.Connect("nats://192.168.31.202:4222")
    // 发布-订阅 模式，向 test1 发布一个 `Hello World` 数据
    _ = nc.Publish("test1", []byte("Hello World test1"))
    // 队列 模式，发布是一样的，只是订阅不同，向 test2 发布一个 `Hello World` 数据
    _ = nc.Publish("test2", []byte("Hello World test2"))
    // 请求-响应， 向 test3 发布一个 `Hello World` 请求数据，设置超时间3秒，如果有多个响
    应，只接收第一个收到的消息
    msg, err := nc.Request("test3", []byte("Hello World test3"), 3*time.Second)
    if err != nil {
        fmt.Println(err)
    } else {
        fmt.Printf(string(msg.Data))
    }
    nc.Publish("test4", []byte("test4 message"))
    nc.Publish("test.aa.cc", []byte("test.aa.cc"))
    nc.Publish("test.bb.cc", []byte("test.bb.cc"))
    // 关闭连接
    defer nc.Close()
    select {}
}
```

