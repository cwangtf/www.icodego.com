---
title: "PHP7下安装使用xhprof性能分析工具"
date: 2018-08-28T22:52:35+08:00
categories: ["PHP"]
tags: ["PHP", "性能分析"]
---

最近的任务是做性能优化，一个一个接口分析，用到xhprof，环境PHP7

### 获取
> 该版本从 https://github.com/longxinH/xhprof 获取  
```bash
cd ~
git clone https://github.com/longxinH/xhprof
```

### 安装
```bash
cd xhprof/extension/
phpize
./configure --with-php-config=/php-bin-path/php-config --enable-xhprof
make
sudo make install
```
如果不确定PHP安装路径，可通过以下命令查找
```bash
whereis php
which php
```
出现
```
Installing shared extensions:     /usr/local/php-7.0.14/lib/php/extensions/no-debug-non-zts-20151012/
```
说明编译成功

### 修改php.ini文件
```bash
php -i | grep php.ini //命令查找php.ini文件的位置
```
在/etc/php.ini中增加配置
```bash
[xhprof]
extension=xhprof.so
xhprof.output_dir=/data/www/xhprof/save_output_dir //该目录自定义即可,用来保存xhprof生成的源文件
```
保存后，查看扩展是否安装成功
```bash
php -m|grep xhprof
```
看到输出，说明安装成功，重启php-fpm
```bash
// 第一种方式
ps -ef|grep php-fpm //通过此命令找到php主进程
kill -USR2 pid //pid为php主进程ID
// 第二种方式
kill -USR2 `cat /usr/local/php-7.0.14/var/run/php-fpm.pid`
```

### 将相关文件拷贝至项目根目录
```bash
//切换到下载的 xhprof 目录
cp -r xhprof/xhprof_html  ROOT_PATH/
cp -r xhprof/xhprof_lib ROOT_PATH/
```

### 分析PHP代码段，使用如下
```php
// 启动 xhprof 性能分析器
// XHPROF_FLAGS_NO_BUILTINS、XHPROF_FLAGS_CPU、XHPROF_FLAGS_MEMORY三个参数不加可能会报502
xhprof_enable(XHPROF_FLAGS_NO_BUILTINS | XHPROF_FLAGS_CPU | XHPROF_FLAGS_MEMORY);
// 测试代码段
phpcode();
// 停止 xhprof 分析器
$xhprof_data = xhprof_disable();
// display raw xhprof data for the profiler run
print_r($xhprof_data);
$XHPROF_ROOT = realpath(dirname(__FILE__) .'/..');
include_once $XHPROF_ROOT . "/xhprof_lib/utils/xhprof_lib.php";
include_once $XHPROF_ROOT . "/xhprof_lib/utils/xhprof_runs.php";
// save raw data for this profiler run using default
// implementation of iXHProfRuns.
$xhprof_runs = new XHProfRuns_Default();
// save the run under a namespace "xhprof_foo"
$run_id = $xhprof_runs->save_run($xhprof_data, "xhprof_foo");
echo "---------------\n".
     "Assuming you have set up the http based UI for \n".
     "XHProf at some address, you can view run at \n".
     "http://<xhprof-ui-address>/index.php?run=$run_id&source=xhprof_foo\n".
     "---------------\n";
```

### 查看数据分析报告
浏览器访问 $host_url/xhprof_html/index.php?run=$run_id&source=xhprof_foo

### 图形化调用栈
点击 *[View Full Callgraph]* 显示调用栈  
如果查看的时候报错
```
failed to execute cmd：" dot -Tpng". stderr：sh： dot：command not found
```
```bash
//解决方案
sudo yum install graphviz
```

### xhprof分析报告解析
xhprof分析报告有很多列，含义见下表：

列名|描述
---|---
Function Name|方法名称。
Calls|方法被调用的次数。
Calls%|方法调用次数在同级方法总数调用次数中所占的百分比。
Incl.Wall Time  (microsec)|方法执行花费的时间，包括子方法的执行时间。（单位：微秒）
IWall%|方法执行花费的时间百分比。
Excl. Wall Time  (microsec)|方法本身执行花费的时间，不包括子方法的执行时间。（单位：微秒）
EWall%|方法本身执行花费的时间百分比。
Incl. CPU  (microsecs)|方法执行花费的CPU时间，包括子方法的执行时间。（单位：微秒）
ICpu%|方法执行花费的CPU时间百分比。
Excl. CPU  (microsec)|方法本身执行花费的CPU时间，不包括子方法的执行时间。（单位：微秒）
ECPU%|方法本身执行花费的CPU时间百分比。
Incl.MemUse  (bytes)|方法执行占用的内存，包括子方法执行占用的内存。（单位：字节）
IMemUse%|方法执行占用的内存百分比。
Excl.MemUse  (bytes)|方法本身执行占用的内存，不包括子方法执行占用的内存。（单位：字节）
EMemUse%|方法本身执行占用的内存百分比。
Incl.PeakMemUse  (bytes)|Incl.MemUse峰值。（单位：字节）
IPeakMemUse%|Incl.MemUse峰值百分比。
Excl.PeakMemUse  (bytes)|Excl.MemUse峰值。单位：（字节）
EPeakMemUse%|Excl.MemUse峰值百分比。