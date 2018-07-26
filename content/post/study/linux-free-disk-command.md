---
title: "如何检查 Linux 中的可用磁盘空间"
date: 2018-07-26T21:39:41+08:00
categories: ["Linux"]
tags: ["Linux", "磁盘利用率"]
---
用这里列出的方便的工具来跟踪你的磁盘利用率

> df

`df` 命令意思是 “disk-free”，显示 Linux 系统上可用和已使用的磁盘空间。

`df -h` 以人类可读的格式显示磁盘空间。

`df -a` 显示文件系统的完整磁盘使用情况，即使 Available（可用） 字段为 0。

![df -ha](http://source.icodego.com/image/png/df-ha.png)

`df -T` 显示磁盘使用情况以及每个块的文件系统类型（例如，xfs、ext2、ext3、btrfs 等）。

`df -i` 显示已使用和未使用的 inode。

![df -Ti](http://source.icodego.com/image/png/df-Ti.png)

> du

`du` 显示文件，目录等的磁盘使用情况，默认情况下以 kb 为单位显示。

`du -h` 以人类可读的方式显示所有目录和子目录的磁盘使用情况。

`du -a` 显示所有文件的磁盘使用情况。

![du -ha](http://source.icodego.com/image/png/du-ha.png)

`du -s` 提供特定文件或目录使用的总磁盘空间。

![du -s](http://source.icodego.com/image/png/du-s.png)

> ls -al

`ls -al` 列出了特定目录的全部内容及大小。

![ls -al](http://source.icodego.com/image/png/ls-al.png)

> stat

`stat` <文件/目录>显示文件/目录或文件系统的大小和其他统计信息。

![stat](http://source.icodego.com/image/png/stat.png)

> fdisk -l

`fdisk -l` 显示磁盘大小以及磁盘分区信息。

![fdisk -l](http://source.icodego.com/image/png/fdisk-l.png)

这些是用于检查 Linux 文件空间的大多数内置实用程序。有许多类似的工具，如 Disks（GUI 工具），Ncdu 等，它们也显示磁盘空间的利用率。你有你最喜欢的工具而它不在这个列表上吗？请在评论中分享。