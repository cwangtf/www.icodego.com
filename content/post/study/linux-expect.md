---
title: "Linux expect命令详解"
date: 2018-12-23T14:27:52+08:00
categories: ["Linux"]
tags: ["Linux", "expect"]
---

Expect是一个用来处理`交互`的命令

#### 关键命令

expect中四个比较关键命令是：`spawn`、 `send`、 `expect`、 `interact`

`spawn` 创建一个新的进程    
`send`  向进程发送字符串  
`expect` 从进程接收字符串  
`interact`  把当前进程的控制权交还给用户，允许用户在控制台进行交互  

#### 用法简介

ssh自动登录脚本
```bash
#!/usr/bin/expect -f
set timeout 30
set user <用户名>
set host <ip地址>
set port <端口>
set password <密码>

spawn ssh $user@$host -p$port
expect "*password:*"
send "$password\r"
interact
expect eof
```
1、[!/usr/bin/expect]  
使用哪种shell执行

2、[set timeout -1]  
设置超时时间30s

3、[set user/host/port/password args]  
设置变量

4、[spawn ssh $user@$host -p$port]  
创建一个执行交互命令的进程

5、[expect "*password:*"]  
判断上次命令输出中是否包含`password:`字符串，有立即返回，没有等待30s再返回

6、[send "$password\r"]  
执行交互操作，跟控制台手动输入密码一样

7、[interact]  
将控制权交还控制台

8、[expect eof]  
结束

注：expect脚本必须以interact或expect eof结束，执行自动化任务通常expect eof就够了。