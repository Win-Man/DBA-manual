# MySQL 与 PostgreSQL 对比
## PostgreSQL的优势
* PGSQL没有CPU核心数限制，MySQL能用128核CPU。
* PG主表采用堆表存放，MySQL采用索引组织表，能够支持比MySQL更大的数据量。
* PG的主备复制属于物理复制(完胜逻辑复制，维护简单)，相对于MySQL基于binlog的逻辑复制（sql_log_bin、binlog_format等参数设置不正确都会导致主从不一致），数据的一致性更加可靠，复制性能更高，对主机性能的影响也更小。
* MySQL的存储引擎插件化机制，存在锁机制复杂影响并发的问题，而PG不存在这个机制。
* PGSQL支持 JIT 执行计划即时编译，使用LLVM编译器，MySQL不支持执行计划即时编译。
* PGSQL一共有255个参数，用到的大概是80个，参数比较稳定，用上个大版本配置文件也可以启动当前大版本数据库(版本兼容性好)，而MySQL一共有707个参数，用到的大概是180个，参数不断增加，就算小版本也会增加参数，大版本之间会有部分参数不兼容情况。参数多引起维护成本加大。比如：PGSQL系统自动设置从库默认只读，不需要人工添加配置，维护简单，MySQL从库需要手动设置参数super_read_only=on，让从库设置为只读。
## MySQL的优势
* MySQL数据库查看sql的执行计划更直观易懂。
* MySQL采用索引组织表，这种存储方式非常适合基于主键匹配的查询、删改操作，但是对表结构设计存在约束。
* MySQL的优化器较简单，系统表、运算符、数据类型的实现都很精简，非常适合简单的查询操作。
* MySQL分区表的实现要优于PG的基于继承表的分区实现，主要体现在分区个数达到上千上万后的处理性能差异较大。