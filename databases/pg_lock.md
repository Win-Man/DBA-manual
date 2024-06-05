# PostgreSQL 锁机制

PostgreSQL 提供了多种锁模式用于控制对表中数据的并发访问，其中最主要的是表级锁和行级锁，除此之外还有页级锁、咨询锁等等

## 表级锁
表级锁通常会在执行各种命令时自动获取，或者通过在事务中使用 LOCK 语句显式获取。
每种锁都有自己的冲突集合。
表级锁：两个事务在同一时刻不能再同一个表上持有互相冲突的锁，但是可以同时持有不冲突的锁。
表级锁共有八种模式，其存在于 PG 的共享内存中，可以通过 pg_locks 系统视图查询。


| 锁模式 | 解释 |
| --- | ---- |
| ACCESS SHARE | 只与 ACCESS EXCLUSIVE 模式冲突。SELECT 命令将在所引用的表上加此类型的锁。通常，任何只读取表而不修改表的查询都会请求这种锁模式。|
| ROW SHARE | 与 EXCLUSIVE 和 ACCESS EXCLUSIVE 锁模式冲突。SELECT FOR UPDATE 和 SELECT FOR SHARE 命令会在目标表上加此类型的锁 |
| ROW EXCLUSIVE | 与 SHARE、SHARE ROW EXCLUSIVE、EXCLUSIVE、ACCESS EXCLUSIVE 锁模式冲突。UPDATE、DELETE、INSERT 命令会自动在所修改的表上请求加上这个锁。通常修改表中数据的命令都是加这种锁 |
| SHARE UPDATE EXCLUSIVE | 与 SHARE UPDATE EXCLUSIVE、SHARE、SHARE ROW EXCLUSIVE、EXCLUSIVE、ACCESS EXCLUSIVE 锁模式冲突。在模式改变和运行 VACUUM 并发的情况下，这个模式保护一个表。VACUUM(不带 FULL 选项)、ANALYZE、CREATE INDEX CONCURR_ENTLY 命令请求这样的锁。|
| SHARE | 与 ROW EXLUSIVE、SHARE UPDATE EXCLUSIVE、SHARE ROW EXCLUSIVE、EXCLUSIVE、ACCESS EXCLUSIVE 锁模式冲突。这个模式可避免表的并发数据修改。CREATE INDEX（不带 CONCURRENTLY）语句要求这样的锁模式 |
| SHARE ROW EXCLUSIVE | 与 ROW EXCLUSIVE、SHARE UPDATE EXCLUSIVE、SHARE、SHARE ROW EXCLUSIVE、EXCLUSIVE、ACCESS EXCLUSIVE 锁模式冲突。任何 PostgreSQL 命令都不会自动请求这个锁模式 |
| EXCLUSIVE | 与 ROW SHARE、ROW EXCLUSIVE、SHARE UPDATE EXCLUSIVE、SHARE、SHARE ROW EXCLUSIVE、EXCLUSIVE、ACCESS EXCLUSIVE 锁模式冲突。这个模式只允许并发 ACCESS SHARE 锁。也就是说，只有对表的读动作可以和持有这个锁的事务并发执行。任何 PostgreSQL 命令都不会再用户表上自动请求这个锁模式。不过，在执行某些操作时，会在某些系统表上请求这个锁 |
| ACCESS EXCLUSIVE | 与所有模式冲突，这个模式保证只能有一个人访问此表。ALTER TABLE、DROP TABLE、TRUNCATE、REINDEX、CLUSTER、VACUUM FULL 命令要求这样的锁。在 LOCK TABLE 命令中没有明确声明需要的锁模式时，它是默认锁模式。|

![](https://raw.githubusercontent.com/Win-Man/pic-storage/master/img/202405281050521.png)


## 行级锁

行级锁：同一个事务可能会在相同的行上保持冲突的锁，甚至是在不同的子事务中。但是除此之外，两个事务永远不可能在相同的行上持有冲突的锁。
行级锁不影响数据查询，它们只阻塞对同一行的写入者和加锁者。行级锁在事务结束时或保存点回滚的时候释放，就像表级锁一样。

常用的行级锁模式

| 锁模式 | 解释 |
| ----- | ----- |
| FOR UPDATE 更新 | FOR UPDATE 会导致由 SELECT 语句检索到的行被锁定，就好像它们要被更新。 |
| FOR NO KEY UPDATE 无键更新 | 行为与 FOR UPDATE 类似，不过获得的锁较弱，这种锁将不会阻塞尝试在相同行上获得锁的  SELECT FOR KEY SHARE 命令。|
| FOR SHARE 共享 |行为与 FOR NO KEY UPDATE 类似 ，不过它在每个检索到的行上获得一个共享锁锁而不是排他锁 |
| FOR KEY SHARE 键共享 | 行为与 FOR SHARE 类似，不过锁较弱，SELECT FOR  UPDATE 会被阻塞，但是 SELECT FOR NO KEY UPDATE 不会被阻塞，一个键共享锁会阻塞其他事务执行修改键值的 DELETE 或 UPDATE ，但不会阻塞其他 UPDATE ，也不会阻止 SELECT FOR NO KEY UPDATE、SELECT FOR SHARE 或者 SELECT FOR KEY SHARE。|

![](https://raw.githubusercontent.com/Win-Man/pic-storage/master/img/202406051415523.png)

## 执行语句 timeout 参数

* lock_timeout: 获取一个表、索引、行上的锁超过这个时间，直接报错，不等待，0 为禁用
* statement_timeout: 当 SQL 语句的执行时间超过这个设置时间，终止执行 SQL ， 0 为禁用
* idle_in_transaction_session_timeout: 在一个空闲的事务中，空闲时间超过这个值，将视为超时，0 为禁用
* deadlock_timeout: 死锁时间超时这个值将直接报错，不会等待，默认设置为 1s。

## 页级锁
## 劝告锁
## 监控锁

## 锁相关 SQL 
### 查看锁等待情况 SQL 

```sql
with
 t_wait as
 (
 select a.mode,a.locktype,a.database,a.relation,a.page,a.tuple,a.classid,a.granted,
  a.objid,a.objsubid,a.pid,a.virtualtransaction,a.virtualxid,a.transactionid,a.fastpath,
  b.state,b.query,b.xact_start,b.query_start,b.usename,b.datname,b.client_addr,b.client_port,b.application_name
 from pg_locks a,pg_stat_activity b where a.pid=b.pid and not a.granted
 ),
 t_run as
 (
 select a.mode,a.locktype,a.database,a.relation,a.page,a.tuple,a.classid,a.granted,
  a.objid,a.objsubid,a.pid,a.virtualtransaction,a.virtualxid,a.transactionid,a.fastpath,
  b.state,b.query,b.xact_start,b.query_start,b.usename,b.datname,b.client_addr,b.client_port,b.application_name
 from pg_locks a,pg_stat_activity b where a.pid=b.pid and a.granted
 ),
 t_overlap as
 (
 select r.* from t_wait w join t_run r on
  (
  r.locktype is not distinct from w.locktype and
  r.database is not distinct from w.database and
  r.relation is not distinct from w.relation and
  r.page is not distinct from w.page and
  r.tuple is not distinct from w.tuple and
  r.virtualxid is not distinct from w.virtualxid and
  r.transactionid is not distinct from w.transactionid and
  r.classid is not distinct from w.classid and
  r.objid is not distinct from w.objid and
  r.objsubid is not distinct from w.objsubid and
  r.pid <> w.pid
  )
 ),
 t_unionall as
 (
 select r.* from t_overlap r
 union all
 select w.* from t_wait w
 )
 select locktype,datname,relation::regclass,page,tuple,virtualxid,transactionid::text,classid::regclass,objid,objsubid,
  string_agg(
   'Pid: '||case when pid is null then 'NULL' else pid::text end||chr(10)||
   'Lock_Granted: '||case when granted is null then 'NULL' else granted::text end||' , Mode: '||case when mode is null then 'NULL' else mode::text end||' , FastPath: '||case when fastpath is null then 'NULL' else fastpath::text end||' , VirtualTransaction: '||case when virtualtransaction is null then 'NULL' else virtualtransaction::text end||' , Session_State: '||case when state is null then 'NULL' else state::text end||chr(10)||
   'Username: '||case when usename is null then 'NULL' else usename::text end||' , Database: '||case when datname is null then 'NULL' else datname::text end||' , Client_Addr: '||case when client_addr is null then 'NULL' else client_addr::text end||' , Client_Port: '||case when client_port is null then 'NULL' else client_port::text end||' , Application_Name: '||case when application_name is null then 'NULL' else application_name::text end||chr(10)||
   'Xact_Start: '||case when xact_start is null then 'NULL' else xact_start::text end||' , Query_Start: '||case when query_start is null then 'NULL' else query_start::text end||' , Xact_Elapse: '||case when (now()-xact_start) is null then 'NULL' else (now()-xact_start)::text end||' , Query_Elapse: '||case when (now()-query_start) is null then 'NULL' else (now()-query_start)::text end||chr(10)||
   'SQL (Current SQL in Transaction): '||chr(10)||
   case when query is null then 'NULL' else query::text end,
   chr(10)||'--------'||chr(10)
  order by
   ( case mode
   when 'INVALID' then 0
   when 'AccessShareLock' then 1
   when 'RowShareLock' then 2
   when 'RowExclusiveLock' then 3
   when 'ShareUpdateExclusiveLock' then 4
   when 'ShareLock' then 5
   when 'ShareRowExclusiveLock' then 6
   when 'ExclusiveLock' then 7
   when 'AccessExclusiveLock' then 8
   else 0
   end  ) desc,
   (case when granted then 0 else 1 end)
  ) as lock_conflict
 from t_unionall
 group by
  locktype,datname,relation,page,tuple,virtualxid,transactionid::text,classid,objid,objsubid ;
```

### 查看阻塞会话，并生成 kill sql 

```sql
SELECT pg_cancel_backend(pid); – session还在，事物回退;
SELECT pg_terminate_backend(pid); --session消失，事物回退

with recursive 
tmp_lock as (
 select distinct
  --w.mode w_mode,w.page w_page,
  --w.tuple w_tuple,w.xact_start w_xact_start,w.query_start w_query_start,
  --now()-w.query_start w_locktime,w.query w_query
  w.pid as id,--w_pid,
  r.pid as parentid--r_pid,
  --r.locktype,r.mode r_mode,r.usename r_user,r.datname r_db,
  --r.relation::regclass,
  --r.page r_page,r.tuple r_tuple,r.xact_start r_xact_start,
  --r.query_start r_query_start,
  --now()-r.query_start r_locktime,r.query r_query,
 from (
  select a.mode,a.locktype,a.database,
  a.relation,a.page,a.tuple,a.classid,
  a.objid,a.objsubid,a.pid,a.virtualtransaction,a.virtualxid,
  a.transactionid,
  b.query as query,
  b.xact_start,b.query_start,b.usename,b.datname
  from pg_locks a,
  pg_stat_activity b
  where a.pid=b.pid
  and not a.granted
 ) w,
 (
  select a.mode,a.locktype,a.database,
  a.relation,a.page,a.tuple,a.classid,
  a.objid,a.objsubid,a.pid,a.virtualtransaction,a.virtualxid,
  a.transactionid,
  b.query as query,
  b.xact_start,b.query_start,b.usename,b.datname
  from pg_locks a,
  pg_stat_activity b -- select pg_typeof(pid) from pg_stat_activity
  where a.pid=b.pid
  and a.granted
 ) r
 where 1=1
  and r.locktype is not distinct from w.locktype
  and r.database is not distinct from w.database
  and r.relation is not distinct from w.relation
  and r.page is not distinct from w.page
  and r.tuple is not distinct from w.tuple
  and r.classid is not distinct from w.classid
  and r.objid is not distinct from w.objid
  and r.objsubid is not distinct from w.objsubid
  and r.transactionid is not distinct from w.transactionid
  and r.pid <> w.pid
 ),
tmp0 as (
 select *
 from tmp_lock tl
 union all
 select t1.parentid,0::int4
 from tmp_lock t1
 where 1=1
 and t1.parentid not in (select id from tmp_lock)
 ),
tmp3 (pathid,depth,id,parentid) as (
 SELECT array[id]::text[] as pathid,1 as depth,id,parentid
 FROM tmp0
 where 1=1 and parentid=0
 union
 SELECT t0.pathid||array[t1.id]::text[] as pathid,t0.depth+1 as depth,t1.id,t1.parentid
 FROM tmp0 t1, tmp3 t0
 where 1=1 and t1.parentid=t0.id
)
select distinct
 '/'||array_to_string(a0.pathid,'/') as pathid,
 a0.depth,
 a0.id,a0.parentid,lpad(a0.id::text, 2*a0.depth-1+length(a0.id::text),' ') as tree_id,
 --'select pg_cancel_backend('||a0.id|| ');' as cancel_pid,
 --'select pg_terminate_backend('||a0.id|| ');' as term_pid,
 case when a0.depth =1 then 'select pg_terminate_backend('|| a0.id || ');' else null end  as term_pid,
 case when a0.depth =1 then 'select cancel_backend('|| a0.id || ');' else null end  as cancel_pid
 ,a2.datname,a2.usename,a2.application_name,a2.client_addr,a2.wait_event_type,a2.wait_event,a2.state
 --,a2.backend_start,a2.xact_start,a2.query_start
from tmp3 a0
left outer join (select distinct '/'||id||'/' as prefix_id,id
 from tmp0
 where 1=1 ) a1
on position( a1.prefix_id in '/'||array_to_string(a0.pathid,'/')||'/' ) >0
left outer join pg_stat_activity a2 -- select * from pg_stat_activity
on a0.id = a2.pid
order by '/'||array_to_string(a0.pathid,'/'),a0.depth;
```

### 查询当前执行时间超过 60s 的 SQL 

```sql
select
 pg_stat_activity.datname,
 pg_stat_activity.pid,
 pg_stat_activity.query,
 pg_stat_activity.client_addr,
 clock_timestamp() - pg_stat_activity.query_start
from
 pg_stat_activity pg_stat_activity
where
 (pg_stat_activity.state = any (array['active'::text,
 'idle in transaction'::text]))
 and (clock_timestamp() - pg_stat_activity.query_start) > '00:00:60'::interval
order by
 (clock_timestamp() - pg_stat_activity.query_start) desc;
```

### 查询所有已经获取锁的 SQL 

```sql
SELECT pid, state, usename, query, query_start 
from pg_stat_activity 
where pid in (
  select pid from pg_locks l  join pg_class t on l.relation = t.oid 
  and t.relkind = 'r' 
);
```