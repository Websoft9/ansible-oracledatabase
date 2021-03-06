# Oracle Database速学

## 概念

### 数据库

Oracle Database同其他数据一样，本质上就数据的物理存储。包括数据文件ORA、DBF、控制文件、联机日志、参数文件。

#### 物理结构

数据库的物理存储结构是由一些多种物理文件组成，主要有数据文件、控制文件、重做日志文件、归档日志文件、参数文件、口令文件、警告文件等。

 - 控制文件：存储实例、数据文件及日志文件等信息的二进制文件。
 - 数据文件：存储数据，以.dbf做后缀。一句话：一个表空间对多个数据文件，一个数据文件只对一个表空间。
 - 日志文件：即Redo Log Files和Archivelog Files。记录数据库修改信息。
 - 参数文件：记录基本参数。spfile和pfile。
 - 警告文件：show parameter background_dump_dest---使用共享服务器连接。
 - 跟踪文件：show parameter user_dump_dest---使用专用服务器连接。

#### 逻辑结构

数据库逻辑结构由逻辑存储结构(表空间,段,范围,块)和逻辑数据结构(表、视图、序列、存储过程、同义词、索引、簇和数据库链等)组成,而其中的模式对象(逻辑数据结构)和关系形成了数据库的关系设计。逻辑存储结构包括表空间、段和范围，用于描述怎样使用数据库的物理空间。

 - 表空间：表空间是一个用来管理数据存储逻辑概念，表空间只是和数据文件（ORA或者DBF文件）发生关系，数据文件是物理的，一个表空间可以包含多个数据文件，而一个数据文件只能隶属一个表空间。
 - 段（Segment）：是表空间中一个指定类型的逻辑存储结构，它由一个或多个范围组成，段将占用并增长存储空间。其中包括：数据段：用来存放表数据；索引段：用来存放表索引；临时段：用来存放中间结果；
回滚段：用于出现异常时，恢复事务。
 - 范围（Extent）：是数据库存储空间分配的逻辑单位，一个范围由许多连续的数据块组成，范围是由段依次分配的，分配的第一个范围称为初始范围，以后分配的范围称为增量范围。
 - 数据块（Block）：是数据库进行IO操作的最小单位，它与操作系统的块不是一个概念。oracle数据库不是以操作系统的块为单位来请求数据，而是以多个Oracle数据库块为单位。

#### 数据库名

数据库名就是一个数据库的标识，就像人的身份证号一样。他用参数DB_NAME表示，如果一台机器上装了多个数据库，那么每一个数据库都有一个数据库名。在创建数据库时就应考虑好数据库名，
并且在创建完数据库之后，数据库名不宜修改，即使要修改也会很麻烦。因为，数据库名还被写入控制文件中，控制文件是以 二进制型式存储的，用户无法修改控制文件的内容。假设用户修改了
参数文件中的数据库名，即修改DB_NAME的值。


### 数据库实例

数据库实例（instance）是一组用于管理数据库文件的内存结构。由一系列的后台进程（Backguound Processes)和内存结构（Memory Structures)组成。一个数据库可以有多个实例。

#### 实例结构

当一个实例启动时，Oracle 数据库分配一个称为系统全局区（SGA）的内存区域，并启动一个或多个后台进程。
SGA 的作用包括：
 - 维护多个进程和线程并发访问的内部数据结构
 - 缓存从磁盘读取的数据块
 - 在写入在线重做日志文件之前缓冲重做数据
 - 存储 SQL 执行计划
同一个服务器上的 Oracle 进程之间共享 SGA。Oracle 进程与 SGA 的交互方式取决于操作系统。

一个数据库实例包括多个后台进程（background process）。服务器进程（server process），以及分配给它们的内存，也位于实例之中。实例在服务器进程结束后仍然继续存在。
下图显示了 Oracle Database 实例中的主要组件。
  ![](https://libs.websoft9.com/Websoft9/DocsPicture/zh/oracle_database/oracle-instancecomponent-websoft9.png)

#### 实例名和实例配置

数据库实例名是用于和操作系统进行联系的标识，就是说数据库和操作系统之间的交互用的是数据库实例名。实例名也被写入参数文件中，该参数为instance_name。
数据库名和实例名可以相同也可以不同。在单实例情况下，数据库名和实例名是一对一的关系，但如果在应用集群（Oracle RAC）配置中，数据库名和实例名是一对多的关系。无论是单实例还是 Oracle RAC 配置，一个实例每次只能与一个数据库关联。管理员可以启动一个实例，然后加载（关联）一个数据库，但是不能同时加载两个数据库。
下图显示了两种可能的数据库实例配置。
  ![](https://libs.websoft9.com/Websoft9/DocsPicture/zh/oracle_database/oracle-instancedb-websoft9.png)

如何查询数据库实例名？
```
select instance_name from v$instance;
show parameter instance_name;
```

### 数据库服务

#### 服务名

参数是从Oracle8i新引进的。在8i以前，我们用SID来表示标识数据库的一个实例，但是在Oracle的并行环境中，一个数据库对应多个实例，这样就需要多个网络服务名，设置繁琐。为了方便并行环境中的设置，引进了Service_name参数，该参数对应一个数据库，而不是一个实例，而且该参数有许多其它的好处。该参数的缺省值为Db_name. Db_domain，即等于Global_name。一个数据库可以对应多Service_name，以便实现更灵活的配置。该参数与SID没有直接关系，即不必Service name 必须与SID一样。如果数据库有域名，则数据库服务名就是全局数据库名，否则，数据库服务名与数据库名相同。

### Oracle Database 启动

#### 启动过程

Oracle  Database 的启动需要经历四个状态，SHUTDOWN 、NOMOUNT 、MOUNT 、OPEN。如果使用 Oracle Net 启动一个数据库实例，需要满足以下条件：
 - 数据库通过静态方式注册到 Oracle Net 监听器中
 - 使用 SYSDBA 权限进行连接

监听器启动一个专用的服务器进程，用于启动数据库实例，过程如下：
 - 启动实例，未加载数据库
 - 加载数据库
 - 打开数据库

  ![](https://libs.websoft9.com/Websoft9/DocsPicture/zh/oracle_database/oracle-startup-websoft9.png)

#### 管理员登录
数据库的启动和关闭是非常强大的管理功能，只能由具有管理员权限的用户执行。普通用户无法控制数据库的当前状态。以下特殊的系统权限能够在数据库未打开时访问实例：
 - SYSDBA
 - SYSOPER
 - SYSBACKUP
 - SYSDG
 - SYSKM
以上权限的管理不在数据库的自身范围之内。使用 SYSDBA 系统权限连接时，用户位于 SYS 模式中。使用 SYSOPER 连接时，用户位于公共模式中。SYSOPER 权限是 SYSDBA 权限的一个子集。

### Oracle Database 用户

#### 默认用户

Oracle Database 默认用户分为：SYS、SYSTEM、SCOTT。
 - SYS:数据库中所有数据字典表和视图都存储在 SYS 模式中。SYS用户主要用来维护系统信息和管理实例。
 - SYSTEM:SYSTEM 是默认的系统管理员，该用户拥有Oracle管理工具使用的内部表和视图。通常通过SYSTEM用户管理数据库用户、权限和存储等。普通用户授予DBA角色后基本和SYSTEM用户一样。
 - SCOTT:SCOTT用户是Oracle 数据库的一个示范帐户，在数据库安装时创建。

#### Oracle Database 用户权限

Oracle 用户权限：SYSDBA、SYSOPER、NORMAL。
 - SYSDBA权限，即数据库管理员权限，最高的系统权限。任何用户通过SYSDBA登录后用户变成SYS。权限包括：管理功能, 创建数据库(CREATE DATABASE)以及 SYSOPER的所有权限
 - SYSOPER权限，即数据库操作员权限，SYSOPER主要用来启动、关闭数据库，普通用户授权SYSOPER登陆后用户是 public。权限包括：打开数据库，关闭数据库服务器，备份数据库 ，恢复数据库，日志归档，会话限制
 - NORMAL权限的用户代表普通用户

### Oracle Database 远程访问

Oracle Database客户端连接服务端的过程，是通过监听器(LISTENER)和Oracle实例发生连接。因此，在监听的配置文件中设置是否能通过远程访问。
listener.ora配置如下，当HOST为localhost时无法通过远程访问；如果为机器名或者IP地址时，可以通过公网IP访问。
```
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )
```

### 监听器(LISTENER)

监听器是Oracle Database基于服务器端的一种网络服务，主要用于监听客户端向数据库服务器端提出的连接请求。既然是基于服务器端的服务，那么它也只存在于数据库服务器端，进行监听器的设置也是在数据库服务器端完成的。

#### 为什么需要监听？

用通俗的话来讲，监听就是数据库的管家。客户端访问实例的时候，管家去查看访问者IP是否上了黑名单，能否让他进入，如果决定让访问者进入；访问者还需要输入密码。密码输入正确后，
他就可以进入屋子，至于那个房间可以进入那个不可以，那是权限的问题了。

#### 如何配置监听

1. Oracle Net配置

2. 直接配置$ORACLE_HOME/network/admin/listerer.ora，示例如下：

```
# listener.ora Network Configuration File: /opt/oracle/product/19c/dbhome_1/network/admin/listener.ora
# Generated by Oracle configuration tools.

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = iZj6cdy96p2tjgceyevws4Z)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )

SID_LIST_LISTENER=
  (SID_LIST=
      (SID_DESC=
         (GLOBAL_DBNAME=ORCLCDB)
         (SID_NAME=ORCLCDB)
         (ORACLE_HOME=/opt/oracle/product/19c/dbhome_1)
       )
   )

ADR_BASE_LISTENER = /opt/oracle/
```


### Web可视化工具（EM）

Oracle Database 安装时会自动安装EM，浏览器访问网址：*https://域名:5500/em* 或 *https://Internet IP:5500/em*

  ![](https://libs.websoft9.com/Websoft9/DocsPicture/zh/oracle_database/oracle-emlogin-websoft9.png)

请使用系统用户登陆数据库([不知道账号密码？](/zh/stack-accounts.html))，登陆成功进入主页面
  ![](https://libs.websoft9.com/Websoft9/DocsPicture/zh/oracle_database/oracle-emmain-websoft9.png)

### 客户端可视化工具
