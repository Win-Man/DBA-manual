# Oracle 命令

## 常用操作
* 连接数据库

```
sqlplus 用户名/密码@连接 as sysdba

在服务器使用的是操作系统验证的方式登陆的，所以没有用户名、密码、连接实际是一样的

sqlplus / as sysdba
```

* 查看当前用户名

```
show user
select user from dual
```

* 查看所有用户名

```
select * from all_users
```

* sqlplus 输出格式化控制命令

```
show linesize : 查看当前设置的sqlplus输出的最大行宽
set linesize : 设置sqlplus输出的最大行宽
column : 修改显示字段的长度或名称
  column col_name format a15       --将列col_name（字符型）显示最大宽度调整为15个字符
  column col_num format 999999     --将列col_num（num型）显示最大宽度调整为6个字符
  column col_num heading col_num2  --将col_num的列名显示为col_num2
```

* 创建用户、授权、连接

```
SQL> create user c##gangshen identified by 123456;

用户已创建。

SQL> grant connect,resource,dba to c##gangshen;

授权成功。

SQL> connect c##gangshen/123456@orcl
```

* 查看所有表

```
select table_name from user_tables order by table_name;
```

* 启动 Oracle 

```
lsnrctl start
sqlplus / as sysdba
startup;

alter system register;
```

* 查看当前字符集

```
select userenv('LANGUAGE') a from dual;

SELECT * FROM NLS_DATABASE_PARAMETERS;--数据库服务器字符集，来源于PROPS$
SELECT * FROM NLS_INSTANCE_PARAMETERS;--客户端字符集，来源于V$PARAMETER
SELECT * FROM NLS_SESSION_PARAMETERS;--会话字符集，来源于V$NLS_PARAMETERS，表示会话自己的设置，可能是会话的环境变量或者是由ALTER SESSION完成，如果会话没有特殊的设置，将与NLS_INSTANCE_PARAMETERS一致
SELECT * FROM V$NLS_PARAMETERS;
SELECT * FROM SYS.PROPS$;
```

* 创建 schema 以及用户

```
create tablespace ecasdb datafile '/home/oracle/oradata/orcl/ecasdb.dbf' size 200m autoextend on next 100m maxsize unlimited;

create user ecasdb identified by ecasdb default tablespace ecasdb;

grant connect,resource,dba to ecasdb;

grant create session to ecasdb;
```


* 查看表空间大小
```
SELECT t.tablespace_name, round(SUM(bytes / (1024 * 1024)), 0) ts_size
FROM dba_tablespaces t, dba_data_files d
WHERE t.tablespace_name = d.tablespace_name
GROUP BY t.tablespace_name;
```