---
title: "备忘"
date: 2018-07-10T17:13:30+08:00
categories: ["备忘"]
---
#### Linux硬链接和软链接  
文件都有文件名与数据，这在 Linux 上被分成两个部分：用户数据 (user data) 与元数据 (metadata)。
用户数据，即文件数据块 (data block)，数据块是记录文件真实内容的地方；而元数据则是文件的附加属性，如文件大小、创建时间、所有者等信息。
在 Linux 中，元数据中的 inode 号（inode 是文件元数据的一部分但其并不包含文件名，inode 号即索引节点号）才是文件的唯一标识而非文件名。
文件名仅是为了方便人们的记忆和使用，系统或程序通过 inode 号寻找正确的文件数据块。
```$bash
ln source target #硬链接
ln -s source target #软链接
```

#### XSS 攻击和 CSRF 攻击的常见防御措施  
参考链接：<a href="https://github.com/dwqs/blog/issues/68" target="_blank">浅说 XSS 和 CSRF</a>
  1. 防御XSS攻击  
    a. HttpOnly 防止劫取 Cookie  
    b. 用户的输入检查  
    c. 服务端的输出检查  
  2. 防御 CSRF 攻击  
    a. 验证码  
    b. Referer Check  
    c. Token 验证  

#### 强大的xargs命令  

*示例1*

例如文件 urls.txt 中有一个url列表，现在想下载他们，可以使用命令一次完成：
```bash
cat urls.txt | xargs wget
```
xargs 会把 cat 的输出结果作为参数传给 wget

*示例2*
  
再比如需要杀掉 tomcat 进程
```bash
ps -ax | grep tomcat | grep -v grep | awk '{print $1}' | xargs kill -9
```
grep tomcat 过滤出含有 tomcat 的进程

grep -v grep 是排除含有 grep 的进程

awk '{print $1}' 取得进程号那列内容

xargs kill -9 把前面取得的tomcat进程号传给 kill命令

*示例3*
  
如果要传递的命令中需要多个参数，如 cp 有2个参数，xargs 要把之前命令的输出作为其中一个参数传给 cp
```bash
ls *.txt | xargs -i cp {} /tmp
```

#### PHP-FPM配置优化

> PHP-FPM管理的方式是一个master主进程，多个pool进程池，多个worker子进程。其中每个进程池监听一个socket套接字

> 其中的worker子进程实际处理连接请求，master主进程负责管理子进程：

1. `master`进程，设置1s定时器，通过`socket`文件监听

2. 在`pm=dynamic`时，如果`idle worker`数量<`pm.min_spare_servers`，创建新的子进程

3. 在`pm=dynamic`时，如果`idle worker`数量>`pm.max_spare_servers`，杀死多余的空闲子进程

4. 在`pm=ondemand`时，如果`idle worker`空闲时间>`pm.process_idle_timeout`，杀死该空闲进程

5. 当连接到达时，检测如果`worker`数量>`pm.max_children`，打印`warning`日志，退出；如果无异常，使用`idle worker`服务，或者新建`worker`服务

##### 保障基本安全

> 我们为了避免PHP-FPM主进程由于某些糟糕的PHP代码挂掉，需要设置重启的全局配置：

```
#如果在1min内有10个子进程被中断失效，重启主进程
emergency_restart_threshold = 10
emergency_restart_interval = 1m
```
##### 进程数调优

> 每一个子进程同时只能服务一次连接，所以控制同时存在多少个进程数就很重要，如果过少会导致很多不必要的重建和销毁的开销，如果过多又会占用过多的内存，影响其他服务使用。

> 我们应该测试自己的PHP进程使用多少内存，一般来说刚启动时是8M左右，运行一段时间由于内存泄漏和缓存会上涨到30M左右，所以你需要根据自己的预期内存大小设定进程的数量。同时根据进程池的数量来看一个进程管理器的子进程数量限制。

测试平均PHP子进程占用的内存：
```bash
$ps auxf | grep php | grep -v grep
work     26829  0.0  0.0 715976  4712 ?        Ss   Jul11   0:00 php-fpm: master process (./etc/php-fpm.conf)
work     21889  0.0  0.0 729076 29668 ?        S    03:12   0:20  \_ php-fpm: pool www         
work     21273  0.0  0.0 728928 31380 ?        S    03:25   0:21  \_ php-fpm: pool www         
work     15114  0.0  0.0 728052 29084 ?        S    03:40   0:19  \_ php-fpm: pool www         
work     17072  0.0  0.0 728800 34240 ?        S    03:54   0:22  \_ php-fpm: pool www         
work     22763  0.0  0.0 727904 20352 ?        S    11:29   0:04  \_ php-fpm: pool www         
work     38545  0.0  0.0 727796 19484 ?        S    12:34   0:01  \_ php-fpm: pool www

// 共占用的内存数量
$ps auxf | grep php | grep -v grep | grep -v master | awk '{sum+=$6} END {print sum}'
162712

// 所有的子进程数量
$ ps auxf | grep php | grep -v grep | grep -v master | wc -l 
6
```
> 可以看到第6列，每一个子进程的内存占用大概在19-34M之间（单位为KB）。平均的内存占用为162712KB/6 = 27.1M。

查看服务器总的内存大小
```bash
$ free -g
             total       used       free     shared    buffers     cached
Mem:           157        141         15          0          4        123
-/+ buffers/cache:         13        143
Swap:            0          0          0
```
> 可以看出我的服务器总得内存大小是157G（-g采用了G的单位）。

进程数限制
> 此时如果我们分配全部的内存给PHP-FPM使用，那么进程数可以限制在157000/27 = 5814,但是由于我的服务器同时服务了很多内容，所以我们可以向下调整到512个进程数：
```
process.max = 512
pm = dynamic
pm.max_children = 512
pm.start_servers = 16
pm.min_spare_servers = 8
pm.max_spare_serveres = 30
```
防止内存泄漏
> 由于糟糕的插件和库，内存泄漏时有发生，所以我们需要对每一个子进程服务的请求数量做限制，防止无限制的内存泄漏：
```
pm.max_requests = 1000
```
重启
> 如果上面的配置都按照你的实际需求和环境配置好了，不要忘记重启PHP-FPM服务。

参考：  
<a href="http://www.php.cn/php-weizijiaocheng-406483.html" target="_blank">关于php-fpm的进程数管理</a>

#### Linux下批量替换文件内容方法

1、查找  
`find . -type f -name "*.php"|xargs grep 'findstring'`  
2、查找并替换  
`find -name '要查找的文件名' | xargs perl -pi -e 's|被替换的字符串|替换后的字符串|g'`  
3、批量修改文件夹权限  
`find . -type -d -name *.php|xargs chmod 755`  
4、批量修改文件权限  
`find . -type -f -name *.php|xargs chmod 644`

参考：  
<a href="https://www.cnblogs.com/fjping0606/p/4428850.html" target="_blank">Linux下批量替换文件内容方法</a>

#### Chrome工具集合

`chrome://about`
  
抓包工具`chrome://net-internals`，对开发调试有帮助

* 聚簇索引和非聚簇索引
* nginx中rewrite指令last、break区别
* 组合索引最左优先原则
* <a href="http://www.php.cn/php-weizijiaocheng-383032.html" target="_blank">TP5 自动加载机制详解</a>
* <a href="https://my.oschina.net/EIKPE2lvl3wigMQG/blog/1832646" target="_blank">Hugo+Caddy打造个人博客</a>
* <a href="http://www.cnblogs.com/cnblogsfans/p/5075073.html" target="_blank">Git 在团队中的最佳实践--如何正确使用Git Flow</a>
* <a href="https://www.cnblogs.com/hafiz/p/8146324.html" target="_blank">GitLab配置ssh key</a>
* <a href="https://github.com/dubbo/dubbo-php-framework" target="_blank">dubbo-php-framework(php语言的RPC通讯框架)</a>
* <a href="https://www.jianshu.com/p/191d1e21f7ed" target="_blank">markdown基本语法</a>
* <a href="https://yq.aliyun.com/articles/74395" target="_blank">GitLab的安装及使用教程</a>
* <a href="http://man.linuxde.net/tar" target="_blank">tar命令</a>
* <a href="https://www.kancloud.cn/manual/thinkphp5/118008" target="_blank">ThinkPHP5.0目录结构</a>
* <a href="http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html" target="_blank">常用 Git 命令清单</a>
* <a href="https://psr.phphub.org" target="_blank">PHP 标准规范</a>
* <a href="https://blog.csdn.net/shmilychan/article/details/73433804" target="_blank">Redis+Twemproxy+HAProxy集群</a>
* <a href="https://blog.csdn.net/zhuxineli/article/details/14455029" target="_blank">MYSQL explain详解</a>
* <a href="http://825635381.iteye.com/blog/2276077" target="_blank">高并发的核心技术-幂等的实现方案</a>
* <a href="https://www.jianshu.com/p/a4beee06220c" target="_blank">TCP三次握手和四次挥手深入实践</a>
* <a href="https://github.com/dwqs/blog/issues/24" target="_blank">29个你必须知道的Linux命令</a>
* <a href="https://36kr.com/p/5073181.html" target="_blank">今日头条Go建千亿级微服务的实践</a>
* <a href="http://www.cnblogs.com/xuning/p/8464625.html" target="_blank">高可用Redis服务架构分析与搭建</a>
* <a href="https://www.oschina.net/translate/mysql-high-availability-at-github" target="_blank">GitHub 的 MySQL 高可用性实践分享</a>
* <a href="https://www.cnblogs.com/kismetv/p/8654978.html" target="_blank">深入学习Redis系列</a>
* <a href="https://juejin.im/post/5b45cee0e51d45194b18cdbc" target="_blank">理解分布式系统中的缓存架构系列</a>
* <a href="https://www.itcodemonkey.com/article/7191.html" target="_blank">SpringBoot学习</a>
* <a href="https://github.com/xingshaocheng/architect-awesome" target="_blank">《后端架构师技术图谱》</a>
