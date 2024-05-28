# PostgreSQL 运维命令

## 连接 PG 
```sql
psql -h ${ip} -p ${port} -U ${user} -d ${数据库名} -W 
```

常用命令
```sql
\? #所有命令帮助
\l #列出所有数据库
\d #列出数据库中所有表
\dt #列出数据库中所有表
\d [table_name] #显示指定表的结构
\di #列出数据库中所有 index
\dv #列出数据库中所有 view
\h #sql命令帮助
\q #退出连接
\c [database_name] #切换到指定的数据库
\c #显示当前数据库名称和用户
\conninfo #显示客户端的连接信息
\du #显示所有用户
\dn #显示数据库中的schema
\encoding #显示字符集
select version(); #显示版本信息
\i testdb.sql #执行sql文件
\x #扩展展示结果信息，相当于MySQL的\G
\o /tmp/test.txt #将下一条sql执行结果导入文件中
```

## 用户管理

### 创建用户
```sql
create user user1 password 'password1';
# 设置只读权限
alter user user1 default_transaction_read_only = on;
#设置可操作的数据库
grant all on database dbname to user1;

# 授权
grant select on all tables in schema public to user1;

# 授权
grant all on table public.user to mydata;
grant select,update,insert,delete on table public.user to mydata_dml;
grant select on table public.user to mydata_qry;
```

### 删除用户
```sql
# 撤回在 public 模式下的权限
revoke select on all tables in schema public from user1;

# 撤回在 information_schema 模式下的权限
revoke select on all tables in schema information_schema from user1;

# 撤回在 pg_catalog 模式下的权限
revoke select on all tables in schema pg_catalog from user1;

# 撤回对数据库的操作权限
revoke all on database dbname from user1;

# 删除用户
drop user user1;

```

### 权限管理
```sql
# 设置只读权限
alter user user1 set default_transaction_read_only = on;

# 设置可操作的数据库
alter all on database dbname to user1;

# 设置可操作的模式和权限
grant select,insert,update,delete on all tables in schema public to user1;

# 撤回在 public 模式下的权限
revoke select on all tables in schema public from user1;

# 撤回在 information_schema 模式下的权限
revoke select on all tables in schema information_schema from user1;

# 撤回在 pg_catalog 模式下的权限
revoke select on all tables in schema pg_catalog from user1;

# 撤回对数据库的操作权限
revoke all on database dbname from user1;

```

## 模式 schema 管理

```sql
CREATE SCHEMA myschema.mytable(

);
```

创建和当前用户同名模式（schema）
用户名与 schema 同名，且用户具有访问 schema 的权限，用户连入数据库时，默认即为当前 schema 。
```sql
create schema AUTHORIZATION CURRENT_USER;
```

自定义创建模式（schema）
```sql
create schema 模式名称;
```

查看数据库下所有 schema
```sql
select * from information_schema.schemata;
```

## 数据库管理
查询所有数据库
```sql
select datname from pg_database;
```

创建数据库
```sql
create database 数据库名 owner 所属用户 encoding UTF8;
```

创建完数据库，需要切换到数据库下，创建和当前用户同名 shchema ，删除数据库后 schema 也会一并删除
```sql
-- 重新登陆到新数据库下，执行以下语句
create schema AUTHORIZATION CURRENT_USER;
```

删除数据库
```sql
drop database 数据库名;
```
删库之前需要关闭所有会话

关闭数据库所有会话
```sql
SELECT pg_terminate_backend(pg_stat_activity.pid)
FROM pg_stat_activity
WHERE datname='mydb' AND pid<>pg_backend_pid();
```

## 表管理
查询 schema 中所有表
```sql
select table_name from information_schema.tables where table_schema = 'myuser';
```

创建表
```sql
CREATE TABLE public.t_user (
  "id" BIGSERIAL NOT NULL,
  "username" VARCHAR(64) NOT NULL,
  "password" VARCHAR(64) NOT NULL,
  "create_time" TIMESTAMP(0) default CURRENT_TIMESTAMP not null,
  "update_time" TIMESTAMP(0) default CURRENT_TIMESTAMP not null
);
-- 注释
COMMENT ON TABLE public.t_user IS '用户表';
COMMENT ON COLUMN public.t_user.id IS '主键';
COMMENT ON COLUMN public.t_user.username IS '用户名';
COMMENT ON COLUMN public.t_user.password IS '密码';
COMMENT ON COLUMN public.t_user.create_time IS '创建时间';
COMMENT ON COLUMN public.t_user.update_time IS '更新时间';
-- 创建自增序列
alter sequence "t_user_ID_seq" restart with 1 increment by 1;
-- 创建主键序列
drop index if exists "t_user_pkey";
alter table "t_user" add constraint "t_user_pkey" primary key ("ID");
```

根据已有表结构创建表
```sql
create table if not exists 新表 (like 旧表 including indexes including comments including defaults);
```

删除表
```sql
drop table if exists "t_template" cascade;
```

查询注释
```sql
SELECT
a.attname as "字段名",
col_description(a.attrelid,a.attnum) as "注释",
concat_ws('',t.typname,SUBSTRING(format_type(a.atttypid,a.atttypmod) from '(.*)')) as "字段类型"
FROM
pg_class as c,
pg_attribute as a,
pg_type as t
WHERE
c.relname = 't_batch_task'
and a.atttypid = t.oid
and a.attrelid = c.oid
and a.attnum>0;
```

## 索引管理
查看索引
```sql
\d t_user
```

创建索引
```sql
drop index if exists t_user_username;
create index t_user_username on t_user (username);
```

创建唯一索引
```

```

## 执行 sql 脚本

方式一：先登陆再执行
```sql
\i testdb.sql
```

方式二：通过 psql 执行
```sql
psql -d testdb -U postgres -f /pathA/xxx.sql
```

## 导出数据
```sql
pg_dump -h localhost -p 5432 -U postgres --column-inserts -t table_name -f save_sql.sql database_name


--column-inserts #以带有列名的 `INSERT` 命令形式转储数据。
-t #只转储指定名称的表。
-f #指定输出文件或目录名。
```


## 连接访问控制
PostgreSQL 数据库安装后跟连接相关的配置 postgresql.conf,pg_hba.conf,pg_ident.conf

### 配置文件

postgresql.conf 文件
```
listen_addresses = '*'
port = 5866
```

pg_hba.conf 文件
数据库集群安装部署完成后，默认只允许本地连接，连接认证方式均为 trust ，生产环境建议更改为 md5 连接方式
```
# TYPE       DATABASE      USER        ADDRESS      METHOD
 local            all       all                        md5
  host            all       all   127.0.0.1/32         md5
  host            all       all      0.0.0.0/0         md5
  host            all       all        ::1/128         md5
 local    replication       all                        md5
  host    replication       all   127.0.0.1/32         md5
  host    replication       all        ::1/128         md5
```
修改完 pg_hba.conf 文件之后，需要重新加载配置，不需要重启数据库
```
pg_ctl reload
或
select pg_reload_conf();
```


TYPE 数据库连接方式：
* local: 匹配使用 Unix 域套接字的连接，如果没有此类型的记录，则不允许使用 Unix 域套接字连接
* host: 匹配使用 TCP/IP 进行的连接，主机记录匹配 SSL或非 SSL 连接，需要配置 listen_address
* hostssl: 匹配使用 TCP/IP 进行的连接，仅限于使用 SSL 加密进行连接，需要配置 ssl 参数
* hostnossl: 匹配通过 TCP/IP 进行的连接，不使用 SSL 的连接

DATABASE：指定哪些数据库可以被连接
* 匹配的数据库名称，all 指定它匹配所有数据库
* 复制（prelication） 不指定数据库
* 多个数据库可以用逗号分隔

USER 指定哪些用户可以连接
* 匹配的数据库用户名， all 指定它匹配所有用户
* 可以通过逗号分隔来提供多个用户名


ADDRESS 指定哪些 IP 地址可以连接
* 匹配的客户端计算机地址，all 匹配任何 IP 地址
* 0.0.0.0/0 表示所有 IPV4 地址
* :: 0/0 表示所有 IPV6 地址
* 192.168.100.101/32 允许这个 ip 登陆
* 192.168.100.0/24 允许 192.168.100.0~192.168.100.255 网段登陆数据库

METHOD 客户端认证方式
* trust: 只要知道数据库用户名就不需要密码或 ident 就能登陆，建议不要在生产环境中使用
* am-sha-256 ：密码认证，这是当前提供的方法中最安全的一种，但是旧的客户端库不支持这种方法
* md5：是常用的密码认证方式
* password: 以明文密码传送给数据库
* ident： Linux 下 PostgreSQL 默认的 local 认证方式，凡是能正确登陆操作系统用户（不是数据库用户）就能使用本用户映射的数据库用户不需要密码登陆数据库。
* reject: 拒绝认证

pg_ident.conf 文件
数据库映射文件，ident 认证方式的扩展，标注操作系统用户与数据库用户的映射关系，配合 pg_hba.conf 使用。允许数据库服务器上指定的操作系统用户，使用指定的数据库用户，免密连入数据库

```
pg_ident.conf 文件
# MAPNAME    SYSTEM-USERNAME    PG-USERNAME
      ss                aaa            test
      ss                syd             syd
      
pg_hba.conf 文件
# TYPE       DATABASE      USER        ADDRESS      METHOD
 local            all       all                      ident map=ss
  host            all       all   127.0.0.1/32         md5
  host            all       all      0.0.0.0/0         md5
  host            all       all        ::1/128         md5
 local    replication       all                        md5
  host    replication       all   127.0.0.1/32         md5
  host    replication       all        ::1/128         md5
```

* MAPNAME：映射名，自定义配置在 pg_hba.conf 文件中。
* SYSTEM-USERNAME：系统用户名。
* PG-USERNAME ：数据库用户名。