---
title: "ssh免密登录Linux"
date: 2018-08-26T22:31:45+08:00
categories: ["Linux"]
tags: ["Linux", "ssh"]
---

### 前言

惯用的登录姿势
```bash
# ssh -p 端口 用户名@IP 
```
输入密码后登录

ssh免密码登录方式
```bash
# ssh server_name
```
回车就可登录，server_name是自定义的命名，记住就行

只要三步就可以实现，创建密钥，本地配置，服务器端配置。Mac详细步骤如下：

### 1、创建秘钥（公钥和私钥）
打开一个终端，输入以下命令
```bash
# cd
# cd .ssh
#ssh-keygen -t rsa -f my_service
```
说明：

第一行，进入用户主目录。

第二行，.ssh 文件夹如果没有的话，创建一个。

第三行，rsa表示加密方式，-f my_service 表示创建的密钥的文件名字。

此时会在 .ssh 文件夹中生成两个文件，一个是my_service,一个是my_service.pub，也就是私钥和公钥。

### 2、配置ssh config文件
```bash
# vim ~/.ssh/config
```
打开配置文件，追加以下内容：
```bash
Host service_name
Hostname IP地址
Port 22
User root
IdentityFile ~/.ssh/my_service
```
保存并退出。

### 3、服务器端的配置

使用scp命令将my_service.pub上传的服务器家目录，放置到.ssh目录下。

将my_service.pub文件内容追加到默认验证文件authorized_keys中，保存退出。

配置完事，打开终端，输入命令
```bash
# ssh service_name
```
回车，登录