# Redis支持的数据结构：

Redis支持诸如字符串（strings）、哈希（hashes）、列表（lists）、集合（sets）、带范围查询的排序集合（sorted sets）、位图（bitmaps）、hyperloglogs、带半径查询和流的地理空间索引等数据结构（geospatial indexes）。

# Redis应用场景：

缓存系统，减轻主数据库（MySQL）的压力。
计数场景，比如微博、抖音中的关注数和粉丝数。
热门排行榜，需要排序的场景特别适合使用ZSET。
利用LIST可以实现队列的功能。



| **类型**             | **简介**                                               | **特性**                                                     | **场景**                                                     |
| -------------------- | ------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| String(字符串)       | 二进制安全                                             | 可以包含任何数据,比如jpg图片或者序列化的对象,一个键最大能存储512M | ---                                                          |
| Hash(字典)           | 键值对集合,即编程语言中的Map类型                       | 适合存储对象,并且可以像数据库中update一个属性一样只修改某一项属性值(Memcached中需要取出整个字符串反序列化成对象修改完再序列化存回去) | 存储、读取、修改用户属性                                     |
| List(列表)           | 链表(双向链表)                                         | 增删快,提供了操作某一段元素的API                             | 1,最新消息排行等功能(比如朋友圈的时间线) 2,消息队列          |
| Set(集合)            | 哈希表实现,元素不重复                                  | 1、添加、删除,查找的复杂度都是O(1) 2、为集合提供了求交集、并集、差集等操作 | 1、共同好友 2、利用唯一性,统计访问网站的所有独立ip 3、好友推荐时,根据tag求交集,大于某个阈值就可以推荐 |
| Sorted Set(有序集合) | 将Set中的元素增加一个权重参数score,元素按score有序排列 | 数据插入集合时,已经进行天然排序                              | 1、排行榜 2、带权重的                                        |

# String（字符串）

string 是 redis 最基本的类型，你可以理解成与 Memcached 一模一样的类型，一个 key 对应一个 value。

string 类型是二进制安全的。意思是 redis 的 string 可以包含任何数据。比如jpg图片或者序列化的对象。

string 类型是 Redis 最基本的数据类型，string 类型的值最大能存储 512MB。

## 实例

redis 127.0.0.1:6379>set runoob "hello world"

127.0.0.1:6379> get runoob

 

Redis 的 **SET** 和 **GET** 命令。键为 runoob，对应的值为 "hello world"。

**注意：**一个键最大能存储 512MB。

 

 

# Hash（哈希）

Redis hash 是一个键值(key=>value)对集合。

Redis hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。

127.0.0.1:6379> hmset hmvalue field1 "name" field2 "liming"

127.0.0.1:6379> hget hmvalue field1

"name"

 

 

 

实例中我们使用了 Redis **HMSET, HGET** 命令，**HMSET** 设置了两个 **field=>value** 对, HGET 获取对应 **field** 对应的 **value**。

每个 hash 可以存储 232 -1 键值对（40多亿）。

 

 

 

 

# List（列表）

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。



127.0.0.1:6379> lpush list 1

(integer) 1

127.0.0.1:6379> LRANGE list 0 10

1) "4"
2) "2"
3) "1"



列表最多可存储 232 - 1 元素 (4294967295, 每个列表可存储40多亿)。





# Set（集合）

Redis 的 Set 是 string 类型的无序集合。

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。

## sadd 命令

添加一个 string 元素到 key 对应的 set 集合中，成功返回 1，如果元素已经在集合中返回 0。

sadd key member

 

 

127.0.0.1:6379> sadd setgather one

(integer) 1

127.0.0.1:6379> sadd setgather two

(integer) 1

127.0.0.1:6379> sadd setgather there

(integer) 1

127.0.0.1:6379> smembers setgather

1) "two"
2) "there"
3) "one"

 

 

 

 

## zset(sorted set：有序集合)

Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

zset的成员是唯一的,但分数(score)却可以重复。

### zadd 命令

添加元素到集合，元素在集合中存在则插入到score后

zadd key score member 

 

 

 

 

127.0.0.1:6379> zadd zsetgather 0 redis

(integer) 1

127.0.0.1:6379> zadd zsetgather 1 redis2

(integer) 1

127.0.0.1:6379> zadd zsetgather 0 update

(integer) 1

127.0.0.1:6379> ZRANGE zsetgather 0 3

1) "redis"
2) "update"
3) "redis2"