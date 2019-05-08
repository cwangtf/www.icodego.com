---
title: "Linux curl访问https发生错误"
date: 2019-05-08T23:31:23+08:00
categories: ["Linux", "curl"]
tags: ["Linux"]
---

最近工作上碰到CentOS 6环境curl访问https发生如下错误  
![curl_error](http://source.icodego.com/image/jpg/curl_error.jpg)

通过以下方式解决
```bash
sudo yum update -y nss
sudo yum update -y curl
```
mark一下