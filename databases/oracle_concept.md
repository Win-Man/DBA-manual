# Oracle 基本概念

## Oracle 架构

- Oracle 数据库服务器由一个数据库和至少一个数据库实例组成。一个数据库可以有多个数据库实例
- 数据库和实例是紧密相连的，一般说的 Oracle 数据库是指实例和数据库。
    - 数据库是一组存储数据库的文件，由后台进程组成
    - 数据库实例是管理数据库文件的内存结构
    ![](https://raw.githubusercontent.com/Win-Man/pic-storage/master/img/202406040910264.png)
    - 数据文件（dbf）
    数据文件是数据库的物理存储范围。表空间由一个或多个数据文件组成，一个数据文件只能属于一个表空间。
    - 表空间
    表空间是 Oracle 对物理数据库上相关数据文件（ORA 或者 DBF 文件）的逻辑映射。一个数据库在逻辑上被划分成一个到若干个表空间
    - 用户
    用户是在数据库实例下面建立的。不同实例中可以建相同名字的用户。
    ![](https://raw.githubusercontent.com/Win-Man/pic-storage/master/img/202406040911392.png)
    - schema
    schema 就是数据库对象的集合，schema 名字与 user 的名字相同。等于 MySQL 中 database 的概念

## 用户权限
默认创建 sys 用户

所有 Oracle 的数据字典信息的基表和视图都放在 sys 用户中，sys 用户拥有 dba,sysdba,sysoper 等角色和权限，是 oracle 权限最高的用户

system 用户用于存放次一级的内部数据，如 oracle 的一些特性或工具的管理信息。system 用户拥有普通 dba 角色权限

sys 用户和 system 用户区别：

- system用户只能用normal身份登陆em，除非你对它授予了sysdba的系统权限或者syspoer系统权限。
- sys用户具有“SYSDBA”或者“SYSOPER”系统权限，登陆em也只能用这两个身份，不能用normal。

normal、sysdba、sysoper 区别：

- normal 是普通用户
- sysdba 拥有最高权限，登陆后是 sys
- sysoper 主要用来启动、关闭数据库,sysoper 登陆后用户是 public