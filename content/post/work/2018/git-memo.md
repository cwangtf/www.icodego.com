---
title: "Git操作报错解决备忘"
date: 2018-12-09T21:27:16+08:00
categories: ["Git"]
tags: ["Git", "解决办法", "备忘"]
---

记录一些工作中碰到的Git操作报错解决方法

1、Git切换分支提示“The following untracked working tree files would be overwritten by checkout”

提示一些未被Git跟踪的文件将会被重写

解决办法  
git 2.11 (包括)之后版本
```bash
git clean  -d  -fx .
```
之前版本
```bash
git clean  -d  -fx ""
```
* `-x` 意味着删除了被忽略的文件以及git未知的文件。  
* `-d` 表示除了未跟踪的文件外，还删除未跟踪的目录。
* `-f` 删除当前目录下所有没有被track(跟踪)的文件

注意：执行此命令需要非常小心，因为可能会删除一些未被git跟踪的配置文件

因此，做一次检查命令会比较好
```bash
git clean -dfxn
//或
git clean -dfx --dry-run
```
运行这个命令可以先查看哪些文件将受影响

相关知识点
```bash
git clean -n
```
是一次clean的演习, 告诉你哪些文件会被删除. 记住他不会真正的删除文件, 只是一个提醒.
```bash
git clean -f
```
删除当前目录下所有没有track过的文件. 他不会删除.gitignore文件里面指定的文件夹和文件, 不管这些文件有没有被track过.
```bash
git clean -f <path>
```
删除指定路径下的没有被track过的文件.
```bash
git clean -df
```
删除当前目录下没有被track过的文件和文件夹.
```bash
git clean -xf
```
删除当前目录下所有没有track过的文件. 不管他是否是.gitignore文件里面指定的文件夹和文件.

`git reset --hard`和`git clean -f`是一对好基友. 结合使用他们能让你的工作目录完全回退到最近一次commit的时候. 

`git clean`对于刚编译过的项目也非常有用. 如, 他能轻易删除掉编译后生成的.o和.exe等文件.  这个在打包要发布一个release的时候非常有用. 