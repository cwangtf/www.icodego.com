---
title: "Nginx写IO占用高故障处理"
date: 2018-08-29T22:49:39+08:00
categories: ["Nginx"]
tags: ["Nginx", "PHP", "故障处理"]
---

### 问题描述

突然收到一台服务器负载过高告警，网站打开缓慢

### 问题分析

1、使用 top 命令看到cpu行的 iowait 达到了70%以上，断定是IO负载过高的原因

2、使用 iotop -o 命令发现Nginx的写IO特别大，并且在上一步的top命令看到Nginx的进程状态为D，表示Nginx在等待IO已经为僵死状态

这时候可以知道是Nginx产生大量写操作导致的系统负载过高了，但还不能知道具体Nginx在写什么文件

3、找到其中一个nginx worker进程的pid，使用 lsof -p pid 列出来的文件发现除了一些系统库文件及日志文件，还有相当多的fastcgi_temp/xxx文件,有可能与这些文件有关联

4、使用 strace -p pid 追踪，发现nginx进程对某个fd进行大量的写操作，与 lsof 命令列出来的文件刚好符合

5、使用 iostat 1 输出的大量写io的分区与fastcgi_temp所在分区相符合

猜测可能是外部正在上传大量的大文件给php-fpm，于是通过EZHTTP的小工具来查看实时流量,发现入站流量其实不大

### 解决方案

知道了是 fastcgi_temp io 压力大，目前无法短时间从根本上解决问题，决定先紧急处理一下，把 fastcgi_temp 指向 /dev/shm，也就是映射到了内存，重启nginx之后服务恢复了正常，之后再找开发人员协同查找解决根本问题