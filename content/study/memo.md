---
title: "备忘"
date: 2018-07-10T17:13:30+08:00
categories: ["备忘"]
---
* Linux硬链接和软链接  
文件都有文件名与数据，这在 Linux 上被分成两个部分：用户数据 (user data) 与元数据 (metadata)。
用户数据，即文件数据块 (data block)，数据块是记录文件真实内容的地方；而元数据则是文件的附加属性，如文件大小、创建时间、所有者等信息。
在 Linux 中，元数据中的 inode 号（inode 是文件元数据的一部分但其并不包含文件名，inode 号即索引节点号）才是文件的唯一标识而非文件名。
文件名仅是为了方便人们的记忆和使用，系统或程序通过 inode 号寻找正确的文件数据块。
```$bash
ln source target #硬链接
ln -s source target #软链接
```

* XSS 攻击和 CSRF 攻击的常见防御措施：  
参考链接：<a href="https://github.com/dwqs/blog/issues/68" target="_blank">浅说 XSS 和 CSRF</a>
  1. 防御XSS攻击  
    a. HttpOnly 防止劫取 Cookie  
    b. 用户的输入检查  
    c. 服务端的输出检查  
  2. 防御 CSRF 攻击  
    a. 验证码  
    b. Referer Check  
    c. Token 验证  

* 聚簇索引和非聚簇索引
* nginx中rewrite指令last、break区别
* 组合索引最左优先原则
* <a href="http://www.php.cn/php-weizijiaocheng-383032.html" target="_blank">TP5 自动加载机制详解</a>
* <a href="https://my.oschina.net/EIKPE2lvl3wigMQG/blog/1832646" target="_blank">Hugo+Caddy打造个人博客</a>
* <a href="http://www.cnblogs.com/cnblogsfans/p/5075073.html" target="_blank">Git 在团队中的最佳实践--如何正确使用Git Flow</a>
* <a href="https://www.cnblogs.com/hafiz/p/8146324.html" target="_blank">GitLab配置ssh key</a>
* <a href="https://github.com/dubbo/dubbo-php-framework" target="_blank">dubbo-php-framework(php语言的RPC通讯框架)</a>
* <a href="http://wowubuntu.com/markdown/index.html" target="_blank">Markdown 语法说明 (简体中文版)</a>
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
