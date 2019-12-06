---
layout: post
title:  "CentOS 7 安装 Nginx 并配置 HTTPS 访问"
date:   2019-10-21 10:33:00+0800
categories: centos nginx letsencrypt
---
## 安装 Nginx

通过 yum 安装 Nginx 是最简单快捷的方式，而且便于日后的维护和更新。但是 yum 安装也有其局限，比如 Nginx 用来获取真实 IP 的 ngx_http_realip_module 模块默认是不包含在发行包里的，需要在配置编译时指定参数才可以，这就需要从源码编译安装。

通过 yum 安装 Nginx：

```
$ yum update -y
$ yum install -y epel-release
$ yum install -y nginx
```

配置开机自动启动并启动 Nginx：

```
$ systemctl enable nginx
$ systemctl start nginx
```

## 获取证书

Let's Encrypt 提供免费的 HTTPS 证书，需要使用 Certbot 进行证书的获取与更新，而 Certbot 其实是使用 Python 开发的一个开源工具。官方教程依然在使用 yum 安装 Python 2 版本的 Certbot，Python 2 即将退出舞台，推荐使用 Python 3。CentOS 7 系统自带 Python 2，需要首先安装 Python 3，然后使用 pip3 安装 Certbot。

安装 Python 3.6：

```
$ yum -y install python36 python36-libs python36-devel python36-pip
$ pip3.6 install --upgrade pip
```

一般在通过 pip 安装 Python 依赖时，都推荐先创建虚拟环境，然后安装进虚拟环境中，这样可以保证系统级别的依赖库不被污染，也便于多版本管理。但是这里安装 Certbot 是为了提供一个系统级别的服务，所以直接安装进系统默认依赖库即可。

安装 Certbot：

```
$ pip3.6 install certbot-nginx
```

根据提示为域名配置证书：

```
$ certbot --nginx
```

测试证书更新：

```
$ certbot renew --dry-run
```

## 配置证书自动更新

从 Let's Encrypt 获取的证书有效期 90 天，所以需要定期检查证书是否将要到期，在快到期时进行更新操作。

使用 crontab 配置证书自动更新：

```
$ crontab -e
```

输入以下内容：

```
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

0 2,4 * * 2 certbot renew
```

更新操作配置为每周二的凌晨 2 点和 4 点分别执行一次。
