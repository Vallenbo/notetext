Redis是一个开源的内存数据库，Redis提供了多种不同类型的数据结构，很多业务场景下的问题都可以很自然地映射到这些数据结构上。除此之外，通过复制、持久化和客户端分片等特性，我们可以很方便地将Redis扩展成一个能够包含数百GB数据、每秒处理上百万次请求的系统。

# Redis支持的数据结构：
Redis支持诸如字符串（strings）、哈希（hashes）、列表（lists）、集合（sets）、带范围查询的排序集合（sorted sets）、位图（bitmaps）、hyperloglogs、带半径查询和流的地理空间索引等数据结构（geospatial indexes）。

# Redis应用场景：
缓存系统，减轻主数据库（MySQL）的压力。
计数场景，比如微博、抖音中的关注数和粉丝数。
热门排行榜，需要排序的场景特别适合使用ZSET。
利用LIST可以实现队列的功能。

# go-redis库
安装：区别于另一个比较常用的Go语言redis client库：redigo，我们这里采用https://github.com/go-redis/redis连接Redis数据库并进行操作，因为go-redis支持连接哨兵及集群模式的Redis。
下载并安装：go get -u github.com/go-redis/redis

# V8新版本相关
最新版本的go-redis库的相关命令都需要传递context.Context参数
go get -u github.com/go-redis/redis/v8

