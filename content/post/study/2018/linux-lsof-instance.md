---
title: "Linux lsof 命令的实用案例"
date: 2018-07-25T22:17:55+08:00
categories: ["Linux"]
tags: ["Linux", "lsof"]
---
> lsof 简介
  
lsof（list open files）是一个列出当前系统中所有打开文件的工具

Linux中一切皆文件，所以在系统中，被打开的文件可以是普通文件、目录、网络文件系统中的文件、字符设备、管道、socket等

如何知道现在系统打开的是哪些文件？及这些文件的相关信息呢？

lsof命令就是帮我们查看打开文件的信息的

> 基本用法

*查看进程打开的文件*

例如查看mysql在操作哪些文件
```bash
# lsof -c mysql
```

*查看文件对应的进程*

例如查看系统日志文件是在被谁操作
```bash
# lsof /var/log/messages
```

> 实用案例

*（1）查看某进程正在操作哪些文件*

命令
```bash
# lsof -p PID
```


这个命令很有用，例如系统I/O负载过高时，我们可以使用top、iotop找出是哪些进程导致了I/O压力，然后就使用lsof命令查看这个进程正在操作哪些文件，从而分析出现异常的原因

*（2）查看某端口正在被谁使用*

使用 lsof 还可以查找使用了某个端口的进程

比如发现系统有个不明端口，就需要使用lsof命令检查是谁在使用，来判定是否出现安全问题

命令
```bash
# lsof -i:端口号
```

*（3）恢复删除的文件*

linux中删除文件要谨慎，不像windows那么容易被恢复，如果文件被不小心删除，可以使用lsof来恢复，但前提是：这个文件正在被某个进程使用

还有，当系统受到入侵时，常见的情况是日志文件被删除，以掩盖攻击者的踪迹，如果能恢复日志文件，对解决安全问题非常有帮助

现在假设/var/log/messages被删除了，首先来确认一下当前是否有进程正在使用这个文件，如果有，就可以恢复了
```bash
//查看哪个进程在使用此文件
# lsof | grep message

//重写文件
# cat /proc/端口号/fd/2 > /var/log/messages
```