---
title: "HTTP访问控制(CORS)"
date: 2018-12-02T22:03:38+08:00
categories: ["HTTP"]
tags: ["HTTP", "CORS", "跨域"]
---

### 简介

跨域资源共享(CORS) 是一种机制，它使用额外的 HTTP 头来告诉浏览器  让运行在一个 origin (domain) 上的Web应用被准许访问来自不同源服务器上的指定的资源。当一个资源从与该资源本身所在的服务器不同的域或端口请求一个资源时，资源会发起一个跨域 HTTP 请求。

跨域资源共享（ CORS ）机制允许 Web 应用服务器进行跨域访问控制，从而使跨域数据传输得以安全进行。现代浏览器支持在 API 容器中（例如 XMLHttpRequest 或 Fetch ）使用 CORS，以降低跨域 HTTP 请求所带来的风险。

### 简单请求

某些请求不会触发 CORS 预检请求。本文称这样的请求为“简单请求”，请注意，该术语并不属于 Fetch （其中定义了 CORS）规范。若请求满足所有下述条件，则该请求可视为“简单请求”：

* 使用下列方法之一：
   + GET
   + HEAD
   + POST
* Fetch 规范定义了对 CORS 安全的首部字段集合，不得人为设置该集合之外的其他首部字段。该集合为：
   + Accept
   + Accept-Language
   + Content-Language
   + Content-Type （需要注意额外的限制）
   + DPR
   + Downlink
   + Save-Data
   + Viewport-Width
   + Width
* Content-Type 的值仅限于下列三者之一：
   + text/plain
   + multipart/form-data
   + application/x-www-form-urlencoded
* 请求中的任意XMLHttpRequestUpload 对象均没有注册任何事件监听器；XMLHttpRequestUpload 对象可以使用 XMLHttpRequest.upload 属性访问。
* 请求中没有使用 ReadableStream 对象。

```
Access-Control-Allow-Origin: http://foo.example
```

### 预检请求

与前述简单请求不同，“需预检的请求”要求必须首先使用 OPTIONS   方法发起一个预检请求到服务器，以获知服务器是否允许该实际请求。"预检请求“的使用，可以避免跨域请求对服务器的用户数据产生未预期的影响。

当请求满足下述任一条件时，即应首先发送预检请求：

* 使用了下面任一 HTTP 方法：
   + PUT
   + DELETE
   + CONNECT
   + OPTIONS
   + TRACE
   + PATCH
* 人为设置了对 CORS 安全的首部字段集合之外的其他首部字段。该集合为：
   + Accept
   + Accept-Language
   + Content-Language
   + Content-Type (but note the additional requirements below)
   + DPR
   + Downlink
   + Save-Data
   + Viewport-Width
   + Width
* Content-Type 的值不属于下列之一:
   + application/x-www-form-urlencoded
   + multipart/form-data
   + text/plain
* 请求中的XMLHttpRequestUpload 对象注册了任意多个事件监听器。
* 请求中使用了ReadableStream对象。

### HTTP 响应首部字段

本节列出了规范所定义的响应首部字段。
```
Access-Control-Allow-Origin
```
响应首部中可以携带一个`Access-Control-Allow-Origin`字段，其语法如下:

> Access-Control-Allow-Origin: <origin> | *

其中，origin 参数的值指定了允许访问该资源的外域 URI。对于不需要携带身份凭证的请求，服务器可以指定该字段的值为通配符，表示允许来自所有域的请求。

例如，下面的字段值将允许来自 http://mozilla.com 的请求：

> Access-Control-Allow-Origin: http://mozilla.com

如果服务端指定了具体的域名而非“*”，那么响应首部中的 Vary 字段的值必须包含 Origin。这将告诉客户端：服务器对不同的源站返回不同的内容。
```
Access-Control-Expose-Headers`
```
注：在跨域访问时，XMLHttpRequest对象的getResponseHeader()方法只能拿到一些最基本的响应头，Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma，如果要访问其他头，则需要服务器设置本响应头。

`Access-Control-Expose-Headers`头让服务器把允许浏览器访问的头放入白名单，例如：

> Access-Control-Expose-Headers: X-My-Custom-Header, X-Another-Custom-Header

这样浏览器就能够通过getResponseHeader访问`X-My-Custom-Header`和`X-Another-Custom-Header`响应头了。
```
Access-Control-Max-Age
```
`Access-Control-Max-Age`头指定了preflight请求的结果能够被缓存多久。

> Access-Control-Max-Age: <delta-seconds>

`delta-seconds`参数表示preflight请求的结果在多少秒内有效。
```
Access-Control-Allow-Credentials
```
`Access-Control-Allow-Credentials`头指定了当浏览器的credentials设置为true时是否允许浏览器读取response的内容。当用在对preflight预检测请求的响应中时，它指定了实际的请求是否可以使用credentials。请注意：简单 GET 请求不会被预检；如果对此类请求的响应中不包含该字段，这个响应将被忽略掉，并且浏览器也不会将相应内容返回给网页。

> Access-Control-Allow-Credentials: true
```
Access-Control-Allow-Methods
```
`Access-Control-Allow-Methods`首部字段用于预检请求的响应。其指明了实际请求所允许使用的 HTTP 方法。

> Access-Control-Allow-Methods: <method>[, <method>]*
```
Access-Control-Allow-Headers
```
`Access-Control-Allow-Headers`首部字段用于预检请求的响应。其指明了实际请求中允许携带的首部字段。

> Access-Control-Allow-Headers: <field-name>[, <field-name>]*

### HTTP 请求首部字段

本节列出了可用于发起跨域请求的首部字段。请注意，这些首部字段无须手动设置。 当开发者使用 XMLHttpRequest 对象发起跨域请求时，它们已经被设置就绪。
```
Origin
```
`Origin`首部字段表明预检请求或实际请求的源站。

> Origin: <origin>

origin 参数的值为源站 URI。它不包含任何路径信息，只是服务器名称。

Note: 有时候将该字段的值设置为空字符串是有用的，例如，当源站是一个 data URL 时。

注意，不管是否为跨域请求，ORIGIN 字段总是被发送。
```
Access-Control-Request-Method
```
`Access-Control-Request-Method`首部字段用于预检请求。其作用是，将实际请求所使用的 HTTP 方法告诉服务器。

> Access-Control-Request-Method: <method>
```
Access-Control-Request-Headers节
```
`Access-Control-Request-Headers`首部字段用于预检请求。其作用是，将实际请求所携带的首部字段告诉服务器。

> Access-Control-Request-Headers: <field-name>[, <field-name>]*

### 前端代码示例

```html
<!DOCTYPE html>
<html lang="en">
<head>
   <meta charset="UTF-8">
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   <meta http-equiv="X-UA-Compatible" content="ie=edge">
   <title>CORS跨域请求</title>
   <script>
       function createCORSRequest(method, url) {
           var xhr = new XMLHttpRequest();
           if ("withCredentials" in xhr) {
               xhr.open(method, url, true);
           } else if (typeof XDomainRequest != "undefined") {
               xhr = new XDomainRequest();
               xhr.open(method, url);
           } else {
               xhr = null;
           }
           return xhr;
       }

       window.onload = function () {
           var oBtn = document.getElementById('btn1');
           oBtn.onclick = function () {
               var xhr = createCORSRequest("get", "http://wpdic.com/cors.php");
               if (xhr) {
                   xhr.onload = function () {
                       var json = JSON.parse(xhr.responseText);
                       alert(json.a);
                   };
                   xhr.onerror = function () {
                       alert('请求失败.');
                   };
                   xhr.send();
               }
           };
       };
   </script>
</head>
<body>
   <input type="button" value="获取数据" id="btn1">
</body>
</html>
```
注意点：

1.上面代码兼容IE8,因为用了XDomainRequest

2.其它代码你就当成XMLHttpRequset用，别考虑什么2.0不2.0的

3.如果你想post数据，可以往 xhr.send()里面搞

### 后端代码示例

```php
<?php
header('content-type:application:json;charset=utf8');
header('Access-Control-Allow-Origin:*');
header('Access-Control-Allow-Methods:GET,POST');
header('Access-Control-Allow-Credentials: true');
header('Access-Control-Allow-Headers:x-requested-with,content-type');
$str = '{"a":1,"b":2,"c":3,"d":4,"e":5}'; 
echo $str;
?>
```

注意点：

1.Access-Control-Allow-Origin:* 表示允许任何域名跨域访问，如果需要指定某域名才允许跨域访问，只需把Access-Control-Allow-Origin:*改为Access-Control-Allow-Origin:允许的域名,实际工作也要这么做

2.Access-Control-Allow-Methods:GET,POST  规定允许的方法，建议控制严格些，不要随意放开DELETE之类的权限

3.Access-Control-Allow-Credentials

该字段可选。它的值是一个布尔值，表示是否允许发送Cookie。默认情况下，Cookie不包括在CORS请求之中。设为true，即表示服务器明确许可，Cookie可以包含在请求中，一起发给服务器。这个值也只能设为true，如果服务器不要浏览器发送Cookie，删除该字段即可。

### 最后，面试常考问题：

CORS和JSONP的应用场景区别？

---

CORS要求浏览器(>IE10)和服务器的同时支持，是跨域的根本解决方法，由浏览器自动完成。优点在于功能更加强大支持各种HTTP Method，缺点是兼容性不如JSONP。  

**参考**

* [HTTP访问控制（CORS）](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)