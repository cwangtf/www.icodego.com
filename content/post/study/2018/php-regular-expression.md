---
title: "9个实用的PHP正则表达式"
date: 2018-09-16T23:53:34+08:00
categories: ["PHP"]
tags: ["PHP", "正则表达式"]
---

1. 验证Email地址  
这是一个用于验证电子邮件的正则表达式。但它并不是高效、完美的解决方案。在此不推荐使用。
![email-regular](http://source.icodego.com/image/jpg/email-regular.jpg)
为了更加有效验证电子邮件地址，推荐使用filer_var。
![php-validate-email](http://source.icodego.com/image/jpg/php-validate-email.jpg)

2. 验证用户名  
这是一个用于验证用户名的实例，其中包括字母、数字(AZ，az，09)、下划线以及最低5个字符，最大20个字符。同时，也可以根据需要，对最小值和最大值做合理的修改。
![php-validate-username](http://source.icodego.com/image/jpg/php-regular-username.jpg)

3. 验证电话号码  
这是一个验证美国电话号码的实例。
![php-validate-phonenum](http://source.icodego.com/image/jpg/php-regular-phonenum.jpg)

4. 验证IP地址  
这是一个用来验证IPv4地址的实例。
![php-validate-ip](http://source.icodego.com/image/jpg/php-regular-ip.jpg)

5. 验证邮政编码  
这是一个用来验证邮政编码的实例。
![php-validate-zipcode](http://source.icodego.com/image/jpg/php-regular-zipcode.jpg)

6. 验证信用卡号
![php-validate-credit-card-num](http://source.icodego.com/image/jpg/php-regular-credit-card-num.jpg)

7. 验证域名
![php-validate-url](http://source.icodego.com/image/jpg/php-regular-url.jpg)

8. 从特定URL中提取域名
![php-match-url](http://source.icodego.com/image/jpg/php-regular-match-url.jpg)

9. 将文中关键词高亮显示
![php-preg-replace](http://source.icodego.com/image/jpg/php-regular-preg-replace.jpg)