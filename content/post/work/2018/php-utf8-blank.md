---
title: "空格无法替换解决思路"
date: 2018-08-01T23:16:01+08:00
categories: ["PHP"]
tags: ["PHP", "utf8空格替换"]
---

今天碰到一个同事问utf8空格无法替换问题，这个问题之前解决过，今天总结一下

### 编码规则

解决问题理论基础是对编码的理解，以下是几篇推荐文章

<a href="http://www.imkevinyang.com/2009/02/字符编解码的故事（ascii，ansi，unicode，utf-8区别）.html" target="_blank">字符编解码的故事（ASCII，ANSI，Unicode，Utf-8区别）</a>

<a href="http://www.imkevinyang.com/2010/06/关于字符编码，你所需要知道的.html" target="_blank">关于字符编码，你所需要知道的（ASCII,Unicode,Utf-8,GB2312…）</a>

<a href="http://www.cnblogs.com/skynet/archive/2011/05/03/2035105.html" target="_blank">字符集和字符编码（Charset & Encoding）</a>

（可以这样理解：Unicode是字符集，UTF-32/ UTF-16/ UTF-8是三种字符编码方案。）

### php的trim函数

trim — 去除字符串首尾处的空白字符（或者其他字符）

string trim ( string $str [, string $character_mask = " \t\n\r\0\x0B" ] )

此函数返回字符串 str 去除首尾空白字符后的结果。如果不指定第二个参数，trim() 将去除这些字符：
```
" " (ASCII 32 (0x20))，普通空格符。
"\t" (ASCII 9 (0x09))，制表符。
"\n" (ASCII 10 (0x0A))，换行符。
"\r" (ASCII 13 (0x0D))，回车符。
"\0" (ASCII 0 (0x00))，空字节符。
"\x0B" (ASCII 11 (0x0B))，垂直制表符。
```
但空白字符并不止这些，比如全角空格(ascii:227)和一些控制字符，乱码字符等等。全角空格是碰到比较多的情况，如确定空白字符是全角空格可以直接
```php
$value = str_replace("全角空格"," ",$value);
```
这样可以解决大部分问题。如果还没解决，恭喜，你遇到了变态数据。

### 思路

对于变态数据可以考虑用以下方法处理

1、分割字符串，将字符串分割成一个字节为单位的字符组：`str_split($str)`

2、查看空白字符编码。（比如遇到一个变态字符串前面的空格字符是：194 160 194 160，基本可以确定是这个字符的问题）

3、替换掉空白编码 。比如用正则函数：
```php
$value = preg_replace("/^[\s\v".chr(194).chr(160)."]+/","", $value); //替换开头空字符
$value = preg_replace("/[\s\v".chr(194).chr(160)."]+$/","", $value); //替换结尾空字符
```

### 例子

如下代码：
```php
//比如字符串： “　abc”(前面是两个全角空格)
$str = "　　abc";
$sArray = str_split($str);
foreach ($sArray as $s){
    var_dump(ord($s));
}
```
输出：
```console
int(227)
int(128)
int(128)
int(227)
int(128)
int(128)
int(97)
int(98)
int(99)
```
发现a(97)前面有两轮
```console
int(227)
int(128)
int(128)
```
断定这就是一个utf-8的空白字符，以下代码去除这个字符就可以
```php
$str = preg_replace("/^[\s\v".chr(227).chr(128)."]+/","", $str); //替换开头空字符
$str = preg_replace("/[\s\v".chr(227).chr(128)."]+$/","", $str); //替换结尾空字符

var_dump($str);
```
输出：
```console
string(3) "abc"
```
成功去除空白字符

今天的字符串
```php
$text = '上海交通安全知识宣传教育活动开展多元非常；发挥 回复的规划虚报火车富后才更好电饭锅';
echo ord(' ');die;
```
输出
```console
194
```
用以下代码：
```php
$text = '上海交通安全知识宣传教育活动开展多元非常；发挥 回复的规划虚报火车富后才更好电饭锅';
$text = str_replace(chr(194).chr(160), '', $text);
var_dump($text);die;
```
输出
```console
string(120) "上海交通安全知识宣传教育活动开展多元非常；发挥回复的规划虚报火车富后才更好电饭锅"
```
解决