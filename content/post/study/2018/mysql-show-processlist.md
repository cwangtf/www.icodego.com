---
title: "MySQL SHOW PROCESSLIST 命令详解"
date: 2018-08-31T23:19:45+08:00
categories: ["MySQL"]
tags: ["MySQL", "数据库管理语句"]
---

### SHOW PROCESSLIST 命令

语法：
```mysql
SHOW [FULL] PROCESSLIST
```

在程序中，如果SQL使用不当，导致查询非常消耗性能，在业务高峰期的时候可能会导致连接数不够用的情况，提示"too many connections"，
使用本语句能了解正在发生的情况。show processlist 是显示用户正在运行的线程，需要注意的是，除了 root 用户能看到所有正在运行的线程外，
其他用户都只能看到自己正在运行的线程，看不到其它用户正在运行的线程。除非单独个这个用户赋予了PROCESS 权限。

SHOW PROCESSLIST输出示例：
```mysql
mysql> show full processlist\G
*************************** 1. row ***************************
     Id: 123
   User: root
   Host: localhost
     db: performance_schema
Command: Query
   Time: 0
  State: starting
   Info: show full processlist
1 row in set (0.00 sec)
```
所产生的列SHOW PROCESSLIST具有以下含义：

Id

连接标识符，这是同一类型的在所显示的值ID的列INFORMATION_SCHEMA.PROCESSLIST表，该PROCESSLIST_ID性能视图的列threads 表，并且通过返回的 CONNECTION_ID()功能。

User

发出声明的MySQL用户，如果是system user，它是指由服务器产生的非客户线程，以在内部处理任务。这可能是复制从站或延迟行处理程序使用的I / O或SQL线程。unauthenticated user指的是已经与客户端连接关联但是还没有完成客户机用户的认证的线程。 event_scheduler指的是监视预定事件的线程。如果是system user那么在Host列中不会指定主机 。

Host

发出该语句的客户端的主机名（system user没有主机除外），以便更容易地确定哪个客户端正在做什么，显示方式：host_name:client_port。

db

当前执行语句对应的默认数据库，如果选择了；否则为NULL。

Command

显示这个线程此刻正在执行的命令，一般对应DDL或DML语句。

Time

表示线程处于当前状态的时间长短，线程当前时间的概念在某些情况下可能会发生改变：线程可以改变时间。对于正在从主机处理事件的从站上运行的线程，线程时间设置为事件中发现的时间，因此反映了主站而不是从站的当前时间。SET TIMESTAMP = value。

State

对应Command指令，大多数状态对应于非常快速的操作。如果线程在给定状态下保持多秒，则可能存在需要调查的问题。

Info

包含由线程执行的语句的文本或者NULL，如果它不是执行的话。默认情况下，此值仅包含语句的前100个字符。要查看完整的语句，请使用SHOW FULL PROCESSLIST。

### 线程命令（Command）值
线程可以具有以下任何 Command值：

Binlog Dump：这是主服务器上的线程，用于将二进制日志内容发送到从服务器。

Table Dump：线程将表内容发送到从服务器。

Change user：线程正在执行改变用户操作。

Close stmt：线程正在关闭准备好的语句。

Connect：复制中，从服务器连接到其主服务器。

Connect Out：复制中，从服务器正在连接到其主服务器。

Create DB：线程正在执行create-database操作。

Daemon：此线程在服务器内部，而不是服务客户端连接的线程。

Debug：线程正在生成调试信息。

Delayed insert：线程是一个延迟插入处理程序。

Drop DB：线程正在执行drop-database操作。

Execute：线程正在执行一个准备好的语句（prepare statement类型就是预编译的语句，JDBC支持次类型执行SQL）。

Fetch：线程正在执行一个准备语句的结果。

Field List：线程正在检索表列的信息。

Init DB：线程正在选择默认数据库。

Kill：线程正在杀死另一个线程。

Long Data：该线程在执行一个准备语句的结果中检索长数据。

Ping：线程正在处理服务器ping请求。

Prepare：线程正在为语句生成执行计划。

Processlist：线程正在生成有关服务器线程的信息。

Query：该线程正在执行一个语句。

Quit：线程正在终止。

Refresh：线程是刷新表，日志或缓存，或重置状态变量或复制服务器信息。

Register Slave：线程正在注册从服务器。

Reset stmt：线程正在重置一个准备好的语句。

Set option：线程正在设置或重置客户端语句执行选项。

Shutdown：线程正在关闭服务器。

Sleep：线程正在等待客户端向其发送新的语句。

Statistics：线程正在生成服务器状态信息。

Time：没用过。

### 线程状态（State）值
* 一般线程状态（State）值

以下列表描述State 了与常规查询处理关联的线程值，而不是更复杂的活动，例如复制。其中许多仅用于在服务器中查找错误。

After create：当线程创建表（包括内部临时表）时，会在创建表的函数的末尾创建。即使由于某些错误而无法创建表，也会使用此状态。

Analyzing：线程正在计算MyISAM表密钥分布（例如:for ANALYZE TABLE）。

checking permissions：线程正在检查服务器是否具有执行语句所需的权限。

Checking table：线程正在执行表检查操作。

cleaning up：线程已经处理了一个命令，正在准备释放内存并重置某些状态变量。

closing tables：线程将更改的表数据刷新到磁盘并关闭已用表。这应该是一个快速的操作。如果没有，请验证您是否没有完整的磁盘，并且磁盘没有被非常大的使用。

copy to tmp table：线程正在处理ALTER TABLE语句。此状态发生在已创建新结构的表之后，但是将行复制到该表之前。对于此状态的线程，可以使用性能模式来获取有关复制操作的进度。

Copying to group table：如果语句具有不同ORDER BY和GROUP BY标准，各行按组排列和复制到一个临时表。

Creating index：线程正在处理ALTER TABLE … ENABLE KEYS一个MyISAM表。

Creating sort index：线程正在处理一个SELECT使用内部临时表解析的线程 。

creating table：线程正在创建一个表，这包括创建临时表。

committing alter table to storage engine：服务器已经完成就位ALTER TABLE并提交结果。

deleting from main table：服务器正在执行多表删除的第一部分，它仅从第一个表中删除，并从其他（引用）表中保存要用于删除的列和偏移量。

deleting from reference tables：服务器正在执行多表删除的第二部分，并从其他表中删除匹配的行。

discard_or_import_tablespace：线程正在处理ALTER TABLE … DISCARD TABLESPACE或ALTER TABLE … IMPORT TABLESPACE声明。

end：这发生在结束，但的清理之前ALTER TABLE， CREATE VIEW， DELETE， INSERT， SELECT，或UPDATE语句。

executing：该线程已经开始执行一个语句。

Execution of init_command：线程正在init_command系统变量的值中执行语句 。

freeing items：线程已经执行了一个命令，在这种状态下完成的项目的一些释放涉及查询缓存，这个状态通常在后面cleaning up。

FULLTEXT initialization：服务器正在准备执行自然语言全文搜索。

init：此操作在初始化ALTER TABLE, DELETE, INSERT, SELECT, or UPDATE之前发生，服务器在该状态中采取的操作包括刷新二进制日志、Innodb日志和一些查询缓存清理操作。对于最终状态, 可能会发生以下操作：更改表中的数据后删除查询缓存项、将事件写入二进制日志、释放内存缓冲区, 包括blob。

Killed：执行KILL语句，向线程发送了一个声明，下次检查kill标志时应该中断。在MySQL的每个主循环中检查该标志，但在某些情况下，线程可能需要很短时间才能死掉。如果线程被某个其他线程锁定，则一旦其他线程释放锁定，该kill就会生效。

Locking system tables：线程正在尝试锁定系统表（例如，时区或日志表）。

login：连接线程的初始状态，直到客户端成功认证为止。

manage keys：服务器启用或禁用表索引。

NULL：该状态用于SHOW PROCESSLIST状态。

Opening system tables：线程尝试打开系统表（例如，时区或日志表）。

Opening tables：线程正在尝试打开一个表，这应该是非常快的程序，除非有事情阻止打开。例如，一个ALTER TABLE或一个LOCK TABLE语句可以阻止打开一个表，直到语句完成。还可能需要关注table_open_cache参数的值是否足够大。对于系统表，使用Opening system tables状态。

optimizing：服务器正在执行查询的初始优化。

preparing：此状态发生在查询优化期间。

Purging old relay logs：线程正在删除不需要的中继日志文件。

query end：处理查询之后，freeing items状态之前会发生这种状态。

Removing duplicates：该查询的使用SELECT DISTINCT方式使得MySQL不能在早期阶段优化不同的操作。因此，MySQL需要一个额外的阶段来删除所有重复的行，然后将结果发送给客户端。

removing tmp table：处理语句后，该线程正在删除一个内部临时表SELECT 。如果没有创建临时表，则不使用该状态。

rename：线程正在重命名一个表。

rename result table：线程正在处理一个ALTER TABLE语句，已经创建了新表，并重新命名它来替换原始表。

Reopen tables：线程获得了表的锁，但在获得基础表结构更改的锁之后注意到。它释放了锁，关闭了table，并试图重新打开它。

Repair by sorting：修复代码正在使用排序来创建索引。

preparing for alter table：服务器正在准备就地执行ALTER TABLE。

Repair done：线程已经完成了一个MyISAM表的多线程修复 。

Repair with keycache：修复代码通过密钥缓存逐个使用创建密钥，这比慢得多Repair by sorting。

Rolling back：线程正在回滚事务。

Saving state：对于MyISAM表操作（如修复或分析），线程将新的表状态保存到.MYI文件头。状态包括行数， AUTO_INCREMENT计数器和键分布等信息。

Searching rows for update：线程正在进行第一阶段，以便在更新之前查找所有匹配的行。如果UPDATE要更改用于查找涉及的行的索引，则必须执行此操作 。

setup：线程正在开始一个ALTER TABLE操作。

Sorting for group：线程正在做一个满足一个GROUP BY。

Sorting for order：线程正在做一个满足一个ORDER BY。

Sorting index：线程是排序索引页，以便在MyISAM表优化操作期间更有效地访问。

Sorting result：对于一个SELECT语句，这类似于Creating sort index，但是对于非临时表。

statistics：服务器正在计算统计信息以开发查询执行计划。如果一个线程长时间处于这种状态，服务器可能是磁盘绑定的，执行其他工作。

update：线程正在准备开始更新表。

Updating：线程正在搜索要更新的行并正在更新它们。

updating main table：服务器正在执行多表更新的第一部分，它仅更新第一个表，并保存用于更新其他（引用）表的列和偏移量。

updating reference tables：服务器正在执行多表更新的第二部分，并从其他表更新匹配的行。

User lock：线程将要求或正在等待通过GET_LOCK()呼叫请求的咨询锁定 。因为 SHOW PROFILE，这个状态意味着线程正在请求锁定（不等待它）。

User sleep：线程调用了一个 SLEEP()调用。

* 故障诊断状态（State）值（个人提取）

logging slow query：线程正在向慢查询日志写入语句。

altering table：服务器正在执行就地ALTER TABLE。

Receiving from client：服务器正在从客户端读取数据包。

Copying to tmp table：服务器正在复制磁盘到内存的临时表，是直接在磁盘创建的临时表而并非从内存转到磁盘的临时表。

Copying to tmp table on disk：对于线程将临时表从内存中更改为基于磁盘的格式存储以节省内存后，又把临时表从磁盘复制到内存时的状态。

Creating tmp table：线程正在内存或磁盘上创建临时表。如果表在内存中创建，但后来转换为磁盘表，则该操作中的状态将为Copying to tmp table on disk。

Sending data：线程正在读取和处理SELECT语句的行，并将数据发送到客户端。由于在此状态期间发生的操作往往执行大量的磁盘访问（读取），所以在给定查询的整个生命周期内通常是最长的运行状态。

Sending to client：服务器正在向客户端写入数据包。

Waiting for commit lock：FLUSH TABLES WITH READ LOCK正在等待提交锁。

Waiting for global read lock：FLUSH TABLES WITH READ LOCK正在等待全局读锁定或read_only正在设置全局系统变量。

Waiting for tables：线程得到一个通知，表格的底层结构已经改变，需要重新打开表以获得新的结构。但是，要重新打开表格，必须等到所有其他线程都关闭该表。如果另一个线程已使用FLUSH TABLES或下面的语句之一：FLUSH TABLES tbl_name, ALTER TABLE, RENAME TABLE, REPAIR TABLE, ANALYZE TABLE, 或OPTIMIZE TABLE都会发生通知。

Waiting for table flush：线程正在执行FLUSH TABLES并正在等待所有线程关闭它们的表，或者线程得到一个通知，表中的底层结构已经改变，并且需要重新打开表以获得新的结构。但是，要重新打开表，必须等到所有其他线程都关闭该表。如果另一个线程已使用FLUSH TABLES或下面的语句之一：FLUSH TABLES tbl_name, ALTER TABLE, RENAME TABLE, REPAIR TABLE, ANALYZE TABLE, 或OPTIMIZE TABLE都会发出这个通知。

Waiting for lock_type lock：服务器正在等待THR_LOCK从元数据锁定子系统获取锁或锁，其中lock_type指示锁的类型。THR_LOCK状态表示：Waiting for table level lock；这些状态表示等待元数据锁定：Waiting for event metadata lock、Waiting for global read lock、Waiting for schema metadata lock、Waiting for stored function metadata lock、Waiting for stored procedure metadata lock、Waiting for table metadata lock、Waiting for trigger metadata lock。

Writing to net：服务器正在将数据包写入网络，如果一个线程长时间在执行并且一直处于Writing to net状态，那么一直在发送数据包到网络，可以试着调整max_allowed_packet大小。另外，这可能会导致其他线程大量阻塞。

Waiting on cond：线程等待条件成为true的一般状态，没有特定的状态信息可用。

System lock：线程已经调用mysql_lock_tables() ，且线程状态从未更新。这是一个非常普遍的状态，可能由于许多原因而发生。例如, 线程将请求或正在等待表的内部或外部系统锁。当InnoDB在执行锁表时等待表级锁时, 可能会发生这种情况。如果此状态是由于请求外部锁而导致的，并且不使用正在访问相同表的多个mysqld服务器MyISAM，则可以使用该–skip-external-locking选项禁用外部系统锁 。但是，默认情况下禁用外部锁定，因此这个选项很有可能不起作用。因为SHOW PROFILE，这个状态意味着线程正在请求锁定（不等待它）。对于系统表，使用Locking system tables状态。

* 查询缓存状态（State）值

checking privileges on cached query：服务器正在检查用户是否具有访问缓存查询结果的权限。

checking query cache for query：服务器正在检查当前查询是否存在于查询缓存中。

invalidating query cache entries：查询缓存条目被标记为无效，因为底层表已更改。

sending cached result to client：服务器正在从查询缓存中获取查询的结果，并将其发送给客户端。

storing result in query cache：服务器将查询结果存储在查询缓存中。

Waiting for query cache lock：当会话正在等待采取查询缓存锁定时，会发生此状态。这种情况可能需要执行一些查询缓存操作，如使查询缓存无效的INSERT或DELETE语句，以及RESET QUERY CACHE等等。

* 事件调度器线程状态（State）值

这些状态适用于事件调度程序线程，创建用于执行调度事件的线程或终止调度程序的线程。

Clearing

调度程序线程或正在执行事件的线程正在终止，即将结束。

Initialized

调度程序线程或将执行事件的线程已初始化。

Waiting for next activation

调度程序具有非空事件队列，但下一次激活是将来。

Waiting for scheduler to stop

线程发出SET GLOBAL event_scheduler=OFF并正在等待调度程序停止。

Waiting on empty queue

调度程序的事件队列是空的，它正在休眠。

* 其他

除了上述几类，还有如复制主线程状态（State）值、复制从库IO线程状态（State）值、复制从库SQL线程（State）值、复制从库Connect线程（State）值

### KILL 命令

KILL 指令用于终止一个正在执行的 SQL。

语法：
```mysql
KILL PROCESS_ID | 'PHYSICAL_PROCESS_ID' | 'ALL'
```
有三种用法：

1、终止一个特定的逻辑 SQL： **KILL PROCESS_ID**  
PROCESS_ID 为 SHOW [FULL] PROCESSLIST 指令返回的 ID 列。

2、终止一个特定的物理 SQL： **KILL 'PHYSICAL_PROCESS_ID'**  
其中的 PHYSICAL_PROCESS_ID 来自SHOW PHYSICAL_PROCESS_ID指令返回的 ID 列。

3、终止当前库上所有通过 DRDS 执行的物理 SQL： **KILL 'ALL'**  
当底层 MySQL/RDS 因为一些 SQL 导致压力非常大的时候，可以使用 KILL 'ALL' 指令终止当前 DRDS 库上所有正在执行的物理 SQL。

参考链接：  
<a href="http://www.ywnds.com/?p=9337" target="_blank">MySQL SHOW PROCESSLIST协助故障诊断</a>  
<a href="https://help.aliyun.com/document_detail/56771.html" target="_blank">SHOW PROCESSLIST 指令与 KILL 指令</a>  
<a href="https://help.aliyun.com/document_detail/71369.html" target="_blank">SHOW PROCESSLIST</a>