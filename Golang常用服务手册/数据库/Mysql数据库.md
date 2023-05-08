**关系型数据库**：用表来存储一类数据
**存储引擎**：类似于电脑和内存条
**常见的存储引擎**：MyISAM和InnoDB
**MyISAM**：查询速度快、支持表锁、不支持事务
**InnoDB**：整体速度快、支持表锁和行锁
**事务**：把一组SQL操作当做一个整体
**事务特点**：ACID
1、原子性：事务要么失败，要么成功，没有中间状态
2、一致性：数据库的完整性没有被破坏
3、隔离性：事务之间操作是相互隔离的
4、持久性：事务操作的结果是不会丢失的

```go
func (db *DB) Begin() (*Tx, error)：开始事务
func (tx *Tx) Commit() error：提交事务
func (tx *Tx) Rollback() error：回滚事务
```



| 条件   | 解释                                                         |
| ------ | ------------------------------------------------------------ |
| 原子性 | 一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。 |
| 一致性 | 在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。 |
| 隔离性 | 数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。 |
| 持久性 | 事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。 |

索引：
索引原理：B树和B+树

# 连接
Go语言中的database/sql包提供了保证SQL或类SQL数据库的泛用接口，并不提供具体的数据库驱动。
使用database/sql包时必须注入（至少）一个数据库驱动。
Mysql下载依赖：go get -u github.com/go-sql-driver/mysql

func Open(driverName, dataSourceName string) (*DB, error)
Open打开一个指定dirverName的数据库，dataSourceName指定数据源，一般至少包括数据库文件名和连接必要的信息。
*DB是表示连接的数据库对象（结构体实例），它保存了连接数据库相关的所有信息。它内部维护着一个具有零到多个底层连接的连接池，它可以安全地被多个goroutine同时使用。

#Open函数可能只是验证其参数格式是否正确，实际上并不创建与数据库的连接。如果要检查数据源的名称是否真实有效，应该调用db.Ping()方法。
#返回的DB对象可以安全地被多个goroutine并发使用，并且维护其自己的空闲连接池。因此，Open函数应该仅被调用一次，很少需要关闭这个DB对象。

# 连接池
func (db *DB) SetMaxOpenConns(n int)：设置与数据库建立连接的最大数目。
如果n大于0且小于最大闲置连接数，会将最大闲置连接数减小到匹配最大开启连接数的限制。 如果n<=0，不会限制最大开启连接数，默认为0（无限制）。

func (db *DB) SetMaxIdleConns(n int)：设置连接池中的最大闲置连接数。
如果n大于最大开启连接数，则新的最大闲置连接数会减小到匹配最大开启连接数的限制。 如果n<=0，不会保留闲置连接。


# SQL操作
func (db *DB) QueryRow(query string, args ...interface{}) *Row
单行查询db.QueryRow()执行一次查询，并期望返回最多一行结果（即Row）。
QueryRow总是返回非nil的值，直到返回值的Scan方法被调用时，才会返回被延迟的错误。（如：未找到结果）

func (db *DB) Query(query string, args ...interface{}) (*Rows, error)
多行查询db.Query()执行一次查询，返回多行结果（即Rows），一般用于执行select命令。参数args表示query中的占位参数。

插入、更新和删除操作都使用Exec方法。
func (db *DB) Exec(query string, args ...interface{}) (Result, error)
Exec执行一次命令（包括查询、删除、更新、插入等），返回的Result是对已执行的SQL命令的总结。参数args表示query中的占位参数。
Exec().LastInsertId() // 返回新插入数据的主键id
Exec()..RowsAffected() // 返回被操作数据的行数

# MySQL预处理
什么是预处理？**普通SQL语句执行过程**：
客户端对SQL语句进行占位符替换得到完整的SQL语句。
客户端发送完整SQL语句到MySQL服务端
MySQL服务端执行完整的SQL语句并将结果返回给客户端。

**预处理执行过程：**
把SQL语句分成两部分，命令部分与数据部分。
先把命令部分发送给MySQL服务端，MySQL服务端进行SQL预处理。
然后把数据部分发送给MySQL服务端，MySQL服务端对SQL语句进行占位符替换。
MySQL服务端执行完整的SQL语句并将结果返回给客户端。

**为什么要预处理？**
优化MySQL服务器重复执行SQL的方法，可以提升服务器性能，提前让服务器编译，一次编译多次执行，节省后续编译的成本。

func (db *DB) Prepare(query string) (*Stmt, error)
Prepare预处理方法会先将sql语句发送给MySQL服务端，返回一个准备好的状态用于之后的查询和命令。返回值可以同时执行多个查询和命令。

| 数据库     | 占位符语法 |
| ---------- | ---------- |
| MySQL      | ?          |
| PostgreSQL | $1, $2等   |
| SQLite     | ? 和$1     |
| Oracle     | :name      |


# sqlx包
介绍：sqlx可以认为是Go语言内置database/sql的超集，它在优秀的内置database/sql基础上提供了一组扩展。
这些扩展中除了大家常用来查询的Get(dest interface{}, ...) error和Select(dest interface{}, ...) error外还有很多其他强大的功能。
sqlx安装：`go get github.com/jmoiron/sqlx`

DB.NamedExec方法用来绑定SQL语句与结构体或map中的同名字段。

NamedQuery方法与DB.NamedExec同理，这里是支持查询。

事务操作，我们可以使用sqlx中提供的db.Beginx()和tx.Exec()方法

sqlx.In批量插入
