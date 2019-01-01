---
title: "SSH 端口转发"
date: 2019-01-01T23:16:11+08:00
categories: ["Linux"]
tags: ["Linux", "SSH", "端口转发"]
---

工作中有时候会遇到内网的某些服务外网访问不了，例如数据库，了解下端口转发

SSH 端口转发功能能够将其他 TCP 端口的网络数据通过 SSH 链接来转发，并且自动提供了相应的加密及解密服务。其实这一技术就是我们常常听说的隧道(tunnel)技术，原因是 SSH 为其他 TCP 链接提供了一个安全的通道来进行传输。

### 本地端口转发
```bash
$ ssh -L <local port>:<remote host>:<remote port> <SSH server host>
```

### 远程端口转发
```bash
$ ssh -R <local port>:<remote host>:<remote port> <SSH server host>
```

### 动态端口转发
```bash
$ ssh -D <local port> <SSH Server Host>
```