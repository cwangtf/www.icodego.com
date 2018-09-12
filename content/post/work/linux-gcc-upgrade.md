---
title: "CentOS升级GCC"
date: 2018-09-12T23:35:03+08:00
categories: ["Linux"]
tags: ["CentOS", "GCC"]
---

鸟哥博客对PHP7性能提升的Tips有一条，使用新的编译器，推荐GCC 4.8以上，PHP会开启Global Register for opline and execute_data支持, 这个会带来5%左右的性能提升  
下面通过shell脚本进行升级，耗时跟服务器性能及带宽有关，比较久
```bash
cd /usr/src/
vim upgradeGcc.sh
```
在upgradeGcc.sh输入
```bash
#!/bin/bash
#在非root账户下，使用sudo命令
#获取源码
sudo wget http://ftp.gnu.org/gnu/gcc/gcc-8.2.0/gcc-8.2.0.tar.gz
#wget http://ftp.gnu.org/gnu/gcc/gcc-8.2.0/gcc-8.2.0.tar.gz

#解压
sudo tar -xvf gcc-8.2.0.tar.gz
#tar -xvf gcc-8.2.0.tar.gz


cd gcc-8.2.0
sudo ./contrib/download_prerequisites
#./contrib/download_prerequisites
cd ..

#建立编译输出目录
sudo mkdir gcc-build-8.2.0
#mkdir gcc-build-8.2.0

#进入下面目录，执行命令，生成Makefile文件
cd gcc-build-8.2.0
sudo ../gcc-8.2.0/configure --enable-checking=release --enable-languages=c,c++ --disable-multilib
#../gcc-8.2.0/configure --enable-checking=release --enable-languages=c,c++ --disable-multilib

#执行命令进行编译，此处利用4个job，数值与服务器CPU核数有关，建议数值为核数，此值不宜设置过高
sudo make -j4
#make -j4

#安装
sudo make install
#make install
```
执行以下命令，等待安装完成，时间略久
```bash
chmod 777 upgradeGcc.sh
./upgradeGcc.sh
```
检查版本
```bash
[root@wonderful src]# gcc -v
使用内建 specs。
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/libexec/gcc/x86_64-redhat-linux/4.8.5/lto-wrapper
目标：x86_64-redhat-linux
配置为：../configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-bootstrap --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-linker-hash-style=gnu --enable-languages=c,c++,objc,obj-c++,java,fortran,ada,go,lto --enable-plugin --enable-initfini-array --disable-libgcj --with-isl=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/isl-install --with-cloog=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/cloog-install --enable-gnu-indirect-function --with-tune=generic --with-arch_32=x86-64 --build=x86_64-redhat-linux
线程模型：posix
gcc 版本 4.8.5 20150623 (Red Hat 4.8.5-28) (GCC)
```