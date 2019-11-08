title: Nginx 配置 SSL 证书 + 让ghost博客系统用上HTTPS
date: 2015-12-03 01:20:35
categories: 安全
thumbnail: /images/https1.jpg
banner: /images/https1.jpg
tags: [SSL, 安全]
---
前两天在DO主机上搭建了Ghost博客系统，看到很多博客都用上了SSL加密，为了注重隐私和安全，也开始动手配置SSL证书。

<!--more-->
![cover-image](/images/https1.jpg)
# 使用 OpenSSL 生成 SSL Key 和 CSR
跳过自签证书的步骤，直接使用第三方可信任的SSL证书，普通的 SSL 证书认证分两种形式，一种是 DV（Domain Validated），还有一种是 OV （Organization Validated），前者只需要验证域名，后者需要验证你的组织或公司，在安全性方面，肯定是后者要好。
这里我选择了免费的沃通DV证书
首先在vps上使用OpenSSL生成key和csr，命令如下：

```
$ openssl req -new -newkey rsa:2048 -sha256 -nodes -out example_com.csr -keyout example_com.key -subj "/C=CN/ST=Beijing/L=Beijing/O=Example Inc./OU=Web Security/CN=example.com"  
```
命令包含信息如下:

- 域名，也称为 Common Name，因为特殊的证书不一定是域名：example.com
- 组织或公司名字（Organization）：Example, Inc.
部门（Department）：可以不填写，这里我们写 Web Security
- 城市（City）：Beijing
- 省份（State / Province）：Beijing
- 国家（Country）：CN
- 加密强度：2048 位，如果你的机器性能强劲，也可以选择 4096 位

会自动在当前目录生成 example_com.csr 和 example_com.key 这两个文件

其中 example_com.csr 中的内容需要我们提交给SSL认证机构，通过之后会得到一个crt文件，将crt csr 和key这三个文件一起放入/etc/ssl/private/

而 example_com.key 是需要用在 Nginx 配置里和 example_com.crt 配合使用的，需要好好保管，千万别泄露给任何第三方。

# setup SSL for Ghost
Nginx 代理配置文件,并将代理指向到本地的Ghost端口，最新的配置文件在、/etc/nginx/conf.d/ghost.conf

```
server {  
        listen 80;
        server_name raighne.xyz www.raighne.xyz;

        if ($http_user_agent !~* baidu.com){
                return 301 https://$host$request_uri;
        }
        location / {
                proxy_set_header   X-Real-IP $remote_addr;
                proxy_set_header   Host      $http_host;
                proxy_pass         http://127.0.0.1:2368;
                client_max_body_size 35m;
        }
}


server {  
        listen 443 ssl http2;
        server_name raighne.xyz www.raighne.xyz;
        ssl_certificate        /etc/ssl/private/example_com.crt;
        ssl_certificate_key    /etc/ssl/private/example_com.key;
        location / {
                proxy_set_header   X-Real-IP $remote_addr;
                proxy_set_header   Host      $http_host;
                proxy_pass         http://127.0.0.1:2368;
                client_max_body_size 35m;
        }
}
```
Restart nginx

```
$ nginx -t && nginx -s reload
```
```
$ sudo service nginx restart
```

OK,成功，最后，在直接浏览器访问，输入http://example.com/ 地址，看下是否可以正常跳转到https://example.com

本文内容出自由以下链接整理而成，感谢.

http://support.ghost.org/setup-ssl-self-hosted-ghost/

https://luolei.org/ghost-https-baidu-seo-support/
