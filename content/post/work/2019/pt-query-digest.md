---
title: "MySQL慢日志分析工具pt-query-digest使用详解"
date: 2019-07-21T22:21:24+08:00
categories: ["MySQL"]
tags: ["MySQL", "慢日志"]
---

### 一、 简介
pt-query-digest是用于分析mysql慢查询的一个工具，它可以分析binlog、General log、slowlog，也可以通过SHOWPROCESSLIST或者通过tcpdump抓取的MySQL协议数据来进行分析。可以把分析结果输出到文件中，分析过程是先对查询语句的条件进行参数化，然后对参数化以后的查询进行分组统计，统计出各查询的执行时间、次数、占比等，可以借助分析结果找出问题进行优化。

### 二、 安装pt-query-digest
#### 2.1 下载

下载地址: https://www.percona.com/doc/percona-toolkit/3.0/index.html

#### 2.2 安装依赖
```bash
yum install -y perl-CPAN perl-Time-HiRes
```
#### 2.3 安装
##### 2.3.1 rpm安装
```bash
cd /usr/local/src
wget percona.com/get/percona-toolkit.rpm
yum install -y percona-toolkit.rpm
rm percona-toolkit.rpm
```
##### 2.3.2 源码安装
```bash
cd /usr/local/src
wget percona.com/get/percona-toolkit.tar.gz
tar zxf percona-toolkit.tar.gz
cd percona-toolkit-2.2.19
perl Makefile.PL PREFIX=/usr/local/percona-toolkit
make && make install
```

>工具安装目录在：/usr/local/percona-toolkit/bin

#### 2.4 工具使用
##### 2.4.1 慢查询日志分析统计
```bash
pt-query-digest /usr/local/mysql/data/slow.log
```
#### 2.4.2 服务器摘要
```bash
pt-summary
```
##### 2.4.3 服务器磁盘监测
```bash
pt-diskstats 
```
##### 2.4.4 mysql服务状态摘要
```bash
pt-mysql-summary -- --user=root --password=root
```

### 三、pt-query-digest语法及重要选项
```bash
pt-query-digest [OPTIONS] [FILES] [DSN]
--create-review-table  当使用--review参数把分析结果输出到表中时，如果没有表就自动创建。
--create-history-table  当使用--history参数把分析结果输出到表中时，如果没有表就自动创建。
--filter  对输入的慢查询按指定的字符串进行匹配过滤后再进行分析
--limit    限制输出结果百分比或数量，默认值是20,即将最慢的20条语句输出，如果是50%则按总响应时间占比从大到小排序，输出到总和达到50%位置截止。
--host  mysql服务器地址
--user  mysql用户名
--password  mysql用户密码
--history 将分析结果保存到表中，分析结果比较详细，下次再使用--history时，如果存在相同的语句，且查询所在的时间区间和历史表中的不同，则会记录到数据表中，可以通过查询同一CHECKSUM来比较某类型查询的历史变化。
--review 将分析结果保存到表中，这个分析只是对查询条件进行参数化，一个类型的查询一条记录，比较简单。当下次使用--review时，如果存在相同的语句分析，就不会记录到数据表中。
--output 分析结果输出类型，值可以是report(标准分析报告)、slowlog(Mysql slow log)、json、json-anon，一般使用report，以便于阅读。
--since 从什么时间开始分析，值为字符串，可以是指定的某个”yyyy-mm-dd [hh:mm:ss]”格式的时间点，也可以是简单的一个时间值：s(秒)、h(小时)、m(分钟)、d(天)，如12h就表示从12小时前开始统计。
--until 截止时间，配合—since可以分析一段时间内的慢查询。
```

### 四、分析pt-query-digest输出结果
* 第一部分：总体统计结果
Overall：总共有多少条查询
Time range：查询执行的时间范围
unique：唯一查询数量，即对查询条件进行参数化以后，总共有多少个不同的查询
total：总计 min：最小 max：最大 avg：平均
95%：把所有值从小到大排列，位置位于95%的那个数，这个数一般最具有参考价值
median：中位数，把所有值从小到大排列，位置位于中间那个数
```bash
# 该工具执行日志分析的用户时间，系统时间，物理内存占用大小，虚拟内存占用大小
# 340ms user time, 140ms system time, 23.99M rss, 203.11M vsz
# 工具执行时间
# Current date: Fri Nov 25 02:37:18 2016
# 运行分析工具的主机名
# Hostname: localhost.localdomain
# 被分析的文件名
# Files: slow.log
# 语句总数量，唯一的语句数量，QPS，并发数
# Overall: 2 total, 2 unique, 0.01 QPS, 0.01x concurrency ________________
# 日志记录的时间范围
# Time range: 2016-11-22 06:06:18 to 06:11:40
# 属性               总计      最小    最大    平均    95%  标准    中等
# Attribute          total     min     max     avg     95%  stddev  median
# ============     ======= ======= ======= ======= ======= ======= =======
# 语句执行时间
# Exec time             3s   640ms      2s      1s      2s   999ms      1s
# 锁占用时间
# Lock time            1ms       0     1ms   723us     1ms     1ms   723us
# 发送到客户端的行数
# Rows sent              5       1       4    2.50       4    2.12    2.50
# select语句扫描行数
# Rows examine     186.17k       0 186.17k  93.09k 186.17k 131.64k  93.09k
# 查询的字符数
# Query size           455      15     440  227.50     440  300.52  227.50
```
* 第二部分：查询分组统计结果
Rank：所有语句的排名，默认按查询时间降序排列，通过–order-by指定
Query ID：语句的ID，（去掉多余空格和文本字符，计算hash值）
Response：总的响应时间
time：该查询在本次分析中总的时间占比
calls：执行次数，即本次分析总共有多少条这种类型的查询语句
R/Call：平均每次执行的响应时间
V/M：响应时间Variance-to-mean的比率
Item：查询对象
```bash
# Profile
# Rank Query ID           Response time Calls R/Call V/M   Item
# ==== ================== ============= ===== ====== ===== ===============
#    1 0xF9A57DD5A41825CA  2.0529 76.2%     1 2.0529  0.00 SELECT
#    2 0x4194D8F83F4F9365  0.6401 23.8%     1 0.6401  0.00 SELECT wx_member_base
```
* 第三部分：每一种查询的详细统计结果
由下面查询的详细统计结果，最上面的表格列出了执行次数、最大、最小、平均、95%等各项目的统计。
ID：查询的ID号，和上图的Query ID对应
Databases：数据库名
Users：各个用户执行的次数（占比）
Query_time distribution ：查询时间分布, 长短体现区间占比，本例中1s-10s之间查询数量是10s以上的两倍。
Tables：查询中涉及到的表
Explain：SQL语句
```bash
# Query 1: 0 QPS, 0x concurrency, ID 0xF9A57DD5A41825CA at byte 802 ______
# This item is included in the report because it matches --limit.
# Scores: V/M = 0.00
# Time range: all events occurred at 2016-11-22 06:11:40
# Attribute    pct   total     min     max     avg     95%  stddev  median
# ============ === ======= ======= ======= ======= ======= ======= =======
# Count         50       1
# Exec time     76      2s      2s      2s      2s      2s       0      2s
# Lock time      0       0       0       0       0       0       0       0
# Rows sent     20       1       1       1       1       1       0       1
# Rows examine   0       0       0       0       0       0       0       0
# Query size     3      15      15      15      15      15       0      15
# String:
# Databases    test
# Hosts        192.168.8.1
# Users        mysql
# Query_time distribution
#   1us
#  10us
# 100us
#   1ms
#  10ms
# 100ms
#    1s  ################################################################
#  10s+
# EXPLAIN /*!50100 PARTITIONS*/
select sleep(2)\G
```

### 五、例子
#### 5.1 直接分析慢查询文件:
```bash
pt-query-digest  slow.log > slow_report.log
```
#### 5.2 分析最近12小时内的查询
```bash
pt-query-digest  --since=12h  slow.log > slow_report2.log
```
#### 5.3 分析指定时间范围内的查询
```bash
pt-query-digest slow.log --since '2017-01-07 09:30:00' --until '2017-01-07 10:00:00'> > slow_report3.log
```
#### 5.4 分析指含有select语句的慢查询
```bash
pt-query-digest --filter '$event->{fingerprint} =~ m/^select/i' slow.log> slow_report4.log
```
#### 5.5 针对某个用户的慢查询
```bash
pt-query-digest --filter '($event->{user} || "") =~ m/^root/i' slow.log> slow_report5.log
```
#### 5.6 查询所有所有的全表扫描或full join的慢查询
```bash
pt-query-digest --filter '(($event->{Full_scan} || "") eq "yes") ||(($event->{Full_join} || "") eq "yes")' slow.log> slow_report6.log
```
#### 5.7 把查询保存到query_review表
```bash
pt-query-digest --user=root –password=abc123 --review  h=localhost,D=test,t=query_review--create-review-table  slow.log
```
#### 5.8 把查询保存到query_history表
```bash
pt-query-digest  --user=root –password=abc123 --review  h=localhost,D=test,t=query_history--create-review-table  slow.log_0001
pt-query-digest  --user=root –password=abc123 --review  h=localhost,D=test,t=query_history--create-review-table  slow.log_0002
```
#### 5.9 通过tcpdump抓取mysql的tcp协议数据，然后再分析
```bash
tcpdump -s 65535 -x -nn -q -tttt -i any -c 1000 port 3306 > mysql.tcp.txt
pt-query-digest --type tcpdump mysql.tcp.txt> slow_report9.log
```
#### 5.10 分析binlog
```bash
mysqlbinlog mysql-bin.000093 > mysql-bin000093.sql
pt-query-digest  --type=binlog  mysql-bin000093.sql > slow_report10.log
```
#### 5.11 分析general log
```bash
pt-query-digest  --type=genlog  localhost.log > slow_report11.log
```

### 六、 将分析结果可视化
使用pt-query-digest分析慢查询日志并将查询分析数据保存到MySQL数据库表中，然后使用应用程序来展示分析结果。目前有基于LAMP的Query-Digest-UI、Anemometer开源项目支持。

#### 6.1 将慢日志插入表中
```bash
$ pt-query-digest \
--user=mha \
--password=123456 \
--review h='10.99.73.9',D=test,t=global_query_review \
--history h='10.99.73.9',D=test,t=global_query_review_history \
--no-report \
--create-review-table \
--create-history-table \
--limit=0% slow.log
```
#### 6.2 查看表信息
```bash
mysql> select * from global_query_review limit 1\G
*************************** 1. row ***************************
checksum: 19915890639818927
fingerprint: select * from `bid` where buyer is not ? order by end_time
sample: SELECT * FROM `bid` where buyer is not null ORDER BY end_time
first_seen: 2017-02-10 15:38:10
last_seen: 2017-02-10 15:38:10
reviewed_by: NULL
reviewed_on: NULL
comments: NULL
1 row in set (0.00 sec)
mysql> select * from global_query_review_history limit 1\G
*************************** 1. row ***************************
checksum: 19915890639818927
sample: SELECT * FROM `bid` where buyer is not null ORDER BY end_time
ts_min: 2017-02-10 15:38:10
ts_max: 2017-02-10 15:38:10
ts_cnt: NULL
Query_time_sum: 2.07545
Query_time_min: 2.07545
Query_time_max: 2.07545
Query_time_pct_95: 2.07545
Query_time_stddev: 0
Query_time_median: 2.07545
Lock_time_sum: 0.001107
Lock_time_min: 0.001107
Lock_time_max: 0.001107
Lock_time_pct_95: 0.001107
Lock_time_stddev: 0
Lock_time_median: 0.001107
Rows_sent_sum: 3089
Rows_sent_min: 3089
Rows_sent_max: 3089
Rows_sent_pct_95: 3089
Rows_sent_stddev: 0
Rows_sent_median: 3089
Rows_examined_sum: 6178
Rows_examined_min: 6178
Rows_examined_max: 6178
Rows_examined_pct_95: 6178
Rows_examined_stddev: 0
Rows_examined_median: 6178
Rows_affected_sum: NULL
Rows_affected_min: NULL
Rows_affected_max: NULL
Rows_affected_pct_95: NULL
Rows_affected_stddev: NULL
Rows_affected_median: NULL
Rows_read_sum: NULL
Rows_read_min: NULL
Rows_read_max: NULL
Rows_read_pct_95: NULL
Rows_read_stddev: NULL
Rows_read_median: NULL
Merge_passes_sum: NULL
Merge_passes_min: NULL
Merge_passes_max: NULL
Merge_passes_pct_95: NULL
Merge_passes_stddev: NULL
Merge_passes_median: NULL
InnoDB_IO_r_ops_min: NULL
InnoDB_IO_r_ops_max: NULL
InnoDB_IO_r_ops_pct_95: NULL
InnoDB_IO_r_ops_stddev: NULL
InnoDB_IO_r_ops_median: NULL
InnoDB_IO_r_bytes_min: NULL
InnoDB_IO_r_bytes_max: NULL
InnoDB_IO_r_bytes_pct_95: NULL
InnoDB_IO_r_bytes_stddev: NULL
InnoDB_IO_r_bytes_median: NULL
InnoDB_IO_r_wait_min: NULL
InnoDB_IO_r_wait_max: NULL
InnoDB_IO_r_wait_pct_95: NULL
InnoDB_IO_r_wait_stddev: NULL
InnoDB_IO_r_wait_median: NULL
InnoDB_rec_lock_wait_min: NULL
InnoDB_rec_lock_wait_max: NULL
InnoDB_rec_lock_wait_pct_95: NULL
InnoDB_rec_lock_wait_stddev: NULL
InnoDB_rec_lock_wait_median: NULL
InnoDB_queue_wait_min: NULL
InnoDB_queue_wait_max: NULL
InnoDB_queue_wait_pct_95: NULL
InnoDB_queue_wait_stddev: NULL
InnoDB_queue_wait_median: NULL
InnoDB_pages_distinct_min: NULL
InnoDB_pages_distinct_max: NULL
InnoDB_pages_distinct_pct_95: NULL
InnoDB_pages_distinct_stddev: NULL
InnoDB_pages_distinct_median: NULL
QC_Hit_cnt: NULL
QC_Hit_sum: NULL
Full_scan_cnt: NULL
Full_scan_sum: NULL
Full_join_cnt: NULL
Full_join_sum: NULL
Tmp_table_cnt: NULL
Tmp_table_sum: NULL
Tmp_table_on_disk_cnt: NULL
Tmp_table_on_disk_sum: NULL
Filesort_cnt: NULL
Filesort_sum: NULL
Filesort_on_disk_cnt: NULL
Filesort_on_disk_sum: NULL
1 row in set (0.00 sec)
```
也可以自己做一个简单的web程序，即可获取慢查询日志的结果。

但不管用什么工具，都需要在服务器有一个脚本把慢日志分析结果存储到一个统一的存储中。我用的比较多的就是Anemometer，支持多数据源，支持按主机过滤，支持按库过滤，支持执行计划，并且支持历史数据（很重要，分析对比使用），基本上是足够使用了。缺点就是不支持认证及权限管理。

Anemometer在global_query_review_history表上多加了两个字段：hostname_max，db_max。分别代表主机名和库名，支持按主机或库过滤的关键。其余的跟上面展示的内容一样。

Anemometer安装配置文档很多，这里就不写了。提供一个客户端慢日志收集脚本如下：

```bash
#!/bin/bash
#****************************************************************#
# ScriptName:/usr/local/sbin/slowquery.sh
# Create Date:2017-03-25
#***************************************************************#
# configure slow log storage database;
pt_db_host="172.18.212.17"
pt_db_port=3306
pt_db_user="slowquerylog"
pt_db_password="123456"
pt_db_database="slow_query_log"
# configure slow log collect database;
mysql_client=`which mysql`
mysql_host="172.18.204.10"
mysql_port=3306
mysql_user="root"
mysql_password="123456"
# configure slow log file position;
slowquery_file=`$mysql_client -h$mysql_host -P$mysql_port -u$mysql_user -p$mysql_password -e "show variables like 'slow_query_log_file';" -BN | awk '{print $2}' 2> /dev/null`
slowquery_dir=`dirname $slowquery_file`
pt_query_digest=`which pt-query-digest`
# configure slow_query_log and slow_query_log_file;
slow_query_log=1
long_query_time=0.1
# configure HOSTNAME;
HOSTNAME=172.18.204.10
# configure slow log;
$mysql_client -h$mysql_host -P$mysql_port -u$mysql_user -p$mysql_password -e "set global slow_query_log=$slow_query_log;set global long_query_time=$long_query_time" 2> /dev/null
# collect mysql slow log into database;
$pt_query_digest --user=$pt_db_user --password=$pt_db_password --port=$pt_db_port --charset=utf8 --review h=$pt_db_host,D=$pt_db_database,t=global_query_review --history h=$pt_db_host,D=$pt_db_database,t=global_query_review_history --no-report --limit=100% --filter="\$event->{add_column} = length(\$event->{arg}) and\$event->{hostname}=\"$HOSTNAME\" " $slowquery_file 
if [ $? = 0 ];
# set a new slow log;
new_mysql_slow_log=`$mysql_client -h$mysql_host -P$mysql_port -u$mysql_user -p$mysql_password -e "select concat('$slowquery_dir','/mysql_slow_query_',date_format(now(),'%Y%m%d%H'));" -BN 2> /dev/null`
# configure slow log;
$mysql_client -h$mysql_host -P$mysql_port -u$mysql_user -p$mysql_password -e "set global slow_query_log_file = '$new_mysql_slow_log';" 2> /dev/null
# delete log before 1 days;
datetime=`date -d "1 day ago" +%Y%m%d%H`
if [ -d $slowquery_dir ];then
cd $slowquery_dir
rm -fr mysql_slow_query_${datetime}*
fi
fi
```

或者

```bash
#!/bin/bash
#****************************************************************#
# ScriptName:/home/mysql/bin/slow_query_log.sh
# Create Date:2017-03-25
#***************************************************************#
# configure slow log storage database;
pt_db_host="172.18.212.17"
pt_db_port=3306
pt_db_user="root"
pt_db_password="123456"
pt_db_database="slow_query_log"
# configure slow log collect database;
mysql_client=`which mysql`
mysql_host='172.18.204.10'
mysql_port=3306
mysql_user="root"
mysql_password="123456"
# configure slow log file position;
slowquery_file=`$mysql_client -h$mysql_host -P$mysql_port -u$mysql_user -p$mysql_password -e "show variables like 'slow_query_log_file';" -BN | awk '{print $2}' 2> /dev/null`
slowquery_dir=`dirname "$slowquery_file"`
pt_query_digest=`which pt-query-digest`
# configure slow_query_log and slow_query_log_file;
slow_query_log=1
long_query_time=0.5
# configure HOSTNAME;
HOSTNAME="172.18.204.10:3306"
# configure slow log;
slow_log=`$mysql_client -h$mysql_host -P$mysql_port -u$mysql_user -p$mysql_password -e "show variables like 'slow_query_log';" -BN | awk '{print $2}' 2> /dev/null`
if [ $slow_log == "OFF" ];then
$mysql_client -h$mysql_host -P$mysql_port -u$mysql_user -p$mysql_password -e "set global slow_query_log=$slow_query_log;set global long_query_time=$long_query_time" 2> /dev/null
fi
# collect mysql slow log into database;
$pt_query_digest --user=$pt_db_user --password=$pt_db_password --port=$pt_db_port --charset=utf8 --review h=$pt_db_host,D=$pt_db_database,t=global_query_review --history h=$pt_db_host,D=$pt_db_database,t=global_query_review_history --no-report --limit=100% --filter="\$event->{add_column} = length(\$event->{arg}) and\$event->{hostname}=\"$HOSTNAME\" " $slowquery_file && echo > ${slowquery_file}
```

对于IP和端口，可以通过自己的环境使用命令提取出来，然后做成变量形式。然后在客户端做个执行计划就可以了，多久收集一次慢日志可以自定义。这种方式收集慢查询不是非常实时，如果需要实时性高的，可以使用filebeat类似工具把慢日志全部收集到一台服务器，可以做到实时。

参考：<a href="https://www.cloudmmu.com/360.html" target="_blank">MySQL慢日志分析工具pt-query-digest使用详解</a>