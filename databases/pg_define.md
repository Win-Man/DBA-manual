# PostgreSQL 基本概念

## PostgreSQL 特征

* 多版本并发控制：PostgreSQL 使用多版本并发控制（MVCC）系统进行并发控制，该系统向每个用户提供了一个数据库的“快照”，用户在事务内所作的每个修改，对于其他用户都不可见，直到该事务成功提交
* 数据类型：包括文本、任意精度的数值数组、JSON数据、枚举类型、XML数据等。
* 全文检索：通过 Tsearch2 或 OpenFTS .
* NoSQL: JSON，JSONNB，XML，HStore 原生支持，甚至 NoSQL 数据库的外部数据包装器
* 数据仓库：能平滑迁移至同属 PostgreSQL 生态的 GreenPlum，DeepGreen 等，使用 FDW 进行 ETL
* 函数：通过函数，可以在数据库服务器端执行指令程序
* 索引：用户可以自定义索引方法，或使用内置的 B 树，哈希表与 GiST 索引
* 触发器： 触发器是由 SQL 语句查询所触发的事件。
* 规则：规则（RULE）允许一个查询能被重写，通常用来实现对视图（VIEW）的操作，如插入，更新，删除
* 继承：PostrgeSQL 实现了表继承，一个表可以从 0 个或者多个其他表继承，对一个表的查询则可以使用一个表的所有行或者该表上的所有行加上它所有的后代表

## PostgreSQL 数据库对象
### database
每个 PG 服务可以包含多个独立的 database

### schema
大多数对象隶属于 schema, schema 又隶属于某个 database。
在创建一个新的 database 时，PG 会自动为其创建一个名为 public 的 schema 。如果未设置 sear_path 变量，那么 PG 会将你创建的所有对象默认放入 public schema 中。表数量较多的情况下，建议将表放入不同的 schema 中。

### 表
任何一个数据库中，表都是最核心的对象类型。在 PG 中，表首先属于某个 schema ，而 schema 又属于某个 database ，这样就构成一种三级存储结构。PG 的表支持两种很强大的功能。第一种是继承，即一张表可以有父表和子表，这种层次化的结构可以极大的简化数据库设计，还可以省掉大量的重复查询代码。第二种是创建一张表的同时，系统会自动为此表创建一种对应的自定义数据类型。

## PostgreSQL 整体架构

### 进程架构

PostgreSQL 是一个多进程架构的 C/S 模式的关系统数据库管理系统。PG 数据库中的一系列进程组合起来就是 PostgreSQL 服务端。
* postgres server 进程：PG 数据库中所有进程的父进程
* backend 进程：每个客户端对于一个 backend 进程，处理这个客户端中的所有请求。
* background 进程：包含多个后台进程，比如做脏块刷盘的 BACKGROUND WRITER 进程，做垃圾清理的 AUTOVACUUM 进程，做检查点的 CHECKPOINT 进程等
* replication 相关进程： 处理流复制的进程
* background worker 进程：PG 9.3 版本增加，执行由用户自定义开发的逻辑。

![](https://raw.githubusercontent.com/Win-Man/pic-storage/master/img/202405231450180.png)

#### Postgres Server Process

postgres server process 是所有 PG 进程的父进程，在以前的版本中称为 postmaster。当使用 pg_ctl start 启动数据库时，这个进程就被启动了，然后它会启动一个共享内存 shared memory ，启动多个 background 后台进程，启动复制相关进程，如有需要也启动 background worker progress ，然后等待客户端的连接。

当接收到一个客户端连接时，它就会启动一个 backend progress ，专门服务于这个客户端。

postgres server process 通常有一个对应的监听端口，默认 5432.

#### Backend Process
backend process 也称为 postgres 进程，是由上面的 postgres server process 启动的用于服务于对应的客户端，通过 TCP 协议和客户端进行通信。
PG 允许多个客户端同时连接数据库，由 max_connections 参数控制最大并发连接数，默认是 100 。

#### Background Process 
background process 后台进程有多个，每个进程负责一个模块或是一类任务:
* background writer: 负责将共享内存中的脏页刷到持久化存储，在 PG 9.1 或更早版本中也负责 checkpoint 的工作
* checkpointer: 在 PG9.2 及之后的版本中，引入用来单独处理 checkpoint 任务
* autovacuum launcher: 向 postgres server 进程请求启动 autovacuum worker 进程，用来进行定期的 aotuvacuum 任务
* WAL writer: 负责将共享内存中的 WAL 周期性的疏导持久化存储
* statistics collector: 负责收集统计信息，如 pg_stat_activity 和 pg_stat_database 
* logging collector: 负责把错误消息写到日志文件
* archiver: 负责归档日志

### 内存架构

PG 中的内存主要分为两类：
* 本地内存区：用于每个 backend process 内部使用，每个客户端连接对应一个本地内存区
* 共享内存区：所有 PG 进程共享使用

#### 本地内存区
* work_mem: 进行 ORDER BY/DISTINCT 语句相关的排序工作，以及像 merge 关联或 hash 关联之类的关联工作
* maintenance_work_mem: 一些维护之类的工作，比如 vacuum、reindex 之类
* temp_buffers: 存储临时表相关的工作

#### 共享内存区
* shared buffer pool: 从持久化存储中把表及索引数据加载到这个区域直接操作数据
* WAL bufer: 为了保证数据不丢失，PG 支持 WAL 机制，WAL 数据（XLOG）就是事务相关的日志，WAL buffer 是 WAL 日志落盘前的缓冲区
* commit log: commit log(CLOG) 保存所有事务的状态，如 in_process、commited、aborted

除此之外，共享内存区还包括一些其他的子区域：
* 用于多种访问控制的内存区域
* 用于多种后台进程如 checkpointer、vacuum 的内存区域
* 用于事务处理的区域如 savepoint 、二阶段提交


