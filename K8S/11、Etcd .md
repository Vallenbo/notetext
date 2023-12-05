# Etcd介绍

# 1. Etcd是什么（what）

etcd is a distributed, consistent key-value store for shared configuration and service discovery, with a focus on being:

- Secure: automatic TLS with optional client cert authentication[可选的SSL客户端证书认证：支持https访问 ]
- Fast: benchmarked 10,000 writes/sec[单实例每秒 1000 次写操作]
- Reliable: properly distributed using Raft[使用Raft保证一致性]

etcd是一个分布式、一致性的键值存储系统，主要用于配置共享和服务发现。[以上内容来自etcd官网]

# 2. 为什么使用Etcd（why）

## 2.1. Etcd的优势

1. 简单。使用Go语言编写部署简单；使用HTTP作为接口使用简单；使用Raft算法保证强一致性让用户易于理解。
2. 数据持久化。etcd默认数据一更新就进行持久化。
3. 安全。etcd支持SSL客户端安全认证。

# 3. 如何实现Etcd架构（how）

## 3.1. Etcd的相关名词解释

- Raft：etcd所采用的保证分布式系统强一致性的算法。
- Node：一个Raft状态机实例。
- Member： 一个etcd实例。它管理着一个Node，并且可以为客户端请求提供服务。
- Cluster：由多个Member构成可以协同工作的etcd集群。
- Peer：对同一个etcd集群中另外一个Member的称呼。
- Client： 向etcd集群发送HTTP请求的客户端。
- WAL：预写式日志，etcd用于持久化存储的日志格式。
- snapshot：etcd防止WAL文件过多而设置的快照，存储etcd数据状态。
- Proxy：etcd的一种模式，为etcd集群提供反向代理服务。
- Leader：Raft算法中通过竞选而产生的处理所有数据提交的节点。
- Follower：竞选失败的节点作为Raft中的从属节点，为算法提供强一致性保证。
- Candidate：当Follower超过一定时间接收不到Leader的心跳时转变为Candidate开始竞选。【候选人】
- Term：某个节点成为Leader到下一次竞选时间，称为一个Term。【任期】
- Index：数据项编号。Raft中通过Term和Index来定位数据。

## 3.2. Etcd的架构图

[![etcd的架构图](https://camo.githubusercontent.com/d28d1b1eb637e8bda5d11bbb7f30c01e792e600f1400c84030714f076a4cb11b/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f647178746e3069636b2f696d6167652f75706c6f61642f76313531303537383533322f61727469636c652f657463642f657463642d6172636869746563747572652e6a7067)](https://camo.githubusercontent.com/d28d1b1eb637e8bda5d11bbb7f30c01e792e600f1400c84030714f076a4cb11b/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f647178746e3069636b2f696d6167652f75706c6f61642f76313531303537383533322f61727469636c652f657463642f657463642d6172636869746563747572652e6a7067)

一个用户的请求发送过来，会经由HTTP Server转发给Store进行具体的事务处理，如果涉及到节点的修改，则交给Raft模块进行状态的变更、日志的记录，然后再同步给别的etcd节点以确认数据提交，最后进行数据的提交，再次同步。

## 1、HTTP Server:

用于处理用户发送的API请求以及其它etcd节点的同步与心跳信息请求。

## 2、Raft:

Raft强一致性算法的具体实现，是etcd的核心。

## 3、WAL:

Write Ahead Log（预写式日志），是etcd的数据存储方式，用于系统提供原子性和持久性的一系列技术。除了在内存中存有所有数据的状态以及节点的索引以外，etcd就通过WAL进行持久化存储。WAL中，所有的数据提交前都会事先记录日志。

1. Entry[日志内容]:

   负责存储具体日志的内容。

2. Snapshot[快照内容]:

   Snapshot是为了防止数据过多而进行的状态快照，日志内容发生变化时保存Raft的状态。

## 4、Store:

用于处理etcd支持的各类功能的事务，包括数据索引、节点状态变更、监控与反馈、事件处理与执行等等，是etcd对用户提供的大多数API功能的具体实现。

# Raft算法

# 1. Raft协议[分布式一致性算法]

[![raft](https://camo.githubusercontent.com/060953be1835bafef07d5174b49644579a7109143b70344b50da2a1a42c68b70/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f647178746e3069636b2f696d6167652f75706c6f61642f76313531303537383533322f61727469636c652f657463642f726166742e706e67)](https://camo.githubusercontent.com/060953be1835bafef07d5174b49644579a7109143b70344b50da2a1a42c68b70/68747470733a2f2f7265732e636c6f7564696e6172792e636f6d2f647178746e3069636b2f696d6167652f75706c6f61642f76313531303537383533322f61727469636c652f657463642f726166742e706e67)

raft算法中涉及三种角色，分别是：

- `follower`: 跟随者
- `candidate`: 候选者，选举过程中的中间状态角色
- `leader`: 领导者

# 2. 过程

## 2.1. 选举

有两个timeout来控制选举，第一个是`election timeout`，该时间是节点从follower到成为candidate的时间，该时间是150到300毫秒之间的随机值。另一个是`heartbeat timeout`。

- 当某个节点经历完`election timeout`成为candidate后，开启新的一个选举周期，他向其他节点发起投票请求（Request Vote），如果接收到消息的节点在该周期内还没投过票则给这个candidate投票，然后节点重置他的election timeout。
- 当该candidate获得大部分的选票，则可以当选为leader。
- leader就开始发送`append entries`给其他follower节点，这个消息会在内部指定的`heartbeat timeout`时间内发出，follower收到该信息则响应给leader。
- 这个选举周期会继续，直到某个follower没有收到心跳，并成为candidate。
- 如果某个选举周期内，有两个candidate同时获得相同多的选票，则会等待一个新的周期重新选举。

## 2.2. 同步

当选举过程结束，选出了leader，则leader需要把所有的变更同步的系统中的其他节点，该同步也是通过发送`Append Entries`的消息的方式。

- 首先一个客户端发送一个更新给leader，这个更新会添加到leader的日志中。
- 然后leader会在给follower的下次心跳探测中发送该更新。
- 一旦大多数follower收到这个更新并返回给leader，leader提交这个更新，然后返回给客户端。

## 2.3. 网络分区

- 当发生网络分区的时候，在不同分区的节点接收不到leader的心跳，则会开启一轮选举，形成不同leader的多个分区集群。
- 当客户端给不同leader的发送更新消息时，不同分区集群中的节点个数小于原先集群的一半时，更新不会被提交，而节点个数大于集群数一半时，更新会被提交。
- 当网络分区恢复后，被提交的更新会同步到其他的节点上，其他节点未提交的日志会被回滚并匹配新leader的日志，保证全局的数据是一致的。

# Etcd启动配置参数

# 1. Etcd配置参数

```
/ # etcd --help
usage: etcd [flags]
       start an etcd server

       etcd --version
       show the version of etcd

       etcd -h | --help
       show the help information about etcd

       etcd --config-file
       path to the server configuration file

       etcd gateway
       run the stateless pass-through etcd TCP connection forwarding proxy

       etcd grpc-proxy
       run the stateless etcd v3 gRPC L7 reverse proxy
```



## 1.1. member flags

```
member flags:

	--name 'default'
		human-readable name for this member.
	--data-dir '${name}.etcd'
		path to the data directory.
	--wal-dir ''
		path to the dedicated wal directory.
	--snapshot-count '100000'
		number of committed transactions to trigger a snapshot to disk.
	--heartbeat-interval '100'
		time (in milliseconds) of a heartbeat interval.
	--election-timeout '1000'
		time (in milliseconds) for an election to timeout. See tuning documentation for details.
	--initial-election-tick-advance 'true'
		whether to fast-forward initial election ticks on boot for faster election.
	--listen-peer-urls 'http://localhost:2380'
		list of URLs to listen on for peer traffic.
	--listen-client-urls 'http://localhost:2379'
		list of URLs to listen on for client traffic.
	--max-snapshots '5'
		maximum number of snapshot files to retain (0 is unlimited).
	--max-wals '5'
		maximum number of wal files to retain (0 is unlimited).
	--cors ''
		comma-separated whitelist of origins for CORS (cross-origin resource sharing).
	--quota-backend-bytes '0'
		raise alarms when backend size exceeds the given quota (0 defaults to low space quota).
	--max-txn-ops '128'
		maximum number of operations permitted in a transaction.
	--max-request-bytes '1572864'
		maximum client request size in bytes the server will accept.
	--grpc-keepalive-min-time '5s'
		minimum duration interval that a client should wait before pinging server.
	--grpc-keepalive-interval '2h'
		frequency duration of server-to-client ping to check if a connection is alive (0 to disable).
	--grpc-keepalive-timeout '20s'
		additional duration of wait before closing a non-responsive connection (0 to disable).
```



## 1.2. clustering flags

```
clustering flags:

	--initial-advertise-peer-urls 'http://localhost:2380'
		list of this member's peer URLs to advertise to the rest of the cluster.
	--initial-cluster 'default=http://localhost:2380'
		initial cluster configuration for bootstrapping.
	--initial-cluster-state 'new'
		initial cluster state ('new' or 'existing').
	--initial-cluster-token 'etcd-cluster'
		initial cluster token for the etcd cluster during bootstrap.
		Specifying this can protect you from unintended cross-cluster interaction when running multiple clusters.
	--advertise-client-urls 'http://localhost:2379'
		list of this member's client URLs to advertise to the public.
		The client URLs advertised should be accessible to machines that talk to etcd cluster. etcd client libraries parse these URLs to connect to the cluster.
	--discovery ''
		discovery URL used to bootstrap the cluster.
	--discovery-fallback 'proxy'
		expected behavior ('exit' or 'proxy') when discovery services fails.
		"proxy" supports v2 API only.
	--discovery-proxy ''
		HTTP proxy to use for traffic to discovery service.
	--discovery-srv ''
		dns srv domain used to bootstrap the cluster.
	--strict-reconfig-check 'true'
		reject reconfiguration requests that would cause quorum loss.
	--auto-compaction-retention '0'
		auto compaction retention length. 0 means disable auto compaction.
	--auto-compaction-mode 'periodic'
		interpret 'auto-compaction-retention' one of: periodic|revision. 'periodic' for duration based retention, defaulting to hours if no time unit is provided (e.g. '5m'). 'revision' for revision number based retention.
	--enable-v2 'true'
		Accept etcd V2 client requests.
```



## 1.3. proxy flags

```
proxy flags:
	"proxy" supports v2 API only.

	--proxy 'off'
		proxy mode setting ('off', 'readonly' or 'on').
	--proxy-failure-wait 5000
		time (in milliseconds) an endpoint will be held in a failed state.
	--proxy-refresh-interval 30000
		time (in milliseconds) of the endpoints refresh interval.
	--proxy-dial-timeout 1000
		time (in milliseconds) for a dial to timeout.
	--proxy-write-timeout 5000
		time (in milliseconds) for a write to timeout.
	--proxy-read-timeout 0
		time (in milliseconds) for a read to timeout.
```



## 1.4. security flags

```
security flags:

	--ca-file '' [DEPRECATED]
		path to the client server TLS CA file. '-ca-file ca.crt' could be replaced by '-trusted-ca-file ca.crt -client-cert-auth' and etcd will perform the same.
	--cert-file ''
		path to the client server TLS cert file.
	--key-file ''
		path to the client server TLS key file.
	--client-cert-auth 'false'
		enable client cert authentication.
	--client-crl-file ''
		path to the client certificate revocation list file.
	--trusted-ca-file ''
		path to the client server TLS trusted CA cert file.
	--auto-tls 'false'
		client TLS using generated certificates.
	--peer-ca-file '' [DEPRECATED]
		path to the peer server TLS CA file. '-peer-ca-file ca.crt' could be replaced by '-peer-trusted-ca-file ca.crt -peer-client-cert-auth' and etcd will perform the same.
	--peer-cert-file ''
		path to the peer server TLS cert file.
	--peer-key-file ''
		path to the peer server TLS key file.
	--peer-client-cert-auth 'false'
		enable peer client cert authentication.
	--peer-trusted-ca-file ''
		path to the peer server TLS trusted CA file.
	--peer-auto-tls 'false'
		peer TLS using self-generated certificates if --peer-key-file and --peer-cert-file are not provided.
	--peer-crl-file ''
		path to the peer certificate revocation list file.
```



## 1.5. logging flags

```
logging flags

	--debug 'false'
		enable debug-level logging for etcd.
	--log-package-levels ''
		specify a particular log level for each etcd package (eg: 'etcdmain=CRITICAL,etcdserver=DEBUG').
	--log-output 'default'
		specify 'stdout' or 'stderr' to skip journald logging even when running under systemd.
```



## 1.6. unsafe flags

```
unsafe flags:

Please be CAUTIOUS when using unsafe flags because it will break the guarantees
given by the consensus protocol.

	--force-new-cluster 'false'
		force to create a new one-member cluster.
```



## 1.7. profiling flags

```
profiling flags:
	--enable-pprof 'false'
		Enable runtime profiling data via HTTP server. Address is at client URL + "/debug/pprof/"
	--metrics 'basic'
		Set level of detail for exported metrics, specify 'extensive' to include histogram metrics.
	--listen-metrics-urls ''
		List of URLs to listen on for metrics.
```



## 1.8. auth flags

```
auth flags:
	--auth-token 'simple'
		Specify a v3 authentication token type and its options ('simple' or 'jwt').
```



## 1.9. experimental flags

```
experimental flags:
	--experimental-initial-corrupt-check 'false'
		enable to check data corruption before serving any client/peer traffic.
	--experimental-corrupt-check-time '0s'
		duration of time between cluster corruption check passes.
	--experimental-enable-v2v3 ''
		serve v2 requests through the v3 backend under a given prefix.
```



# Etcd访问控制

# 1. ETCD资源类型

There are three types of resources in etcd

- `permission resources`: users and roles in the user store
- `key-value resources`: key-value pairs in the key-value store
- `settings resources`: security settings, auth settings, and dynamic etcd cluster settings (election/heartbeat)

# 2. 权限资源

**Users**：user用来设置身份认证（user：passwd），一个用户可以拥有多个角色，每个角色被分配一定的权限（只读、只写、可读写），用户分为root用户和非root用户。

**Roles**：角色用来关联权限，角色主要三类：root角色。默认创建root用户时即创建了root角色，该角色拥有所有权限；guest角色，默认自动创建，主要用于非认证使用。普通角色，由root用户创建角色，并分配指定权限。

注意：如果没有指定任何验证方式，即没显示指定以什么用户进行访问，那么默认会设定为 guest 角色。默认情况下 guest 也是具有全局访问权限的。如果不希望未授权就获取或修改etcd的数据，则可收回guest角色的权限或删除该角色，etcdctl role revoke 。

**Permissions**:权限分为只读、只写、可读写三种权限，权限即对指定目录或key的读写权限。

# 3. ETCD访问控制

## 3.1. 访问控制相关命令

```
NAME:
   etcdctl - A simple command line client for etcd.
USAGE:
   etcdctl [global options] command [command options] [arguments...]
VERSION:
   2.2.0
COMMANDS:
   user         user add, grant and revoke subcommands
   role         role add, grant and revoke subcommands
   auth         overall auth controls  
GLOBAL OPTIONS:
   --peers, -C          a comma-delimited list of machine addresses in the cluster (default: "http://127.0.0.1:4001,http://127.0.0.1:2379")
   --endpoint           a comma-delimited list of machine addresses in the cluster (default: "http://127.0.0.1:4001,http://127.0.0.1:2379")
   --cert-file          identify HTTPS client using this SSL certificate file
   --key-file           identify HTTPS client using this SSL key file
   --ca-file            verify certificates of HTTPS-enabled servers using this CA bundle
   --username, -u       provide username[:password] and prompt if password is not supplied.
   --timeout '1s'       connection timeout per request
```



## 3.2. user相关命令

```
[root@localhost etcd]# etcdctl user --help
NAME:
   etcdctl user - user add, grant and revoke subcommands
USAGE:
   etcdctl user command [command options] [arguments...]
COMMANDS:
   add      add a new user for the etcd cluster
   get      get details for a user
   list     list all current users
   remove   remove a user for the etcd cluster
   grant    grant roles to an etcd user
   revoke   revoke roles for an etcd user
   passwd   change password for a user
   help, h  Shows a list of commands or help for one command
    
OPTIONS:
   --help, -h   show help
```



添加root用户并设置密码

etcdctl --endpoints [http://172.16.22.36:2379](http://172.16.22.36:2379/) user add root

添加非root用户并设置密码

etcdctl --endpoints [http://172.16.22.36:2379](http://172.16.22.36:2379/) --username root:123 user add huwh

查看当前所有用户

etcdctl --endpoints [http://172.16.22.36:2379](http://172.16.22.36:2379/) --username root:123 user list

将用户添加到对应角色

etcdctl --endpoints [http://172.16.22.36:2379](http://172.16.22.36:2379/) --username root:123 user grant --roles test1 phpor

查看用户拥有哪些角色

etcdctl --endpoints [http://172.16.22.36:2379](http://172.16.22.36:2379/) --username root:123 user get phpor

## 3.3. role相关命令

```
[root@localhost etcd]# etcdctl role --help
NAME:
   etcdctl role - role add, grant and revoke subcommands
USAGE:
   etcdctl role command [command options] [arguments...]
COMMANDS:
   add      add a new role for the etcd cluster
   get      get details for a role
   list     list all roles
   remove   remove a role from the etcd cluster
   grant    grant path matches to an etcd role
   revoke   revoke path matches for an etcd role
   help, h  Shows a list of commands or help for one command
    
OPTIONS:
   --help, -h   show help
```



添加角色

etcdctl --endpoints [http://172.16.22.36:2379](http://172.16.22.36:2379/) --username root:2379 role add test1

查看所有角色

etcdctl --endpoints [http://172.16.22.36:2379](http://172.16.22.36:2379/) --username root:123 role list

给角色分配权限

```
[root@localhost etcd]# etcdctl role grant --help
NAME:
   grant - grant path matches to an etcd role
USAGE:
   command grant [command options] [arguments...]
OPTIONS:
   --path   Path granted for the role to access
   --read   Grant read-only access
   --write  Grant write-only access
   --readwrite  Grant read-write access
```



1、只包含目录 etcdctl --endpoints [http://172.16.22.36:2379](http://172.16.22.36:2379/) --username root:123 role grant --readwrite --path /test1 test1

2、包括目录和子目录或文件 etcdctl --endpoints [http://172.16.22.36:2379](http://172.16.22.36:2379/) --username root:123 role grant --readwrite --path /test1/* test1

查看角色所拥有的权限

etcdctl --endpoints [http://172.16.22.36:2379](http://172.16.22.36:2379/) --username root:2379 role get test1

## 3.4. auth相关操作

```
[root@localhost etcd]# etcdctl auth --help
NAME:
   etcdctl auth - overall auth controls
USAGE:
   etcdctl auth command [command options] [arguments...]
COMMANDS:
   enable   enable auth access controls
   disable  disable auth access controls
   help, h  Shows a list of commands or help for one command
    
OPTIONS:
   --help, -h   show help
```



开启认证

etcdctl --endpoints [http://172.16.22.36:2379](http://172.16.22.36:2379/) auth enable

# 4. 访问控制设置步骤

| 顺序 | 步骤                                     | 命令                                                         |
| ---- | ---------------------------------------- | ------------------------------------------------------------ |
| 1    | 添加root用户                             | etcdctl --endpoints http://: user add root                   |
| 2    | 开启认证                                 | etcdctl --endpoints http://: auth enable                     |
| 3    | 添加非root用户                           | etcdctl --endpoints http://: –username root: user add        |
| 4    | 添加角色                                 | etcdctl --endpoints http://: –username root: role add        |
| 5    | 给角色授权（只读、只写、可读写）         | etcdctl --endpoints http://: –username root: role grant --readwrite --path |
| 6    | 给用户分配角色（即分配了角色对应的权限） | etcdctl --endpoints http://: –username root: user grant --roles |

# 5. 访问认证的API调用



# etcdctl命令工具

# etcdctl命令工具-V3

> `etcdctl`的`v3`版本与`v2`版本使用命令有所不同，本文介绍`etcdctl v3`版本的命令工具的使用方式。

# 1. etcdctl的安装

`etcdctl`的二进制文件可以在 [github.com/coreos/etcd/releases](https://github.com/coreos/etcd/releases) 选择对应的版本下载，例如可以执行以下`install_etcdctl.sh`的脚本，修改其中的版本信息。

```
#!/bin/bash
ETCD_VER=v3.3.4
ETCD_DIR=etcd-download
DOWNLOAD_URL=https://github.com/coreos/etcd/releases/download

# Download
mkdir ${ETCD_DIR}
cd ${ETCD_DIR}
wget ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz 
tar -xzvf etcd-${ETCD_VER}-linux-amd64.tar.gz

# install
cd etcd-${ETCD_VER}-linux-amd64
cp etcdctl /usr/local/bin/
```



# 2. etcdctl V3

使用`etcdctl`v3的版本时，需设置环境变量`ETCDCTL_API=3`。

```
export ETCDCTL_API=3

# 或者在`/etc/profile`文件中添加环境变量
vi /etc/profile
...
export ETCDCTL_API=3
...
source /etc/profile

# 或者在命令执行前加 ETCDCTL_API=3
ETCDCTL_API=3 etcdctl --endpoints=$ENDPOINTS member list
```



查看当前etcdctl的版本信息`etcdctl version`。

```
[root@k8s-dbg-master-1 etcd]# etcdctl version
etcdctl version: 3.3.4
API version: 3.3
```



更多命令帮助可以查询`etcdctl —help`。

```
[root@k8s-dbg-master-1 etcd]# etcdctl --help
NAME:
	etcdctl - A simple command line client for etcd3.

USAGE:
	etcdctl

VERSION:
	3.3.4

API VERSION:
	3.3


COMMANDS:
	get			Gets the key or a range of keys
	put			Puts the given key into the store
	del			Removes the specified key or range of keys [key, range_end)
	txn			Txn processes all the requests in one transaction
	compaction		Compacts the event history in etcd
	alarm disarm		Disarms all alarms
	alarm list		Lists all alarms
	defrag			Defragments the storage of the etcd members with given endpoints
	endpoint health		Checks the healthiness of endpoints specified in `--endpoints` flag
	endpoint status		Prints out the status of endpoints specified in `--endpoints` flag
	endpoint hashkv		Prints the KV history hash for each endpoint in --endpoints
	move-leader		Transfers leadership to another etcd cluster member.
	watch			Watches events stream on keys or prefixes
	version			Prints the version of etcdctl
	lease grant		Creates leases
	lease revoke		Revokes leases
	lease timetolive	Get lease information
	lease list		List all active leases
	lease keep-alive	Keeps leases alive (renew)
	member add		Adds a member into the cluster
	member remove		Removes a member from the cluster
	member update		Updates a member in the cluster
	member list		Lists all members in the cluster
	snapshot save		Stores an etcd node backend snapshot to a given file
	snapshot restore	Restores an etcd member snapshot to an etcd directory
	snapshot status		Gets backend snapshot status of a given file
	make-mirror		Makes a mirror at the destination etcd cluster
	migrate			Migrates keys in a v2 store to a mvcc store
	lock			Acquires a named lock
	elect			Observes and participates in leader election
	auth enable		Enables authentication
	auth disable		Disables authentication
	user add		Adds a new user
	user delete		Deletes a user
	user get		Gets detailed information of a user
	user list		Lists all users
	user passwd		Changes password of user
	user grant-role		Grants a role to a user
	user revoke-role	Revokes a role from a user
	role add		Adds a new role
	role delete		Deletes a role
	role get		Gets detailed information of a role
	role list		Lists all roles
	role grant-permission	Grants a key to a role
	role revoke-permission	Revokes a key from a role
	check perf		Check the performance of the etcd cluster
	help			Help about any command

OPTIONS:
      --cacert=""				verify certificates of TLS-enabled secure servers using this CA bundle
      --cert=""					identify secure client using this TLS certificate file
      --command-timeout=5s			timeout for short running command (excluding dial timeout)
      --debug[=false]				enable client-side debug logging
      --dial-timeout=2s				dial timeout for client connections
  -d, --discovery-srv=""			domain name to query for SRV records describing cluster endpoints
      --endpoints=[127.0.0.1:2379]		gRPC endpoints
      --hex[=false]				print byte strings as hex encoded strings
      --insecure-discovery[=true]		accept insecure SRV records describing cluster endpoints
      --insecure-skip-tls-verify[=false]	skip server certificate verification
      --insecure-transport[=true]		disable transport security for client connections
      --keepalive-time=2s			keepalive time for client connections
      --keepalive-timeout=6s			keepalive timeout for client connections
      --key=""					identify secure client using this TLS key file
      --user=""					username[:password] for authentication (prompt if password is not supplied)
  -w, --write-out="simple"			set the output format (fields, json, protobuf, simple, table)
```



# 3. etcdctl 常用命令

## 3.1. 指定etcd集群

```
HOST_1=10.240.0.17
HOST_2=10.240.0.18
HOST_3=10.240.0.19
ENDPOINTS=$HOST_1:2379,$HOST_2:2379,$HOST_3:2379

etcdctl --endpoints=$ENDPOINTS member list
```



如果etcd设置了证书访问，则需要添加证书相关参数：

```
ETCDCTL_API=3 etcdctl --endpoints=$ENDPOINTS --cacert=<ca-file> --cert=<cert-file> --key=<key-file>  <command>
```



参数说明如下：

```
  --cacert=""				verify certificates of TLS-enabled secure servers using this CA bundle
  --cert=""					identify secure client using this TLS certificate file
  --key=""					identify secure client using this TLS key file
  --endpoints=[127.0.0.1:2379]		gRPC endpoints
```



可以自定义alias命令

```
# alias 命令，避免每次需要输入证书参数
alias ectl='ETCDCTL_API=3 etcdctl --endpoints=$ENDPOINTS --cacert=<ca-file> --cert=<cert-file> --key=<key-file>'

# 直接使用别名执行命令
ectl <command>
```



## 3.2. 增删改查

**1、增**

```
etcdctl --endpoints=$ENDPOINTS put foo "Hello World!"
```



**2、查**

```
etcdctl --endpoints=$ENDPOINTS get foo
etcdctl --endpoints=$ENDPOINTS --write-out="json" get foo
```



基于相同前缀查找

```
etcdctl --endpoints=$ENDPOINTS put web1 value1
etcdctl --endpoints=$ENDPOINTS put web2 value2
etcdctl --endpoints=$ENDPOINTS put web3 value3

etcdctl --endpoints=$ENDPOINTS get web --prefix
```



列出所有的key

```
etcdctl --endpoints=$ENDPOINTS get / --prefix --keys-only
```



3、**删**

```
etcdctl --endpoints=$ENDPOINTS put key myvalue
etcdctl --endpoints=$ENDPOINTS del key

etcdctl --endpoints=$ENDPOINTS put k1 value1
etcdctl --endpoints=$ENDPOINTS put k2 value2
etcdctl --endpoints=$ENDPOINTS del k --prefix
```



## 3.3. 集群状态

集群状态主要是`etcdctl endpoint status` 和`etcdctl endpoint health`两条命令。

```
etcdctl --write-out=table --endpoints=$ENDPOINTS endpoint status

+------------------+------------------+---------+---------+-----------+-----------+------------+
|     ENDPOINT     |        ID        | VERSION | DB SIZE | IS LEADER | RAFT TERM | RAFT INDEX |
+------------------+------------------+---------+---------+-----------+-----------+------------+
| 10.240.0.17:2379 | 4917a7ab173fabe7 | 3.0.0   | 45 kB   | true      |         4 |      16726 |
| 10.240.0.18:2379 | 59796ba9cd1bcd72 | 3.0.0   | 45 kB   | false     |         4 |      16726 |
| 10.240.0.19:2379 | 94df724b66343e6c | 3.0.0   | 45 kB   | false     |         4 |      16726 |
+------------------+------------------+---------+---------+-----------+-----------+------------+

etcdctl --endpoints=$ENDPOINTS endpoint health

10.240.0.17:2379 is healthy: successfully committed proposal: took = 3.345431ms
10.240.0.19:2379 is healthy: successfully committed proposal: took = 3.767967ms
10.240.0.18:2379 is healthy: successfully committed proposal: took = 4.025451ms
```



## 3.4. 集群成员

跟集群成员相关的命令如下：

```
	member add		    Adds a member into the cluster
	member remove		Removes a member from the cluster
	member update		Updates a member in the cluster
	member list		    Lists all members in the cluster
```



例如 `etcdctl member list`列出集群成员的命令。

```
etcdctl --endpoints=http://172.16.5.4:12379 member list -w table

+-----------------+---------+-------+------------------------+-----------------------------------------------+
|       ID        | STATUS  | NAME  |       PEER ADDRS       |                 CLIENT ADDRS                  |
+-----------------+---------+-------+------------------------+-----------------------------------------------+
| c856d92a82ba66a | started | etcd0 | http://172.16.5.4:2380 | http://172.16.5.4:2379,http://172.16.5.4:4001 |
+-----------------+---------+-------+------------------------+-----------------------------------------------+
```



# 4. etcdctl get

使用`etcdctl {command} --help`可以查看具体命令的帮助信息。

```
# etcdctl get --help
NAME:
	get - Gets the key or a range of keys

USAGE:
	etcdctl get [options] <key> [range_end]

OPTIONS:
      --consistency="l"			Linearizable(l) or Serializable(s)
      --from-key[=false]		Get keys that are greater than or equal to the given key using byte compare
      --keys-only[=false]		Get only the keys
      --limit=0				Maximum number of results
      --order=""			Order of results; ASCEND or DESCEND (ASCEND by default)
      --prefix[=false]			Get keys with matching prefix
      --print-value-only[=false]	Only write values when using the "simple" output format
      --rev=0				Specify the kv revision
      --sort-by=""			Sort target; CREATE, KEY, MODIFY, VALUE, or VERSION

GLOBAL OPTIONS:
      --cacert=""				verify certificates of TLS-enabled secure servers using this CA bundle
      --cert=""					identify secure client using this TLS certificate file
      --command-timeout=5s			timeout for short running command (excluding dial timeout)
      --debug[=false]				enable client-side debug logging
      --dial-timeout=2s				dial timeout for client connections
      --endpoints=[127.0.0.1:2379]		gRPC endpoints
      --hex[=false]				print byte strings as hex encoded strings
      --insecure-skip-tls-verify[=false]	skip server certificate verification
      --insecure-transport[=true]		disable transport security for client connections
      --key=""					identify secure client using this TLS key file
      --user=""					username[:password] for authentication (prompt if password is not supplied)
  -w, --write-out="simple"			set the output format (fields, json, protobuf, simple, table)
```

# etcdctl命令工具-V2

# 1. etcdctl介绍

etcdctl是一个命令行的客户端，它提供了一下简洁的命令，可理解为命令工具集，可以方便我们在对服务进行测试或者手动修改数据库内容。etcdctl与其他xxxctl的命令原理及操作类似（例如kubectl，systemctl）。

用法：etcdctl [global options] command [command options][args...]

# 2. Etcd常用命令

## 2.1. 数据库操作命令

etcd 在键的组织上采用了层次化的空间结构（类似于文件系统中目录的概念），数据库操作围绕对键值和目录的 CRUD [增删改查]（符合 REST 风格的一套操作：Create, Read, Update, Delete）完整生命周期的管理。

具体的命令选项参数可以通过 etcdctl command --help来获取相关帮助。

## 2.1.1. 对象为键值

1. ### [set[增:无论是否存在\]:etcdctl set key value

2. ### [mk[增:必须不存在\]:etcdctl mk key value

3. ### [rm[删\]:etcdctl rm key

4. ### [update[改\]:etcdctl update key value

5. ### [get[查\]:etcdctl get key

## 2.1.2. 对象为目录

1. ### [setdir[增:无论是否存在\]:etcdctl setdir dir

2. ### [mkdir[增:必须不存在\]: etcdctl mkdir dir

3. ### [rmdir[删\]:etcdctl rmdir dir

4. ### [updatedir[改\]:etcdctl updatedir dir

5. ### [ls[查\]:etcdclt ls

## [2.2. 非数据库操作命令

1. ### [backup[备份 etcd 的数据\]

   etcdctl backup

2. ### [watch[监测一个键值的变化，一旦键值发生更新，就会输出最新的值并退出\]

   etcdctl watch key

3. ### [exec-watch[监测一个键值的变化，一旦键值发生更新，就执行给定命令\]

   etcdctl exec-watch key --sh -c "ls"

4. ### [member[通过 list、add、remove、update 命令列出、添加、删除 、更新etcd 实例到 etcd 集群中\]

   etcdctl member list；etcdctl member add 实例；etcdctl member remove 实例；etcdctl member update 实例。

5. ### [etcdctl cluster-health[检查集群健康状态\]

## [2.3. 常用配置参数]

设置配置文件，默认为/etc/etcd/etcd.conf。

| 配置参数                     | 参数说明                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| 配置参数                     | 参数说明                                                     |
| -name                        | 节点名称                                                     |
| -data-dir                    | 保存日志和快照的目录，默认为当前工作目录，指定节点的数据存储目录 |
| -addr                        | 公布的ip地址和端口。 默认为127.0.0.1:2379                    |
| -bind-addr                   | 用于客户端连接的监听地址，默认为-addr配置                    |
| -peers                       | 集群成员逗号分隔的列表，例如 127.0.0.1:2380,127.0.0.1:2381   |
| -peer-addr                   | 集群服务通讯的公布的IP地址，默认为 127.0.0.1:2380.           |
| -peer-bind-addr              | 集群服务通讯的监听地址，默认为-peer-addr配置                 |
| -wal-dir                     | 指定节点的was文件的存储目录，若指定了该参数，wal文件会和其他数据文件分开存储 |
| -listen-client-urls          |                                                              |
| -listen-peer-urls            | 监听URL，用于与其他节点通讯                                  |
| -initial-advertise-peer-urls | 告知集群其他节点url.                                         |
| -advertise-client-urls       | 告知客户端url, 也就是服务的url                               |
| -initial-cluster-token       | 集群的ID                                                     |
| -initial-cluster             | 集群中所有节点                                               |
| -initial-cluster-state       | -initial-cluster-state=new 表示从无到有搭建etcd集群          |
| -discovery-srv               | 用于DNS动态服务发现，指定DNS SRV域名                         |
| -discovery                   | 用于etcd动态发现，指定etcd发现服务的URL [https://discovery.etcd.io/],用环境变量表示 |

# Etcd中的k8s数据

# [1. 读取数据key]

使用以下命令列出所有的key。

```
ETCDCTL_API=3 etcdctl --endpoints=<etcd-ip-1>:2379,<etcd-ip-2>:2379,<etcd-ip-3>:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt  --key=/etc/kubernetes/pki/apiserver-etcd-client.key  --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt get / --prefix --keys-only
```



参数说明：

```
  --cacert=""				verify certificates of TLS-enabled secure servers using this CA bundle
  --cert=""					identify secure client using this TLS certificate file
  --key=""					identify secure client using this TLS key file
  --endpoints=[127.0.0.1:2379]		gRPC endpoints
```



可以使用alias来重命名etcdctl一串的命令

```
alias ectl='ETCDCTL_API=3 etcdctl --endpoints=<etcd-ip-1>:2379,<etcd-ip-2>:2379,<etcd-ip-3>:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt  --key=/etc/kubernetes/pki/apiserver-etcd-client.key  --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt'
```



# [2. 集群数据]

## [2.1. node]

```
/registry/minions/<node-ip-1>
/registry/minions/<node-ip-2>
/registry/minions/<node-ip-3>
```



其他信息：

```
/registry/leases/kube-node-lease/<node-ip-1>
/registry/leases/kube-node-lease/<node-ip-2>
/registry/leases/kube-node-lease/<node-ip-3>

/registry/masterleases/<node-ip-2>
/registry/masterleases/<node-ip-3>
```



# [3. k8s对象数据]

k8s对象数据的格式

## [3.1. namespace]

```
/registry/namespaces/default
/registry/namespaces/game
/registry/namespaces/kube-node-lease
/registry/namespaces/kube-public
/registry/namespaces/kube-system
```



## [3.2. namespace级别对象

```
/registry/{resource}/{namespace}/{resource_name}
```



以下以常见k8s对象为例：

```
# deployment
/registry/deployments/default/game-2048
/registry/deployments/kube-system/prometheus-operator

# replicasets
/registry/replicasets/default/game-2048-c7d589ccf

# pod
/registry/pods/default/game-2048-c7d589ccf-8lsbw

# statefulsets
/registry/statefulsets/kube-system/prometheus-k8s

# daemonsets
/registry/daemonsets/kube-system/kube-proxy

# secrets
/registry/secrets/default/default-token-tbfmb

# serviceaccounts
/registry/serviceaccounts/default/default
```



**service**

```
# service
/registry/services/specs/default/game-2048

# endpoints
/registry/services/endpoints/default/game-2048
```



# [4. 读取数据value]

由于k8s默认etcd中的数据是通过protobuf格式存储，因此看到的key和value的值是一串字符串。

> alias ectl='ETCDCTL_API=3 etcdctl --endpoints=:2379,:2379,:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --key=/etc/kubernetes/pki/apiserver-etcd-client.key --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt'

```
# ectl get /registry/namespaces/test -w json |jq
{
  "header": {
    "cluster_id": 12113422651334595000,
    "member_id": 8381627376898157000,
    "revision": 12321629,
    "raft_term": 20
  },
  "kvs": [
    {
      "key": "L3JlZ2lzdHJ5L25hbWVzcGFjZXMvdGVzdA==",
      "create_revision": 11670741,
      "mod_revision": 11670741,
      "version": 1,
      "value": "azhzAAoPCgJ2MRIJTmFtZXNwYWNlElwKQgoEdGVzdBIAGgAiACokYWM1YmJjOTQtNTkxZi0xMWVhLWJiOTQtNmM5MmJmM2I3NmI1MgA4AEIICJuf3fIFEAB6ABIMCgprdWJlcm5ldGVzGggKBkFjdGl2ZRoAIgA="
    }
  ],
  "count": 1
}
```



其中key可以通过base64解码出来

```
echo "L3JlZ2lzdHJ5L25hbWVzcGFjZXMvdGVzdA==" | base64 --decode

# output
/registry/namespaces/test
```



value是值可以通过安装[etcdhelper](https://github.com/openshift/origin/tree/master/tools/etcdhelper)工具解析出来。

> alias ehelper='etcdhelper -key /etc/kubernetes/pki/apiserver-etcd-client.key -cert /etc/kubernetes/pki/apiserver-etcd-client.crt -cacert /etc/kubernetes/pki/etcd/ca.crt'

```
# ehelper get /registry/namespaces/test
/v1, Kind=Namespace
{
  "kind": "Namespace",
  "apiVersion": "v1",
  "metadata": {
    "name": "test",
    "uid": "ac5bbc94-591f-11ea-bb94-6c92bf3b76b5",
    "creationTimestamp": "2020-02-27T05:11:55Z"
  },
  "spec": {
    "finalizers": [
      "kubernetes"
    ]
  },
  "status": {
    "phase": "Active"
  }
}
```



# [5. 注意事项]

- 由于k8s的etcd数据为了性能考虑，默认通过`protobuf`格式存储，不要通过手动的方式去修改或添加k8s数据。
- 不推荐使用json格式存储etcd数据，如果需要json格式，可以使用`--storage-media-type=application/json`参数存储，参考：[kubernetes/kubernetes#44670](https://github.com/kubernetes/kubernetes/issues/44670)

# [6. 快捷命令]

由于etcdctl的命令需要添加很多认证参数和endpoints的参数，因此可以使用别名的方式来简化命令。

```
# etcdctl 
alias ectl='ETCDCTL_API=3 etcdctl --endpoints=<etcd-ip-1>:2379,<etcd-ip-2>:2379,<etcd-ip-3>:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt  --key=/etc/kubernetes/pki/apiserver-etcd-client.key  --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt'

# etcdhelper
alias ehelper='etcdhelper -key /etc/kubernetes/pki/apiserver-etcd-client.key -cert /etc/kubernetes/pki/apiserver-etcd-client.crt -cacert /etc/kubernetes/pki/etcd/ca.crt'
```



## [6.1. etcdhelper的使用]

`etcdhelper`文档参考：https://github.com/openshift/origin/tree/master/tools/etcdhelper

```
# 必要的认证参数
-key - points to master.etcd-client.key
-cert - points to master.etcd-client.crt
-cacert - points to ca.crt

# 命令操作参数
ls - list all keys starting with prefix
get - get the specific value of a key
dump - dump the entire contents of the etcd
```



示例

```
$ ehelper ls /registry/leases/
/registry/leases/kube-node-lease/<ip-1>
/registry/leases/kube-node-lease/<ip-2>
/registry/leases/kube-node-lease/<ip-3>

$ ehelper get <key>
```



# [7. RBAC]

附RBAC相关的key。

**clusterrolebindings**

```
/registry/clusterrolebindings/cluster-admin
/registry/clusterrolebindings/flannel
/registry/clusterrolebindings/galaxy
/registry/clusterrolebindings/helm
/registry/clusterrolebindings/kube-state-metrics
/registry/clusterrolebindings/kubeadm:kubelet-bootstrap
/registry/clusterrolebindings/kubeadm:node-autoapprove-bootstrap
/registry/clusterrolebindings/kubeadm:node-autoapprove-certificate-rotation
/registry/clusterrolebindings/kubeadm:node-proxier
/registry/clusterrolebindings/lbcf-controller
/registry/clusterrolebindings/prometheus-k8s
/registry/clusterrolebindings/prometheus-operator
/registry/clusterrolebindings/system:aws-cloud-provider
/registry/clusterrolebindings/system:basic-user
/registry/clusterrolebindings/system:controller:attachdetach-controller
/registry/clusterrolebindings/system:controller:certificate-controller
/registry/clusterrolebindings/system:controller:clusterrole-aggregation-controller
/registry/clusterrolebindings/system:controller:cronjob-controller
/registry/clusterrolebindings/system:controller:daemon-set-controller
/registry/clusterrolebindings/system:controller:deployment-controller
/registry/clusterrolebindings/system:controller:disruption-controller
/registry/clusterrolebindings/system:controller:endpoint-controller
/registry/clusterrolebindings/system:controller:expand-controller
/registry/clusterrolebindings/system:controller:generic-garbage-collector
/registry/clusterrolebindings/system:controller:horizontal-pod-autoscaler
/registry/clusterrolebindings/system:controller:job-controller
/registry/clusterrolebindings/system:controller:namespace-controller
/registry/clusterrolebindings/system:controller:node-controller
/registry/clusterrolebindings/system:controller:persistent-volume-binder
/registry/clusterrolebindings/system:controller:pod-garbage-collector
/registry/clusterrolebindings/system:controller:pv-protection-controller
/registry/clusterrolebindings/system:controller:pvc-protection-controller
/registry/clusterrolebindings/system:controller:replicaset-controller
/registry/clusterrolebindings/system:controller:replication-controller
/registry/clusterrolebindings/system:controller:resourcequota-controller
/registry/clusterrolebindings/system:controller:route-controller
/registry/clusterrolebindings/system:controller:service-account-controller
/registry/clusterrolebindings/system:controller:service-controller
/registry/clusterrolebindings/system:controller:statefulset-controller
/registry/clusterrolebindings/system:controller:ttl-controller
/registry/clusterrolebindings/system:coredns
/registry/clusterrolebindings/system:discovery
/registry/clusterrolebindings/system:kube-controller-manager
/registry/clusterrolebindings/system:kube-dns
/registry/clusterrolebindings/system:kube-scheduler
/registry/clusterrolebindings/system:node
/registry/clusterrolebindings/system:node-proxier
/registry/clusterrolebindings/system:public-info-viewer
/registry/clusterrolebindings/system:volume-scheduler
```



**clusterroles**

```
/registry/clusterroles/admin
/registry/clusterroles/cluster-admin
/registry/clusterroles/edit
/registry/clusterroles/flannel
/registry/clusterroles/kube-state-metrics
/registry/clusterroles/lbcf-controller
/registry/clusterroles/prometheus-k8s
/registry/clusterroles/prometheus-operator
/registry/clusterroles/system:aggregate-to-admin
/registry/clusterroles/system:aggregate-to-edit
/registry/clusterroles/system:aggregate-to-view
/registry/clusterroles/system:auth-delegator
/registry/clusterroles/system:aws-cloud-provider
/registry/clusterroles/system:basic-user
/registry/clusterroles/system:certificates.k8s.io:certificatesigningrequests:nodeclient
/registry/clusterroles/system:certificates.k8s.io:certificatesigningrequests:selfnodeclient
/registry/clusterroles/system:controller:attachdetach-controller
/registry/clusterroles/system:controller:certificate-controller
/registry/clusterroles/system:controller:clusterrole-aggregation-controller
/registry/clusterroles/system:controller:cronjob-controller
/registry/clusterroles/system:controller:daemon-set-controller
/registry/clusterroles/system:controller:deployment-controller
/registry/clusterroles/system:controller:disruption-controller
/registry/clusterroles/system:controller:endpoint-controller
/registry/clusterroles/system:controller:expand-controller
/registry/clusterroles/system:controller:generic-garbage-collector
/registry/clusterroles/system:controller:horizontal-pod-autoscaler
/registry/clusterroles/system:controller:job-controller
/registry/clusterroles/system:controller:namespace-controller
/registry/clusterroles/system:controller:node-controller
/registry/clusterroles/system:controller:persistent-volume-binder
/registry/clusterroles/system:controller:pod-garbage-collector
/registry/clusterroles/system:controller:pv-protection-controller
/registry/clusterroles/system:controller:pvc-protection-controller
/registry/clusterroles/system:controller:replicaset-controller
/registry/clusterroles/system:controller:replication-controller
/registry/clusterroles/system:controller:resourcequota-controller
/registry/clusterroles/system:controller:route-controller
/registry/clusterroles/system:controller:service-account-controller
/registry/clusterroles/system:controller:service-controller
/registry/clusterroles/system:controller:statefulset-controller
/registry/clusterroles/system:controller:ttl-controller
/registry/clusterroles/system:coredns
/registry/clusterroles/system:csi-external-attacher
/registry/clusterroles/system:csi-external-provisioner
/registry/clusterroles/system:discovery
/registry/clusterroles/system:heapster
/registry/clusterroles/system:kube-aggregator
/registry/clusterroles/system:kube-controller-manager
/registry/clusterroles/system:kube-dns
/registry/clusterroles/system:kube-scheduler
/registry/clusterroles/system:kubelet-api-admin
/registry/clusterroles/system:node
/registry/clusterroles/system:node-bootstrapper
/registry/clusterroles/system:node-problem-detector
/registry/clusterroles/system:node-proxier
/registry/clusterroles/system:persistent-volume-provisioner
/registry/clusterroles/system:public-info-viewer
/registry/clusterroles/system:volume-scheduler
/registry/clusterroles/view
```



**rolebindings**

```
/registry/rolebindings/kube-public/kubeadm:bootstrap-signer-clusterinfo
/registry/rolebindings/kube-public/system:controller:bootstrap-signer
/registry/rolebindings/kube-system/kube-proxy
/registry/rolebindings/kube-system/kube-state-metrics
/registry/rolebindings/kube-system/kubeadm:kubeadm-certs
/registry/rolebindings/kube-system/kubeadm:kubelet-config-1.14
/registry/rolebindings/kube-system/kubeadm:nodes-kubeadm-config
/registry/rolebindings/kube-system/system::extension-apiserver-authentication-reader
/registry/rolebindings/kube-system/system::leader-locking-kube-controller-manager
/registry/rolebindings/kube-system/system::leader-locking-kube-scheduler
/registry/rolebindings/kube-system/system:controller:bootstrap-signer
/registry/rolebindings/kube-system/system:controller:cloud-provider
/registry/rolebindings/kube-system/system:controller:token-cleaner
```



**roles**

```
/registry/roles/kube-public/kubeadm:bootstrap-signer-clusterinfo
/registry/roles/kube-public/system:controller:bootstrap-signer
/registry/roles/kube-system/extension-apiserver-authentication-reader
/registry/roles/kube-system/kube-proxy
/registry/roles/kube-system/kube-state-metrics-resizer
/registry/roles/kube-system/kubeadm:kubeadm-certs
/registry/roles/kube-system/kubeadm:kubelet-config-1.14
/registry/roles/kube-system/kubeadm:nodes-kubeadm-config
/registry/roles/kube-system/system::leader-locking-kube-controller-manager
/registry/roles/kube-system/system::leader-locking-kube-scheduler
/registry/roles/kube-system/system:controller:bootstrap-signer
/registry/roles/kube-system/system:controller:cloud-provider
/registry/roles/kube-system/system:controller:token-cleaner
```

# Etcd-Operator的使用

> 本文主要介绍etcd-operator的部署及使用

# [1. 部署RBAC]

下载[create_role.sh](https://github.com/coreos/etcd-operator/blob/master/example/rbac/create_role.sh)、[cluster-role-binding-template.yaml](https://github.com/coreos/etcd-operator/blob/master/example/rbac/cluster-role-binding-template.yaml)、[cluster-role-template.yaml](https://github.com/coreos/etcd-operator/blob/master/example/rbac/cluster-role-template.yaml)

例如：

```
|-- cluster-role-binding-template.yaml
|-- cluster-role-template.yaml
|-- create_role.sh

# 部署rbac
kubectl create ns operator
bash create_role.sh --namespace=operator  # namespace与etcd-operator的ns一致
```



示例：

```
bash create_role.sh --namespace=operator
+ ROLE_NAME=etcd-operator
+ ROLE_BINDING_NAME=etcd-operator
+ NAMESPACE=default
+ for i in '"$@"'
+ case $i in
+ NAMESPACE=operator
+ echo 'Creating role with ROLE_NAME=etcd-operator, NAMESPACE=operator'
Creating role with ROLE_NAME=etcd-operator, NAMESPACE=operator
+ sed -e 's/<ROLE_NAME>/etcd-operator/g' -e 's/<NAMESPACE>/operator/g' cluster-role-template.yaml
+ kubectl create -f -
clusterrole.rbac.authorization.k8s.io/etcd-operator created
+ echo 'Creating role binding with ROLE_NAME=etcd-operator, ROLE_BINDING_NAME=etcd-operator, NAMESPACE=operator'
Creating role binding with ROLE_NAME=etcd-operator, ROLE_BINDING_NAME=etcd-operator, NAMESPACE=operator
+ sed -e 's/<ROLE_NAME>/etcd-operator/g' -e 's/<ROLE_BINDING_NAME>/etcd-operator/g' -e 's/<NAMESPACE>/operator/g' cluster-role-binding-template.yaml
+ kubectl create -f -
clusterrolebinding.rbac.authorization.k8s.io/etcd-operator created
```



## [1.1. create_role.sh 脚本]

create_role.sh有三个入参，可以指定--namespace参数，该参数与etcd-operator部署的namespace应一致。默认为default。

```
#!/usr/bin/env bash
set -o errexit
set -o nounset
set -o pipefail

ETCD_OPERATOR_ROOT=$(dirname "${BASH_SOURCE}")/../..

print_usage() {
  echo "$(basename "$0") - Create Kubernetes RBAC role and role bindings for etcd-operator
Usage: $(basename "$0") [options...]
Options:
  --role-name=STRING         Name of ClusterRole to create
                               (default=\"etcd-operator\", environment variable: ROLE_NAME)
  --role-binding-name=STRING Name of ClusterRoleBinding to create
                               (default=\"etcd-operator\", environment variable: ROLE_BINDING_NAME)
  --namespace=STRING         namespace to create role and role binding in. Must already exist.
                               (default=\"default\", environment variable: NAMESPACE)
" >&2
}

ROLE_NAME="${ROLE_NAME:-etcd-operator}"
ROLE_BINDING_NAME="${ROLE_BINDING_NAME:-etcd-operator}"
NAMESPACE="${NAMESPACE:-default}"

for i in "$@"
do
case $i in
    --role-name=*)
    ROLE_NAME="${i#*=}"
    ;;
    --role-binding-name=*)
    ROLE_BINDING_NAME="${i#*=}"
    ;;
    --namespace=*)
    NAMESPACE="${i#*=}"
    ;;
    -h|--help)
      print_usage
      exit 0
    ;;
    *)
      print_usage
      exit 1
    ;;
esac
done

echo "Creating role with ROLE_NAME=${ROLE_NAME}, NAMESPACE=${NAMESPACE}"
sed -e "s/<ROLE_NAME>/${ROLE_NAME}/g" \
  -e "s/<NAMESPACE>/${NAMESPACE}/g" \
  "cluster-role-template.yaml" | \
  kubectl create -f -

echo "Creating role binding with ROLE_NAME=${ROLE_NAME}, ROLE_BINDING_NAME=${ROLE_BINDING_NAME}, NAMESPACE=${NAMESPACE}"
sed -e "s/<ROLE_NAME>/${ROLE_NAME}/g" \
  -e "s/<ROLE_BINDING_NAME>/${ROLE_BINDING_NAME}/g" \
  -e "s/<NAMESPACE>/${NAMESPACE}/g" \
  "cluster-role-binding-template.yaml" | \
  kubectl create -f -
```



## [1.2. cluster-role-binding-template.yaml](https://github.com/huweihuang/kubernetes-notes/blob/master/etcd/etcd-operator-usage.md#12-cluster-role-binding-templateyaml)

```
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: <ROLE_BINDING_NAME>
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: <ROLE_NAME>
subjects:
- kind: ServiceAccount
  name: default
  namespace: <NAMESPACE>
```



## [1.3. cluster-role-template.yaml](https://github.com/huweihuang/kubernetes-notes/blob/master/etcd/etcd-operator-usage.md#13-cluster-role-templateyaml)

```
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: <ROLE_NAME>
rules:
- apiGroups:
  - etcd.database.coreos.com
  resources:
  - etcdclusters
  - etcdbackups
  - etcdrestores
  verbs:
  - "*"
- apiGroups:
  - apiextensions.k8s.io
  resources:
  - customresourcedefinitions
  verbs:
  - "*"
- apiGroups:
  - ""
  resources:
  - pods
  - services
  - endpoints
  - persistentvolumeclaims
  - events
  verbs:
  - "*"
- apiGroups:
  - apps
  resources:
  - deployments
  verbs:
  - "*"
# The following permissions can be removed if not using S3 backup and TLS
- apiGroups:
  - ""
  resources:
  - secrets
  verbs:
  - get
```



# [2. 部署etcd-operator]

```
kubectl create -f etcd-operator.yaml
```



etcd-operator.yaml如下：

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: etcd-operator
  namespace: operator   # 与rbac指定的ns一致
  labels:
    app: etcd-operator
spec:
  replicas: 1
  selector:
    matchLabels:
      app: etcd-operator
  template:
    metadata:
      labels:
        app: etcd-operator
    spec:
      containers:
      - name: etcd-operator
        image: registry.cn-shenzhen.aliyuncs.com/huweihuang/etcd-operator:v0.9.4
        command:
        - etcd-operator
        # Uncomment to act for resources in all namespaces. More information in doc/user/clusterwide.md
        - -cluster-wide
        env:
        - name: MY_POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: MY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
```



查看CRD

```
#kubectl get customresourcedefinitions
NAME                                       CREATED AT
etcdclusters.etcd.database.coreos.com      2020-08-01T13:02:18Z
```



查看etcd-operator的日志是否OK。

```
k logs -f etcd-operator-545df8d445-qpf6n -n operator
time="2020-08-01T13:02:18Z" level=info msg="etcd-operator Version: 0.9.4"
time="2020-08-01T13:02:18Z" level=info msg="Git SHA: c8a1c64"
time="2020-08-01T13:02:18Z" level=info msg="Go Version: go1.11.5"
time="2020-08-01T13:02:18Z" level=info msg="Go OS/Arch: linux/amd64"
time="2020-08-01T13:02:18Z" level=info msg="Event(v1.ObjectReference{Kind:\"Endpoints\", Namespace:\"operator\", Name:\"etcd-operator\", UID:\"7de38cff-1b7b-4bf2-9837-473fa66c9366\", APIVersion:\"v1\", ResourceVersion:\"41195930\", FieldPath:\"\"}): type: 'Normal' reason: 'LeaderElection' etcd-operator-545df8d445-qpf6n became leader"
```



以上内容表示etcd-operator运行正常。

# [3. 部署etcd集群]

```
kubectl create -f etcd-cluster.yaml
```



> 当开启clusterwide则etcd集群与etcd-operator的ns可不同。

etcd-cluster.yaml

```
apiVersion: "etcd.database.coreos.com/v1beta2"
kind: "EtcdCluster"
metadata:
  name: "default-etcd-cluster"
  ## Adding this annotation make this cluster managed by clusterwide operators
  ## namespaced operators ignore it
  annotations:
    etcd.database.coreos.com/scope: clusterwide
  namespace: etcd   # 此处的ns表示etcd集群部署在哪个ns下
spec:
  size: 3
  version: "v3.3.18"
  repository: registry.cn-shenzhen.aliyuncs.com/huweihuang/etcd
  pod:
    busyboxImage: registry.cn-shenzhen.aliyuncs.com/huweihuang/busybox:1.28.0-glibc
```



查看集群部署结果

```
$ kgpo -n etcd
NAME                              READY   STATUS    RESTARTS   AGE
default-etcd-cluster-b6phnpf8z8   1/1     Running   0          3m3s
default-etcd-cluster-hhgq4sbtgr   1/1     Running   0          109s
default-etcd-cluster-ttfh5fj92b   1/1     Running   0          2m29s
```



# [4. 访问etcd集群]

查看service

```
$ kgsvc -n etcd
NAME                          TYPE        CLUSTER-IP        EXTERNAL-IP   PORT(S)             AGE
default-etcd-cluster          ClusterIP   None              <none>        2379/TCP,2380/TCP   5m37s
default-etcd-cluster-client   ClusterIP   192.168.255.244   <none>        2379/TCP            5m37s
```



使用service地址访问

```
# 查看集群健康状态
$ ETCDCTL_API=3 etcdctl --endpoints 192.168.255.244:2379 endpoint health
192.168.255.244:2379 is healthy: successfully committed proposal: took = 1.96126ms

# 写数据
$ ETCDCTL_API=3 etcdctl --endpoints 192.168.255.244:2379 put foo bar
OK

# 读数据
$ ETCDCTL_API=3 etcdctl --endpoints 192.168.255.244:2379 get foo
foo
bar
```



# [5. 销毁etcd-operator]

```
kubectl delete -f example/deployment.yaml
kubectl delete endpoints etcd-operator
kubectl delete crd etcdclusters.etcd.database.coreos.com
kubectl delete clusterrole etcd-operator
kubectl delete clusterrolebinding etcd-operator
```