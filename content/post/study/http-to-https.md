---
title: "网站HTTP升级HTTPS完全配置手册"
date: 2018-08-17T23:59:35+08:00
categories: ["Web开发"]
---

### 申请证书

证书类型分为DV、OV、EV这三种，这三种有什么区别？

* DV（域名型SSL）：个人站点、iOS应用分发站点、登陆等单纯https加密需求的链接；
* OV（企业型SSL）：企业官网；
* EV（增强型SSL）：对安全需求更强的企业官网、电商、互联网金融网站；

SSL证书的部署类型又分为了单域名、多域名、通配符等类型， 这里以葡萄城官网为例，使用的是OV通配符证书，也就是一张证书可以保护 *.grapecity.com.cn 下的所有二级子域名。大家可以根据自己的需求来选择申请购买。

### 安装证书

证书购买完成后，你就可以下载对应域名的证书文件。根据你Web服务器的不同种类一般证书也会分为多种，请根据自己的实际情况下载安装，一般的常见的Web服务器分为Nginx、Apache、Tomcat、IIS 6、IIS 7/8这几种，下面我们来看一下，证书下载完成后，如何在服务器上安装/配置SSL证书。

#### Nginx

1、首先在Nginx的安装目录下创建cert目录，将下载的全部文件拷贝到cert目录中。  
2、打开 Nginx 安装目录下 conf 目录中的 nginx.conf 文件，找到“HTTPS server”部分。  
3、指定证书路径，为如下示意并保存：  
4、
```
server {
    listen 443;
    server_name 你网站的域名;
    ssl on;
    root html;
    index index.html index.htm;
    ssl_certificate   cert/你的证书文件名.pem;
    ssl_certificate_key  cert/你的证书文件名.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    location / {
        root html;
        index index.html index.htm;
    }
}
```
5、重启Nginx，这时候你的站点应该就已经可以通过https方式访问了

#### Apache

1、在Apache的安装目录下创建cert目录，并且将下载的全部文件拷贝到cert目录中。  
2、打开 Apache 安装目录下的 conf 目录中的 httpd.conf 文件，找到以下内容并去掉“#”：  
3、
```
#LoadModule ssl_module modules/mod_ssl.so
#Include conf/extra/httpd-ssl.conf
```
4、打开Apache安装目录下的conf/extra/httpd-ssl.conf文件（或conf.d/ssl.conf），在配置文件中找到以下语句并配置
```
# 添加 SSL 协议支持协议，去掉不安全的协议
SSLProtocol all -SSLv2 -SSLv3
# 修改加密套件如下
SSLCipherSuite HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM
SSLHonorCipherOrder on
# 证书公钥配置
SSLCertificateFile cert/public.pem
# 证书私钥配置
SSLCertificateKeyFile cert/你的证书文件名.key
# 证书链配置，如果该属性开头有 '#'字符，请删除掉
SSLCertificateChainFile cert/chain.pem
```
5、重启 Apache

#### Tomcat

Tomcat 支持JKS格式证书，但从Tomcat7开始也支持PFX格式证书，两种格式任选其一

1、在Tomcat的安装目录下创建cert目录，并且将下载的全部文件拷贝到cert目录中。
2、找到安装Tomcat目录下该文件server.xml，找到Connection port="8443" 标签，并根据证书类型添加如下相应属性：

**如果是PFX证书**
```
keystoreFile="cert/你的证书文件名.pfx"
keystoreType="PKCS12"
keystorePass="证书密码"
```
**如果是JKS证书**
```
keystoreFile="cert/你的证书文件名.jks"
keystorePass="证书密码"
```
3、重启Tomcat

### 设置跳转

经过上面的步骤，相信各位的网站应该都能以https://domainhost的形式访问了，但细心的小伙伴可能已经发现，网站这个时候http和https同时都能够访问。这就需要设置跳转了，使http请求通过301 redirect到https上去。同样的，我们以不同Web服务类型来说明。

**Nginx**
```
server {
        listen 80;
        server_name 您的域名;
        return 301 https://$server_name$request_uri;
}
```
**Apache**
```
新建.htaccess
RewriteEngine On
RewriteCond %{SERVER_PORT} 80
RewriteRule ^(.*)$ https://%{HTTP_HOST}/$1 [R,L]
```
**Tomcat**
```
在conf/web.xml中的</web-app>前加入

<login-config>

       <!-- Authorization setting for SSL -->

       <auth-method>CLIENT-CERT</auth-method>

       <realm-name>Client Cert Users-only Area</realm-name>

</login-config>

<security-constraint>

       <!-- Authorization setting for SSL -->

       <web-resource-collection >

              <web-resource-name >SSL</web-resource-name>

              <url-pattern>/*</url-pattern>

       </web-resource-collection>

       <user-data-constraint>

              <transport-guarantee>CONFIDENTIAL</transport-guarantee>

       </user-data-constraint>

</security-constraint>　
```

### 总结

至此，网站HTTPS化的工作已经全部完成了，另外多啰嗦的内容就是，HTTPS化了之后还有一些收尾工作需要进行，那就是，请尽量将引用图片资源的路径改为相对路径，如果引用的有站外的js或css等资源，也请将http协议头删除，否则会给你带来一些“惊喜”。

参考链接：  
<a href="https://www.cnblogs.com/powertoolsteam/p/http2https.html" target="_blank">葡萄城官网</a>