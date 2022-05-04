---
layout: article
mathjax: true
title: Django+uWsgi+Nginx部署
category: Algorithm
date: 2022-05-04 10:00:00 +0800
tags: [后端]
---
一份Django+uWsgi+Nginx部署流程笔记

## 概述
简单描述了笔者0基础操作Django+uWsgi+Nginx的部署流程（业内通用流程）。
Django不用多说。

### uWsgi
WSGI协议（通讯协议）：Python用于Web开发的协议(用于处理Web服务器和应用程序（APP）的交互信息)（把http通讯的过程抽象出来(请求数据，响应数据的封装)，开发者只负责处理中途的数据） uwsgi协议（传输协议，速度很快）：uWSGI程序实现的一个自有的协议(采用二进制来存储数据，之前的协议都是使用字符串，所以在存储空间和解析速度上，都更快)

**注意：**
WSGI是一种通信协议。
uwsgi是一种线路协议而不是通信协议，在此常用于在uWSGI服务器与其他网络服务器的数据通信。

### Nginx
1. 安全（Nginx 作为专业服务器，暴露在公网相对比较安全）
2. 能更好地处理静态资源（一些http request header）
3. Nginx也可以缓存一些动态内容Nginx可以更好地配合CDN
4. 可以进行多台机器的负载均衡

## 安装必要软件
**需要使用python3，如无特殊说明，笔者下文均是建立在python3的环境下**

### 安装uWsgi
建议使用pip3安装最新的uwsgi
笔者homebrew安装的uWsgi不完，执行测试代码：
```shell
uwsgi --http xxx
```
会报 `--http is ambiguous`
换用pip3后OK，装好之后，检测：
```shell
uwsgi --version

2.0.20
```

### 安装Django

```shell
pip3 install django

#安装成功会在list里面显示Django
pip3 list

Package    Version
---------- -------
asgiref    3.5.1
Django     4.0.4
pip        21.2.4
setuptools 58.1.0
sqlparse   0.4.2
uWSGI      2.0.20
```
### 安装Nginx
笔者是用homebrew安装的nginx

安装完成后，nginx位于`/opt/homebrew/bin/nginx`
conf文件位于`/opt/homebrew/etc/nginx`目录下

每个人安装的方式不同，可能路径也不同，各位需要记住，尤其是*conf*这个目录，因为后续会改到配置文件

有个简单的法子只读自己的conf在哪，执行下面的命令：
`nginx -t`
![nginx1]({{site.url}}/assets/images/posts/nginx1.png)
这个命令是用于验证我们的conf是否合法的。

## 创建你的第一个Django项目
打开命令行，cd 到一个你想放置你代码的目录，然后运行以下命令：
`django-admin startproject DjangoHelloWorld`
然后cd到自己的项目目录下创建自己的app`python manage.py startapp helloworld`。根目录下创建static（用于存放静态资源）和templates（模板，如一些html）文件夹。完成后，整个项目目录如下：
![nginx1]({{site.url}}/assets/images/posts/nginx1.png)

其中**uwsgi.ini**和**helloworld.conf**uWsgi和nginx的配置文件，下文会说到。
