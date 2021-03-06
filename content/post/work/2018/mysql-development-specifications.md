---
title: "数据库开发规范"
date: 2018-07-19T20:26:10+08:00
categories: ["MySQL"]
tags: ["MySQL", "开发规范"]
---
数据库开发规范定义：开发规范是针对内部开发的一系列建议或规则, 由DBA制定(如果有DBA的话)。  
开发规范本身也包含几部分：基本命名和约束规范，字段设计规范，索引规范，使用规范。

*规范存在意义*

* 保证线上数据库schema规范  
* 减少出问题概率  
* 方便自动化管理  
* 规范需要长期坚持，对开发和DBA是一个双赢的事情  

#### 基本命名和约束规范

* 表字符集选择UTF8 ，如果需要存储emoj表情，需要使用UTF8mb4(MySQL 5.5.3以后支持)  
* 存储引擎使用InnoDB  
* 变长字符串尽量使用varchar varbinary  
* 不在数据库中存储图片、文件等  
* 单表数据量控制在1亿以下  
* 库名、表名、字段名不使用保留字  
* 库名、表名、字段名、索引名使用小写字母，以下划线分割 ，需要见名知意  
* 库表名不要设计过长，尽可能用最少的字符表达出表的用途  

#### 字段规范

* 所有字段均定义为NOT NULL ，除非你真的想存Null  
* 字段类型在满足需求条件下越小越好，使用UNSIGNED存储非负整数 ，实际使用时候存储负数场景不多  
* 使用TIMESTAMP存储时间  
* 使用varchar存储变长字符串 ，当然要注意varchar(M)里的M指的是字符数不是字节数；使用UNSIGNED INT存储IPv4 地址而不是CHAR(15) ，这种方式只能存储IPv4，存储不了IPv6  
* 使用DECIMAL存储精确浮点数，用float有的时候会有问题  
* 少用blob text  

关于为什么定义不使用Null的原因

* 1.浪费存储空间，因为InnoDB需要有额外一个字节存储  
* 2.表内默认值Null过多会影响优化器选择执行计划  

#### 索引规范

* 单个索引字段数不超过5，单表索引数量不超过5，索引设计遵循B+ Tree索引最左前缀匹配原则  
* 选择区分度高的列作为索引  
* 建立的索引能覆盖80%主要的查询，不求全，解决问题的主要矛盾  
* DML和order by和group by字段要建立合适的索引  
* 避免索引的隐式转换  
* 避免冗余索引  

关于索引规范，一定要记住索引这个东西是一把双刃剑，在加速读的同时也引入了很多额外的写入和锁，降低写入能力，这也是为什么要控制索引数原因。之前看到过不少人给表里每个字段都建了索引，其实对查询可能起不到什么作用。

冗余索引例子

* idx_abc(a,b,c)
* idx_a(a) 冗余
* idx_ab(a,b) 冗余

隐式转换例子

字段:remark varchar(50) NOT Null
```mysql
MySQL>SELECT id, gift_code FROM gift WHERE deal_id = 640 AND remark=115127; 1 row in set (0.14 sec)
MySQL>SELECT id, gift_code FROM pool_gift WHEREdeal_id = 640 AND remark=‘115127’; 1 row in set (0.005 sec)
```
字段定义为varchar，但传入的值是个int，就会导致全表扫描，要求程序端要做好类型检查

#### SQL类规范

* 尽量不使用存储过程、触发器、函数等  
* 避免使用大表的JOIN，MySQL优化器对join优化策略过于简单  
* 避免在数据库中进行数学运算和其他大量计算任务  
* SQL合并，主要是指的DML时候多个value合并，减少和数据库交互  
* 合理的分页，尤其大分页  
* UPDATE、DELETE语句不使用LIMIT ，容易造成主从不一致  

参考链接：  
<a href="https://mp.weixin.qq.com/s/-TRiWDYFhaO7wqjMPTNGBA" target="_blank">单表60亿记录等大数据场景的MySQL优化和运维之道</a>