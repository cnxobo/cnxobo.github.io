---
title: Nginx 新手入门
tags:
  - nginx
categories:
  - - ops
date: 2026-06-09 20:00:00
---

## 安装
虽然Nginx从源码编译也很简单，但是建议新手还是使用系统预构建的包。
### RedHat系
> CentOS/RHEL/ Oracle Linux/AlmaLinux/Rocky Linux
```Shell
sudo yum install epel-release
sudo yum update
sudo yum install nginx
```
### Debian系
>Debian/Ubuntu
```Shell
sudo apt-get update
sudo apt-get install nginx
```
[Installing NGINX Open Source](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/)

## 确认版本及配置

查看版本号
```Shell
nginx -v
```
`nginx version: nginx/1.23.2`

查看版本及配置
```Shell
nginx -V
```
```
nginx version: nginx/1.23.2
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --user=www --group=www --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user=nginx --group=nginx --with-file-aio --with-ipv6 --with-http_auth_request_module --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_addition_module --with-http_xslt_module=dynamic --with-http_image_filter_module=dynamic --with-http_geoip_module=dynamic --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_slice_module --with-http_stub_status_module --with-http_perl_module=dynamic --with-mail=dynamic --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream=dynamic --with-stream_ssl_module --with-google_perftools_module --with-debug --with-stream_ssl_preread_module
```
这里我们主要关注`--prefix`和`--conf-path`配置
>`--prefix=/usr/share/nginx`和`--conf-path=/etc/nginx/nginx.conf`

prefix 决定了nginx里相对路径配置的路径前缀。
conf-path 决定了nginx配置文件位置。不同的Linux发行版配置文件位置都可能不一样，可以通过这个配置快速定位配置位置。
