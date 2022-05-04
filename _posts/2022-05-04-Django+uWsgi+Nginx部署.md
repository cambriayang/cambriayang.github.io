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
笔者homebrew安装的uWsgi不完整，执行测试代码：

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

![nginx2]({{site.url}}/assets/images/posts/nginx2.png)

其中**uwsgi.ini**和**helloworld.conf**uWsgi和nginx的配置文件，下文会说到。

## 配置
需要配置uWsgi和nginx

### 工程配置
首先项目工程下settings里面的`ALLOWED_HOSTS`需要设置，笔者为了方便，直接使用了
`ALLOWED_HOSTS = ['*']`
当然读者也可以使用具体的配置，精细化管理。静态文件夹配置也在settings.py里面，见上图的目录中的static。

```python
STATIC_URL = 'static/'
STATIC_ROOT = './static'

STATIC_PATH = os.path.join(BASE_DIR, 'static')
STATIC_URL = '/static/'
STATICFILES_DIRS = (
    STATIC_URL,
)
```

配置好可以使用`python manage.py collectstatic`，会将静态资源收集在上面的static文件夹下。

然后在urls.py里面配置入口：

```python
from django.contrib import admin
from django.urls import path, re_path
from helloworld import views

urlpatterns = [
    path('admin/', admin.site.urls),
    re_path('^$', views.index, name='index'),  # 添加的路由
]
```

helloworld下面的views.py就是本次的测试代码：

```python
def index(request):
    return render(request, "helloworld/index.html")
```
html的实现如下：

```html
<h1>Test Django Server</h1>
<h2>Done</h2>
<h2>Haha</h2>
```

*都是很简单的测试代码，大家可以自由发挥*

### 配置uWsgi
其实uWsgin可以通过命令行参数传入，但是笔者觉得还是写个配置文件比较好，起码以后有据可查。

```python
[uwsgi]
#使用nginx连接时使用
socket=127.0.0.1:9000
#直接做web服务器使用
#http=127.0.0.1:9000

master = true         //主进程
#项目目录
chdir=./
#项目中wsgi.py文件的目录，相对于项目目录
wsgi-file=./DjangoHelloWorld/wsgi.py
module = DjangoHelloWorld.wsgi:application
processes=4
threads=2
post-buffering = 65535
buffer-size = 65535
harakiri-verbose = true
harakiri = 300
max-requests = 2000
vacuum = true
#uid = nginx
#gid = nginx
```

**具体参数定义和解释大家可以上网查阅，就不赘述了**

### 配置Nginx
首先配置自己App的conf：

```shell
server {
        listen       80;
        server_name  localhost;

        #静态文件的位置
        location /static {
            alias /Users/argost/Codes/cambriayang/DjangoHelloWorld/static;
        }

        location / {            
            include  /opt/homebrew/etc/nginx/uwsgi_params;
            uwsgi_pass  127.0.0.1:9000;             
        }
    }
```

核心部分如上所示。

**注意**：`include`{:.error}处需要找到uwsgi_params所在目录，上图是用的笔者的uwsgi_params目录，读者需要自行替换。`uwsgi_passs`{:.error}和上面的uwsgi.ini配置要一致。

最后，将自己的App的conf加入的nginx的conf（笔者的目录在：`/opt/homebrew/etc/nginx/nginx.conf`）中去，如下所示：
可以在nginx.conf中加入include语句：

```shell
include path/to/your/helloworld.conf;
```

## 运行
在`uwsgi.ini`所在目录（或者带目录执行也OK）执行` uwsgi --ini uwsgi.ini `，成功之后的截图如下：
![nginx3]({{site.url}}/assets/images/posts/nginx3.png)

然后执行`nginx`就可以在浏览器里输入我们配置好的url+port+path的方式访问啦。如(笔者这里测试nginx使用的默认端口80，所以后面没带port)：

```shell
http://localhost/
```

![nginx4]({{site.url}}/assets/images/posts/nginx3.png)

可以看到，我们的测试html如期显示在服务器上了（笔者使用的是自己机器，部署到各种云也是一样的）。
后续如果项目代码有更改，只需要改好代码，上传服务器就可以啦。

## 语法糖

{:.success} 
启动: nginx

{:.info}
判断配置文件是否有效: nginx -t

{:.warning}
关闭nginx: pkill -9 nginx

{:.error}
修改了配置文件重新load: nginx -s reload