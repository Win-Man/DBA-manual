# PG 目录及配置文件说明

## 环境变量
```
export PGHOME=/usr/local/pgsql/
export PGUSER=postgres
export PGPORT=5432
export PGDATA=/app/pgsql/data
export PGLOG=/app/pgsql/log/postgres.log
export PATH=$PGHOME/bin:$PATH:$HOME/bin
export LD_LIBRARY_PATH=$PGHOME/lib:$LD_LIBRARY_PATH
PATH=/usr/local/pgsql/bin:$PATH
export PATH
```

## 安装目录 PGHOME

## 数据目录 PGDATA

pg_hba.conf：#认证配置文件，配置了允许哪些IP访问数据库，及认证方式等信息。
pg_ident.conf：#"ident"认证方式的用户映射文件。
PG_VERSION：#记录了数据库版本号信息。
postgresql.auto.conf：#作用同 postgresql.conf ，优先级高于 postgresql.conf，在数据库中通过alter命令更改的参数记录在此文件中。
postgresql.conf：#数据库实例主配置文件，基本上所有的数据库参数配置都在此文件中。
postmaster.opts：#记录数据库启动命令。
postmaster.pid：#数据库进程文件，数据库启动时被创建，关闭时消失。
base：#该目录包含数据库用户所创建的各个数据库，同时也包括postgres、template0和template1的pg_defaulttablespace。
global：#该目录包含集群范围的各个表和相关视图。(pg_database、pg_tablespace）
pg_dynshmem：#该目录包含动态共享内存子系统使用的文件。
pg_commit_ts：#该目录包含已提交事务的时间。
pg_logical：#该目录包含逻辑解码的状态数据。
pg_multixact：#该目录包含多事务状态数据。（等待锁定的并发事务）
pg_notify：#该目录包含LISTEN/NOTIFY状态数据。
pg_replslot：#该目录包含复制槽数据。
pg_snapshots：#该目录包含导出的快照。
pg_stat：#该目录包含统计子系统的永久文件。
pg_stat_tmp：#该目录包含统计子系统的临时文件。
pg_subtrans：#该目录包含子事务状态数据。
pg_tblspc：#该目录包含表空间的符号链接。
pg_twophase：#该目录包含预备事务的状态文件。
pg_wal：#该目录包含wal日志。
pg_xact：#该目录包含事务提交状态数据。

## 日志目录 PGLOG

## 配置文件 postgresql.conf 

待抄录：
https://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247541440&idx=2&sn=3a677522547ded1680f2875e7a1e5b1e&chksm=e91847dcde6fceca0d1470d66c299391226f2eef9361ddea99d02f711a3b5fdf7a12067a13d6&scene=21#wechat_redirect
