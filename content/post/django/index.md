---
title: Django
description: Django was invented to meet fast-moving newsroom deadlines, while satisfying the tough requirements of experienced web developers.
date: 2020-02-10 22:13:06
categories:
- PYTHON
tags:
- Python
- Django
- Web
- ORM
---

[Django英文官方文档](https://docs.djangoproject.com/en/3.1/)

[Django中文官方文档](https://docs.djangoproject.com/zh-hans/3.1/)

<!-- markdownlint-disable-file MD036 -->
<!-- markdownlint-disable-file MD024 -->

## Django请求生命周期

### 流程

```python
"""
浏览器发送请求 => WEB网关服务接口 => Django框架
                                  -> 中间件 middleware (-> 缓存数据库 拿到则返回)
                                  -> 路由层 urls.py
                                  -> 视图层 views.py
                                    -> 模板层 templates
                                    -> 模型层 modes.py
                                  -> 拿到数据/模板 渲染后按路径返回
                            
请求经过WEB网关服务接口：
  - django自带的是wsgiref
    1. 请求来的时候 解析请求 并封装为request对象
    2. 响应走的时候 打包处理
  - django自带的wsgiref模块本身能够支持的并发量很小 最多1000左右
  - 上线会换成 nginx(反向代理) + uwsgi
  
  WSGI/wsgiref/uwsgi是什么关系?
    - WSGI 是协议
    - wsgiref/uwsgi是实现该协议的功能模块
    
请求经过Django框架：
  -> 先经过Django中间件(类似与django的保安 门户)
  -> 路由层 urls.py 路由匹配与分发 识别路由匹配对应的视图函数
  -> 视图层 views.py 网站整体的业务逻辑
     - 视图层可能需要模板 -> 模板层(templates文件夹) 网站所有的html页面
     - 视图层可能需要数据 -> 模型层 models.py **重要** ORM
     - 视图层拿到数据之后 渲染并按路径返回
"""
```

### 扩展

**缓存数据库**

```python
"""
提前已经将你想要的数据准备好 你来直接拿即可
 - 减轻后端压力
 - 提高效率和响应时间
    
缓存数据库在请求流程中的位置   
  - 经过中间件之后 直接请求缓存数据库
    - 拿到 直接返回
    - 没拿到 走正常流程 然后存一份到缓存数据库并返回
    
当你在修改数据的时候 你会发现数据并不是立即修改完的
而是需要经过一段时间才会修改(还在访问缓存的内容)
"""
```

## 注意事项

```python
# 如何让你的计算机能够正常的启动Django项目

"""
  1. 计算机的名称不能有中文
  2. 一个Pycharm窗口只开一个项目
  3. 项目里面所有的文件也尽量不要出现中文
  4. Python解释器结论使用 3.4-3.6之间的版本
    4.1. 如果你的项目报错 点击最后一个报错信息 去源码中把逗号删掉
"""

# Django版本问题
推荐使用：LTS 官方长期支持维护的版本
"""
1.x 2.x 3.x(成熟版本出来后考虑)
1.x 和 2.x 本身差距并不大
"""  
```

## 安装

```bash
# 安装django == 指定安装版本
pip install django==x.x.x

# e.g. 直接安装 会自动卸载旧的安装新的
pip install django==1.11.11

# 验证是否安装成功
django-admin  # 看是否输出帮助文档
```

## Django基本操作

- **命令行操作**

```python
# 1. 创建django项目
# 如果已经实现创建项目目录 [目录] 直接换为 .(当前目录)
django-admin startproject [项目名] [目录]
django-admin startproject [name] .

# e.g.
cd MinhoDjango  # 进入已存在的目录 作为项目根目录
django-admin startprojcet mysite .

# 2. 启动django目录
python manage.py runserver  # 默认: 127.0.0.1:8080


# 3. 创建应用
"""
Next, start your first app by running python manage.py startapp [app_label].

应用名应该做到见名知意：
  - user
  - order
  - web
  - ...
"""
python manage.py startapp app01
```

- **Pycharm创建**

```python
"""
1. File --> New Project --> Django项目

2. 启动
    2.1 还是用命令行启动
    2.2 点击绿色小箭头即可
   
3. 创建应用app
    3.1 还是用命令行创建
    3.2 Tools --> Run manage.py Task... # 进入shell
        startapp app01  # 创建app01 应用

4. 修改端口号以及创建server
   右上角 --> Edit Configure

"""
```

- **区别**

**命令行与pycharm创建的区别**

```python
"""
  1. 命令行创建不会自动有 root_projects/templates 文件夹 需要你自己手动创建 而pycharm创建会自动帮你创建并且还会自动在配置文件中配置对应的路径
  意味着 你在用命令创建django项目的时候 不单单需要创建templates文件夹 还需要去配置文件中配置路径
"""

# 命令行创建
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
    },
]

# Pycharm创建
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates']
    },       
]
```

## 应用

```python
"""
django 是一款专门用来开发app的web框架

django框架就类似一所大学(空壳子 需要自己创建具有不同功能的app)
app就类似大虚额里面的各个学院(学院--具体功能的app)

比如开发淘宝：
  - 订单相关
  - 用户相关
  - 投诉相关
  - 创建不同的app对应不同的功能
  
学课系统：
  - 学生功能
  - 老师功能
  
一个app就是一个独立的功能模块
"""
```

**创建的应用 一定要去配置文件中注册**

```python
# Application definition
"""
一定要注册 否则应用不生效
创建出来的应用 第一步先去配置文件中注册
备注: 在使用Pycharm创建项目的时候 Pycharm可以帮你创建一个app并自动注册(仅一个)
"""

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app01.apps.App01Config',  # 全写
    'app02',  # 简写
]  # 全写 or 简写 选择其中一种即可

```

## 主要文件介绍

```python
"""
MinhoDjango/
    - manage.py          django的入口文件
    - db.sqlite3         django自带的sqlite3数据库(小型数据库 功能不全还有BUG)
    - mysite/
        - __init__.py
        - settings.py    配置文件
        - urls.py        路由与视图对应关系(路由层)
        - wsgi.py        wsgiref模块(忽略)
    - app01
        - migrations/    所有的数据库迁移记录(类日志记录)
            - __init__.py
        - __init__.py
        - admin.py       django后台管理
        - apps.py        注册使用
        - models.py      数据库相关的 模型类(模型层 ORM)
        - tests.py       测试文件(忽略)
        - views.py       视图函数(视图层)
"""
```

## 配置文件

```python
# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True  # 上线之后改为Flase

ALLOWED_HOSTS = []  # 上线之后可以写 '*'  允许所有

# 注册的app(app就是自带的功能模块) django一创建出来 自带6个功能模块
# Application definition
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app01.apps.App01Config',
    'app02',
]

# Django中间件
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# HTML存放路径配置
TEMPLATES = []

# 项目指定数据库配置 默认sqlite3
# Database
# https://docs.djangoproject.com/en/3.1/ref/settings/#databases
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

# Password validation
# https://docs.djangoproject.com/en/3.1/ref/settings/#auth-password-validators
AUTH_PASSWORD_VALIDATORS = []

# 日志记录实践
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {process:d} {thread:d} {message}',
            'style': '{',
        },
        'simple': {
            'format': '{levelname} {message}',
            'style': '{',
        },
    },
    'filters': {
        'require_debug_true': {
            '()': 'django.utils.log.RequireDebugTrue',
        },
    },
    'handlers': {
        'console': {
            'level': 'INFO',
            'filters': ['require_debug_true'],
            'class': 'logging.StreamHandler',
            'formatter': 'simple'
        },
        'file': {
            'level': 'INFO',  # 实际开发建议使用ERROR
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': os.path.join(os.path.dirname(BASE_DIR), 'logs', 'chaos.log'),
            'maxBytes': 200 * 1024 * 1024,  # 100M
            'backupCount': 10,              # 日志文件备份数量
            'formatter': 'verbose',         # 日志格式：详细
            'encoding': 'utf-8'             # 文件编码格式
        }
    },
    'loggers': {
        'django': {
            'handlers': ['console', 'file'],
            'propagate': True,
        }
    }
}
```

## Django小白必会三板斧

### HttpResponse

**HttpResponse 直接返回字符串**

```python
from django.shortcuts import HttpResponse

# Create your views here.

# 视图函数必须要接收一个形参：request
def index(request):
    """
    :param request: 请求相关的所有数据对象 比之前的environ更好用 是对象 直接. 取属性 比字典好用
    :return:
    """
    data = HttpResponse('Hello, Django')
    print('1', data)
    print('2', type(data))
    return HttpResponse("Hello, Django")

>>> 1 <HttpResponse status_code=200, "text/html; charset=utf-8">
>>> 2 <class 'django.http.response.HttpResponse'>
```

### render

**render 返回html文件 使用渲染之后的模板**

```python
from django.shortcuts import render

# Create your views here.
def index(request):
    return render(request, 'myfirst.html')

# 命令行创建 需要配置模板目录
$ mkdir ${root_projects}/templates
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')]
    },
]
```

**render两种传值方式**

```python
def index(request):
    """
    1. 直接传一个字典 模板使用key进行使用 精确传值 使用哪个传哪个
    
    2. local()
    locals() 会将所在的名称空间中所有的名字全部传递给HTML页面
    当需要传的变量特别多的时候 使用 locals() 传值
    """
    user_dict = {'username': 'Minho', 'age': 25}
    # render 传值 1.
    # return render(request, 'myfirst.html', {'data': user_dict, 'date': '2021-02-09'})
    
    # render 传值 2.
    return render(request, 'myfirst.html', locals())
```

### redirect

**重定向**

```python
from django.shortcuts import redirect

# Create your views here.
def index(request):
    # return redirect('https://www.baidu.com/')  # 重定向到外部网址
    return redirect('/home/')  # 302重定向到 自己的网站 /home/

def home(request):
    return HttpResponse('home')
```

## 登录功能实现

```python
# 登录功能
"""
html文件：默认都放在templates文件夹下
静态文件：将网站使用的静态文件默认都放在static文件夹下

静态文件：前端已经写好 能够直接调用使用的文件
  - 网站写好的js文件
  - 网站写好的css文件
  - 网站用到的图片文件
  - 第三方前端框架
  ...
  总结：拿来就可以直接使用的  
"""

# django默认不会自动创建static文件夹 需要你手动创建
一般情况下 我们会在static文件夹内 还会做进一步的划分处理
=> 解耦合 方便管理
  - static
    - js
    - css
    - img
    - 其他第三方文件
   
"""
在浏览器中 输入url能够看到对应的资源
是因为后端提前开设了该资源的接口 -- 路由分发
如果访问不到资源 说明后端没有开设该资源的接口
"""

# 访问 login页面 没有加载bootstrap样式的问题
访问 http://127.0.0.1:8000/static/bootstrap-3.3.7-dist/css/bootstrap.min.css 报错 说明后端没有开设该资源的请求路由出来

# 静态文件配置 -- 重点
按照下面方法在settings.py配置文件配置static相关静态文件配置之后 即可访问引用的本地static文件夹下面的静态资源 -- 类似于django自己帮我们开设了对应的路由接口
```

**login.html示例**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Login In</title>
    <link rel="stylesheet" href="/static/bootstrap-3.3.7-dist/css/bootstrap.min.css">
    <script src="/static/bootstrap-3.3.7-dist/js/bootstrap.min.js"></script>
    <script src="/static/js/jquery-3.5.1.min.js"></script>

</head>
<body>
<h1 class="text-center">登录</h1>
</body>
</html>
```

### 静态文件配置

```python
# 类似于访问静态文件的令牌
# 如果你想要访问静态文件 你就必须以static开头
# 切记：该配置 是配置访问url的开头的值 令牌 并不是静态文件存放的目录
STATIC_URL = '/static/'  

"""
/static/  -  令牌
去下面STATICFILES_DIRS列表里面的目录里面 按顺序查找引用的文件
  eg. bootstrap-3.3.7-dist/css/bootstrap.min.css
  都没有找到 则报错
"""

# 静态文件配置 列表 可以有多个(令牌持有者可以访问的文件路径)
# 静态文件真正存放的目录 可以有多个目录 按顺序解析
# 前面的找到请求的文件之后 不再向后查找
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static'),
    os.path.join(BASE_DIR, 'static1'),
]


# 浏览器 设置页面不缓存的方法

"""
当你在写django项目的时候 可能会出现后端代码修改了但是前端页面没有变化的情况
  1. 在同一个端口开了好几个django项目 一直在跑的是第一个django项目
  2. 浏览器缓存的问题 按照下面方法解决
  右键 --> 检查 --> 设置图标(settings) --> Preferences --> Network --> 勾选上 Disable cache(while Devtools is open)
"""
```

**静态文件动态解析**

```html
<!-- 静态文件动态解析
使用模板语法 会自动解析setting.py里面配置的 STATIC_URL 然后自动拼接url
e.g: 
STATIC_URL = '/static/'
url -> /static/bootstrap-3.3.7-dist/js/bootstrap.min.js

or

STATIC_URL= '/xxx/'
url -> /xxx/bootstrap-3.3.7-dist/js/bootstrap.min.js
-->

{% load static %}
<link rel="stylesheet" href="{% static 'bootstrap-3.3.7-dist/css/bootstrap.min.css' %}">
<script src="{% static 'bootstrap-3.3.7-dist/js/bootstrap.min.js' %}"></script>
```

**form表单**

```python
# form表单 默认是GET方法请求数据
# ?后面的内容为参数 不参与(不影响)路径匹配
http://127.0.0.1:8000/login/?username=minho&password=123131313
   
"""
form表单action参数
  1. 不写 默认向当前所在的url提交数据
  2. 全写 指名道姓
  3. 只写后缀 /login/
"""

"""
form表单提交方法 改为post提交数据 提交会报错
  Forbidden (403)
  CSRF verification failed. Request aborted.

解决：
  在前期 我们使用django提交post请求的时候 需要去配置文件中注释掉一行代码
  注释掉 中间件配置中的 CsrfViewMiddleware
"""

# settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    # 'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# views.login处理函数
def login(request):
    # 返回一个登录界面
    """
    GET请求和POST请求 应该有不同的处理机制 -> 如何处理?
    引入下面 request对象方法初始 探索django封装的request对象
    :param request:
    :return:
    """
    print('enter login...')
    return render(request, 'login.html')
```

### request对象方法初识

- **request.method**

```python
def login(request):
    # 返回一个登录界面
    """
    GET请求和POST请求 应该有不同的处理机制 -> 如何处理?
    :param request: 请求相关的数据对象 里面有很多简易的方法
    :return:
    """
    # 返回请求方式 并且是全大写的字符串 <class 'str'>
    # print(request.method)  
    
    """
    这种方式 2层逻辑 推荐下一种
    if request.method == 'GET':
        print('Method GET')
        return render(request, 'login.html')
    elif request.method == 'POST':
        return HttpResponse("收到了 POST请求")
    """
    if request.method == 'POST':
        return HttpResponse('收到POST')
    return render(request, 'login.html')
```

- **request.POST**

```python
# 获取用户post请求提交的普通数据(Form) 不包含文件
request.POST
  - request.PSOT.get()      # 只获取列表最后一个元素
  - request.POST.getlist()  # 直接将整个列表取出
```

- **request.GET**

```python
# 获取用户get请求提交的数据
# get请求携带的数据是有大小限制的 大概只有4KB左右
# 而post请求则没有限制
request.GET
  # 获取到的数据类型 方法的使用 和 request.POST 一模一样
  - request.GET.get()      # 只获取列表最后一个元素
  - request.GET.getlist()  # 直接将整个列表取出
    
def login(request):
    # 返回一个登录界面
    if request.method == 'POST':
        username = request.POST.get('username')
        return HttpResponse('收到POST')
    # 注意get 和 getlist方法区别
    hobby = request.GET.getlist('hobby')
    return render(request, 'login.html')
```

## ORM

### Django本质

```bash
# Django本质
Django本身不是Server(不是 nginx or httpd) 只是个WSGI App 就是一个大函数

# 请求过程
浏览器发起一个HTTP Request报文 到TCPServer(HTTPServer)
Server默认监听80 只不过这个Server正好它能提供http请求报文的解析和Http响应报文的封装 就是说Server是HTTP_Server
得有一个调用者 调用Django得这个函数 调用者就是WSGI_Server

# Nginx 把http请求报文 
==> uwsgi软件/gunicorn软件(软件是WSGI Server) 
==> 解析报文并调用Django App大函数 
==> 函数返回值

# request
--> uwsgi
--> wsgi.py
--> application 注入两个参数调用
--> application(environ, start_response)
--> 返回正文给wsgi封装为response报文
--> 前端显示

# 参考 Django中的实现
uwsgi等软件 调用的就是django/flask的wsgi.py 得到 application 
```

- **入口**

```python
# 入口
根据wsgi的原理 我们知道Django就是APP 入口是项目的wsgi.py文件
1. 请求request来了后 
2. 交给application(WSGIHandler实例) 
3. application(environ, start_response)调用 
4. 调用start_response后返回get_response返回的响应的结果
```

- **Django中的实现**

```python
# wsgi.py
import os
from django.core.wsgi import get_wsgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'cmdb.settings')

# application 就是那个app大函数
application = get_wsgi_application()
```

### ORM快速测试

#### 测试脚本

**根据上面原理得出下面这个脚本**

```python
# django 项目统计目录创建 xx.py

import os
import django

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'cmdb.settings')
# 加载django配置
# 在settings.py 配置好数据库连接即可
django.setup(set_prefix=False)

# 我就是那个app大函数
# 返回一个handler类的实例处理每一个请求
# application = get_wsgi_application()
```

#### 配置`settings.py`

**详细配置参考官网**

```python
# MySQL数据库配置
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'name',  # 连接的数据库名称
        'USER': 'user',  # 连接用户
        'PASSWORD': 'password',  # 连接密码
        'HOST': 'host',  # 主机IP
        'PORT': '3306',  # 端口
        'CHARSET': 'utf8'
    }
}

# Django ORM难学 通过日志看对应SQL
# res.query 或者直接调用返回对象的 query方法 可以直接打印SQL语句
# 具体logging模块详细信息 参考Python标准库文档
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
        },
    },
    'loggers': {
        # 本地 django 2.1.4及之前的版本 该配置无效 无法看到对象的SQL语句 显示为None
        'django.db.backends': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
    },
}
```

**mysqlclient or pymysql**

```python
# 如果要使用pymysql 在项目包文件__init__.py中加入下面代码
import pymysql
pymysql.install_as_MySQLdb()

mysqlcilent  # c语言编写 速度快 推荐
pymsyql      # python原生 兼容度高

# 备注：使用pymysql 去项目配置文件所在目录的__init__.py文件加入以下内容
import pymysql
pymysql.install_as_MySQLdb() 
```

**设置sql_mode**

```python
# 设置sql_mode
# https://docs.djangoproject.com/zh-hans/3.1/ref/databases/

'OPTIONS': {
    "init_command": "SET sql_mode='STRICT_TRANS_TABLES'",
}
```

#### 依赖

```bash
pip install mysqlclient
```

### ORM初识

#### 简介

**ORM 对象关系映射 对象和关系之间的映射 使用面向对象的方式来操作数据库**

```python
# 关系模型和Python对象之间的映射
table  =>  class     # 表 映射为 类
row    =>  object    # 行 映射为 实例
column =>  property  # 字段 映射为 属性(类属性)

# 不足
封装程度太高 有时候sql语句的效率偏低 需要你自己写SQL语句
```

**实例**

```python
# 描述
表名：student
字段-类型：
 - id-int  
 - name-varchar 
 - age-int
    
# 到Python映射
class Student:
    id = ?某字段类型
    name = ?某字段类型
    age = ?某字段类型
   
# 最终得到的实例
class Student:
    def __init__(self):
        self.id = ?
        self.name = ?
        self.age = ?
```

#### 创建model类

```python
# 去应用下面的 models.py 文件
from django.db import models
# Create your models here.
# model类 编写
class User(models.Model):
    id = models.AutoField(primary_key=True)  # id int primary_key auto_increment
    username = models.CharField(max_length=32)  # username varchar(32)
    password = models.IntegerField()  # password int

class Author(models.Model):
    """
    django orm当你不定义主键字段的时候 orm会自动帮你创建一个名为id的主键字段
    后续我们在创建模型表的时候 如果主键字段名没有额外的叫法 那么主键字段可以省略不写 orm帮我们补
    """
    username = models.CharField(max_length=32)
    password = models.IntegerField()
```

#### 数据库迁移

```bash
# 数据库迁移命令
# 只要你修改了models.py中跟 **数据库** 相关的代码 就必须重新执行下面两条命令
$ python manage.py makemigrations    # 将操作记录基础出来(migrations文件夹)
$ python manage.py migrate  # 将操作真正的同步到数据库中
```

#### 字段类型

[DjangoORM字段类型参考](https://docs.djangoproject.com/zh-hans/3.1/ref/models/fields/)

```python
CharField     # 必须要指定 max_length参数 不指定会直接报错
verbose_name  # 该参数是所有字段都有的 用来对字段的解释
...
```

### 字段的增删改查

#### 字段的增加

```python
# 给表增加字段的时候 表里面已经有数据的情况
# **增加外键字段的时候 给的默认值 需要先在对应表中存在 否则迁移报错**
1. 可以在终端内直接给出默认值 django命令行会在终端给出选择
    $ You are trying to add a non-nullable field 'age' to user without a default; we can't do that (the database needs something to populate exi
    sting rows).
    $ Please select a fix:
     1) Provide a one-off default now (will be set on all existing rows with a null value for this column)
     2) Quit, and let me add a default in models.py

2. 设置该字段可以为空(根据实际情况考虑)
    info = models.CharField(max_length=32, verbose_name='个人简介', null=True)
    
3. 直接给字段设置默认值
    hobby = models.CharField(max_length=32, verbose_name='爱好', null=False, default='study')
```

#### 字段的修改

```bash
# models.py 修改后 直接执行迁移命令即可：
$ python manage.py makemigrations
$ python manage.py migrate
```

#### 字段的删除

```python
# 直接注释对应的字段 然后直接执行迁移命令即可
$ python manage.py makemigrations
$ python manage.py migrate

# 比较严重的问题
迁移命令执行完毕之后 数据库对应的数据也会被删除

# **注意**
"""
在操作models.py的时候 一定要细心
  - 千万不要随意注释一些字段
  - 执行迁移命令之前 最好先检查下自己写的代码
"""

# 个人建议： 当你离开你的计算机之后 一定要锁屏
```

### 数据的增删改查

[QuerySet API 参考](https://docs.djangoproject.com/zh-hans/3.1/ref/models/querysets/)

#### 查

[QuerySet.filter](https://docs.djangoproject.com/zh-hans/3.1/ref/models/querysets/#filter)

```python
# 查
models.User.objects.filter(username=username)

"""
返回值可以先看成是列表套数据对象的格式
它也支持索引取值 切片操作 但是不支持负索引
它也不推荐你使用索引的方式取值

models.User.objects.filter(username=username)[0]
models.User.objects.filter(username=username).first()
"""

# 简单登录功能实现 使用 ORM 查询
def login(request):
    if request.method == 'POST':
        # 获取用户提交的用户名和密码 利用orm 校验数据
        username = request.POST.get('username')
        password = request.POST.get('password')
        # 去数据库查询数据
        from app01 import models
        user_obj = models.User.objects.filter(username=username).first()
        if user_obj:
            # 比对密码是否一致
            if password == user_obj.password:
                return HttpResponse('Login Success')
            return HttpResponse('Login Failed, Password Error')
        return HttpResponse('用户不存在')
    return render(request, 'login.html')
```

#### 增

[QuerySet.create](https://docs.djangoproject.com/zh-hans/3.1/ref/models/querysets/#create)

```python
# create 返回当前被创建的对象本身
res = models.User.objects.create(username=username, password=password)

# 实例化对象 调用save方法
user_obj = models.User(username=username, password=password)
user_obj.save()  # 保存数据
```

#### 改

[QuerySet.update](https://docs.djangoproject.com/zh-hans/3.1/ref/models/querysets/#django.db.models.query.QuerySet.update)

```python
# 先将数据库中的数据全部展示到前端 然后给每一个数据2个按钮 一个编辑一个删除
ef userlist(request):
    # 查询用户表里面 所有的数据
    # 方式1 不推荐
    # data = models.User.objects.filter()

    # 方式2
    user_queryset = models.User.objects.all()
    return render(request, 'userlist.html', {'user_info': user_queryset})

# 编辑功能
  1. 点击编辑按钮朝后端发送编辑数据的请求
  """
  Q: 如何告诉后端 用户想要编辑哪条数据?
  A: 将编辑按钮所在的哪一行数据的 主键值 发送给后端 -- 唯一确定值
  
  Q: 如何发送主键值?
  A: <a> href="/edit_user/?user_id={{ user.id }}" </a>
     利用url问号后面携带参数的方式 传主键值给后端获取
  """

  2. 后端查询出用户想要编辑的数据对象 展示到前端页面供用户查看和编辑
    
# view 代码实现
def userlist(request):
    # 查询用户表里面 所有的数据
    # 方式1 不推荐
    # data = models.User.objects.filter()
    
    # 方式2
    user_queryset = models.User.objects.all()
    return render(request, 'userlist.html', {'user_info': user_queryset})

def edit_user(request):
    # 获取url问号后面携带的参数 href="/edit_user/?user_id={{ user.id }}"
    user_id = request.GET.get('user_id')
    # 查询当前用户想要编辑的数据对象
    user_obj = models.User.objects.filter(id=user_id).first()

    if request.method == 'POST':
        username = request.POST.get('username')
        password = request.POST.get('password')
        # 如果是post提交 去数据库修改对象的数据内容
        # 修改方式1
        """
        将filter查询出来的列表中所有的对象全部更新 批量更新操作
        只会修改被修改的字段
        """
        # models.User.objects.filter(id=user_id).update(username=username, password=password)

        # 修改方式2
        """
        该方式 当字段非常多的时候 效率会特别的低
        从头到尾将数据的所有字段全部更新一遍 无论该字段是否被修改
        """
        user_obj.username = username
        user_obj.password = password
        user_obj.save()  # 内部自动识别 不是保存 而且更新

        # 跳转到数据的展示页面
        return redirect('/userlist/')

    # 将数据对象展示到页面上
    return render(request, 'edit_user.html', {'user_info': user_obj})
```

#### 删

[QeurySet.delete](https://docs.djangoproject.com/zh-hans/3.1/ref/models/querysets/#django.db.models.query.QuerySet.delete)

```python
# 删除功能
"""
跟编辑功能逻辑类似
"""

def del_user(request):
    # 获取用户想要删除的数据id值
    user_id = request.GET.get('user_id')
    # 直接去数据库中找到对应的数据 删除即可
    """delete 批量删除"""
    models.User.objects.filter(id=user_id).delete()
    # 跳转到展示页面
    return redirect('/userlist/')


# 数据删除 最好不要真正的删除 添加 is_delete字段
"""
真正的删除功能 需要二次确认
删除数据内部其实并不是真正的删除 我们会被数据添加一个标识字段 用来表示当前数据是否被删除 
如果数据被删除 仅仅是将字段修改一下状态
展示的时候 筛选一下该字段即可

username    password    is_delete
minho        123         0
karubin      123         1

数据值钱 删除危险且浪费 一般情况下 切记不要真正的去删除数据
"""

```

### orm创建表关系

```python
"""
表与表之间的关系
  - 一对多
  
  - 多对多
  
  - 一对一

判断表关系的方法：换位思考
"""
e.g.
图书表 book
id    title    price
1     django   213.12
2     flask    123.33
3     c++      89.08

出版社表 publish
id      name     address
1     上海出版社   上海
2     北京出版社   北京

作者表 author
id    name    age
1     minho   25
2     kimi    37

book2author
id  book_id  author_id
1    1            1
2    1            2
3    2            2
4    3            1

"""
图书和出版社 一对多的关系 外键字段建在多的那一方-book
图书和作者   多对多关系  创建第三张表来专门存储-book2author
一对一 应用场景：
  - 拆表 (作者表 与 作者详情表是一对一)
    - 作者表
    - 作者详情表
"""
```

[一对多ForeignKey](https://docs.djangoproject.com/zh-hans/3.1/ref/models/fields/#django.db.models.ForeignKey)

[多对多ManyToManyField](https://docs.djangoproject.com/zh-hans/3.1/ref/models/fields/#manytomanyfield)

[一对一OneToOneField](https://docs.djangoproject.com/zh-hans/3.1/ref/models/fields/#onetoonefield)

```python
from django.db import models

# Create your models here.
# 创建表关系 先将基表创建出来 然后再添加外键字段
class Book(models.Model):
    title = models.CharField(max_length=32)
    price = models.DecimalField(max_digits=8, decimal_places=2)  # 小数总共8位 小数点后面占2位
    """
    图书和出版社是一对多 并且书是多的一方 所以外键字段放在book表里面
    如果字段对应的是ForeignKey 那么ORM会自动在字段后面加_id -> publish_id
    后面定义ForeignKey的时候 不要自己加_id
    """
    publish = models.ForeignKey(to='Publish')  # 默认就是与出版社表的主键字段做外键关联 可以通过to_field设置
    """
    图书和作者是多对多关系 外键字段建在任意一方均可 但是推荐创建再查询频率较高的一方 (查询方便)
    authors是一个虚拟字段 主要是用来高速ORM 书籍表 和 作者表 是多对多关系 让ORM自动帮你创建第三张表关系
    
    django 1.x 自动级联更新删除 2.x 3.x需要加参数
    """
    authors = models.ManyToManyField(to='Author')

class Publish(models.Model):
    name = models.CharField(max_length=32)
    address = models.CharField(max_length=64)

class Author(models.Model):
    name = models.CharField(max_length=32)
    age = models.IntegerField()
    """
    作者与作者详情是一对一关系 外键字段建在任意一方都可以 但是推荐创建在查询频率较高的表中
    OneToOneField 也会自动给字段加_id后缀
    """
    author_detail = models.OneToOneField(to='AuthorDetail')

class AuthorDetail(models.Model):
    phone = models.BigIntegerField()  # 手机号 或者直接用字符类型
    address = models.CharField(max_length=32)
    
"""
ORM中如何定义三种关系
  一对多：publish = models.ForeignKey(to='Publish')
  多对多：authors = models.ManyToManyField(to='Author')
  一对一：author_detail = models.OneToOneField(to='AuthorDetail')
  
  ForeignKey 和 OneToOneField关系字段 会自动在字段后面加_id后缀
  ManyToManyField关系字段 会自动创建第三张关系对应表 无需我们手动创建
"""

# 注意
"""
1. 在django1.x版本中 外键默认都是级联更新删除的
2. 多对多的表关系可以有好几种(三种)创建方式 这里只是其中一种
3. 针对外键字段里面的其他参数 暂时不做考虑 详情可以参考上面官网链接
"""
```

#### 多对多的三种创建关系

**需要掌握 全自动 和 半自动(扩展性很高 一般情况下都采用半自动)**

- **全自动**

```python
# 利用ORM 自动帮我们创建第三张关系表
class Book(models.Model):
    name = models.CharField(max_length=32)
    authors = models.ManyToManyField(to='Author')
    
class Author(models.Model):
    name = models.CharField(max_length=32)
    
"""
优点： 
  - 第三张关系表的model代码 不需要自己写
  - 支持ORM提供操作第三张关系表的方法(add clear remove 正反向查询...)
  
缺点：
  -  第三张关系表的拓展性极差 没有办法额外添加字段
"""
 
```

- **纯手动**

**不建议使用该方式**

```python
# 自己书写 第三张关系表
class Book(models.Model):
    name = models.CharField(max_length=32)
    
class Author(models.Model):
    name = models.CharField(max_length=32)
    
class Book2Author(models.Model):
    book = models.ForeignKey(to='Book')
    author = models.ForeignKey(to='Author')
    
"""
优点：
  - 第三张表完全取决于你自己 进行额外扩展

缺点：
  - 需要自己写第三张关系表的model代码
  - 不能够再使用ORM提供的简单方法
"""
```

- **半自动**

```python
# 还是手动创建第三张关系表
# 然后通过字段参数 解决全手动创建带来的缺点 无法使用ORM提供的方法

class Book(models.Model):
    name = models.CharField(max_length=32)
    # 利用额外参数 告诉ORM 第三张关系表我已经手动创建 你参考就行 不必再自动创建
    # 多对多外键字段 可以写在任意一方 这里写在哪一边 会影响到through_fields参数的字段顺序
    # 写在book表 through_fields=('book', 'author')
    author = models.ManyToManyField(to='Author', 
                                    through='Book2Author',
                                    through_fields=('book', 'author')
                                   )
    
class Author(models.Model):
    name = models.CharField(max_length=32)
    # OR 写在author表 through_fields=('author', 'book')
    # author = models.ManyToManyField(to='Author', 
    #                                 through='Book2Author',
    #                                 through_fields=('author', 'book')
    #                                )
    
class Book2Author(models.Model):
    book = models.ForeignKey(to='Book')
    author = models.ForeignKey(to='Author')
    
"""
through_fields=('xx', 'oo')

through_fields字段先后顺序：
  本质：由第三张关系表 查询外键所在表 通过(第三张表)哪个字段 就把该字段放在前面一个参数
  总结：简化判断 -> 当前表(外键字段所在的表)是谁 就把对应的关联字段放前面
"""

"""
半自动不足：
  可以使用orm的正反向查询 但是没法使用 add set remove clear这4个方法
"""
```

### Model元编程

**Django Model 背后的故事 -- 元编程**

#### 测试环境

```bash
cd ../project_dir

# 安装django
pip install django

# 创建项目
django-admin startproject cmdb

# 配置修改参考上文

# 创建user包
python manage.py startapp user

# 注册 
一定要在settings.py INSTALLED_APPS 注册user 否则不能迁移
```

#### Model数据库模型

```python
# user/models.py

from django.db import models

# Create your models here.
class User(models.Model):  # 继承的目录 代码复用
    class Meta:  # 元数据
        db_table = 'user'  # 指定mysql中的表名 否则默认为module_class.__name__
    # 字段定义
    id = models.AutoField(primary_key=True)  # 自增 同时是主键 Django只支持单一主键
    # 元编程会在没有主键的请况下 建立一个字段名为:id 的自增主键
    username = models.CharField(max_length=20, null=False)
    password = models.CharField(max_length=128, null=False)  # passwd 密文 hash

    def __repr__(self):
        return "<User {}， {}>".format(self.id, self.username)  # self.id == self.pk 主键

    __str__ = __repr__

# 迁移：Model类生成数据库中的表和字段
```

#### 元类

- **类定义**

```python
class A:  # 语法定义
    pass

# type函数问什么 问你是谁的实例

# type说明什么？ A是type的实例 A称为类对象 由type构造出来的实例
print(1, type(A))
# class构造出来的是类 type是构造类的类
# 元类：  用来构造类的类型
# 元编程: 用编程的方法来编程
# 元数据: 用来描述数据的数据
print(type(type))  # type
# 元类 用来构造类对象 不是用来继承的
# class Model(metaclass=ModelBase) 使用metaclass参数改变元类

# __new__ 两个作用
# 1. 用来实例化
# 2. 用在元编程中

print('=' * 30)
a = A()  # a是A类的实例
print(2, type(a))  # A, a是由A构造出来的实例
```

- **django model 仿写**

```python
class Field:
    def __init__(self, primary_key=False, name=None, null=False):
        self.pk = primary_key
        self.name = name  # ？
        self.null = null

    def __str__(self):
        return "<Field {}>".format(self.name)

    __repr__ = __str__

class Manager:
    pass

# type是Python提供的原始元类
# 不知道怎么构建元类 继承自type解决(因为type是元类)
# 对比：普通类最终继承自object
class ModelBase(type):
    # 元类编程中：构建类对象的
    def __new__(cls, name, bases, attrs):
        print('=' * 30)
        print(cls)
        # name, bases, attrs = args
        # 三元组 (name: 构造类的name, bases:构造出来类的 继承类, attrs:构造出来类的类属性)
        print(name, bases, attrs)
        # attrs['id'] = 'abc'
        print(attrs)
        pks = []  # 主键们
        for k, v in attrs.items():
            # k, v => 'username', Field对象
            if isinstance(v, Field):
                if v.pk:
                    pks.append(v)
                if not v.name:
                    v.name = k
        attrs['pks'] = pks

        # 判断主键不存在 贼默认创建id主键
        if not pks:
            id = Field(primary_key=True, name='id', null=False)
            pks.append(id)
        attrs['pks'] = pks
        print('=' * 30)
        # return None  # 应该返回类对象 而不是None or 其他
        # return super().__new__(cls, name, bases, attrs)
        new_class = super().__new__(cls, name, bases, attrs)
        # 省略1000行
        # 判断objects 挂objects对象
        setattr(new_class, 'objects', Manager())
        return new_class

# 改模子 由type改为使用ModelBase为元类
# 用ModelBase的__new__方法构造Model类
# 本来找type的 指定metaclass改变
# class Model(list, metaclass=ModelBase):
class Model(metaclass=ModelBase):
    pass

# m = Model()  # None() => TypeError
# print(*Model.__dict__.items(), sep='\n')

# User继承自Model Model类由ModelBase构造 那么User也是由ModelBase构造
# 不用显示写：metaclass=ModelBase
class User(Model):  # 由元类构造
    username = Field(null=False)
    password = Field(null=False)

print(User.pks)

# python manage.py runserver  # 测试用的wsgi server
```

## 路由层

### 路由匹配

```python
# 路由匹配
url(r'test', views.test),       
url(r'testadd', views.testadd),
"""
http://127.0.0.1:8000/testadd  永远访问不到 永远访问的是views.test视图函数

url方法的第一个参数是正则表达式
只要第一个参数正则表达式能够匹配到内容 就会立刻停止往下匹配 直接执行对应的视图函数
"""

# 加斜杠/ django匹配的时候会自动加斜杠去匹配
"""
http://127.0.0.1:8000/testadd  第一次匹配 匹配不到 - 301
django会自动在末尾加一个斜杠 进行第二次匹配 - 200 OK

django内部帮你做的重定向
  - 一次匹配不行
  - url末尾加斜杠 再来一次
"""
url(r'test/', views.test),       
url(r'testadd/', views.testadd),
  
# 自动加斜杠功能可以取消 配置settings.py
APPEND_SLASH = False  # 取消自动加斜杠 默认为True 不建议修改

# QA
Q: GET /msafbsvfbtest/ 可以匹配到 test/
A: url(r'^test/', v2.test)   # ^ 解决   
    
Q: GET /test/sdanakvdbkaadad/daskndakjd 可以匹配到 test/
A: url(r'^test/$', v2.test)  # ^test$ 正则精确 test
    
Q：如何写首页URL
A: url(r'^$', v2.home)  # 首页匹配 不能写成url(r'', v2.home)
    
Q: 尾页(404页面 所有页面都没找到) 需要放在最后
A: url(r'', v2.error)  # 了解即可 不这样写
```

### URL分组

```python
"""
分组：就是给某一段正则表达式用小括号括起来
"""
url(r'^test/\d+$', v2.test)  # 正常访问
url(r'^test/(\d+$)', v2.test)  # 分组后 报错TypeError test() takes 1 positional argument but 2 were given
```

#### 无名分组

```python
url(r'^test/(\d+$)', v2.test)  
# 分组后 报错TypeError 缺少位置参数
# test() takes 1 positional argument but 2 were given

# 修改视图函数接收分组传递的参数
def test(request, args):
    print(args)
    return HttpResponse('test')

"""
无名分组 就是将括号内正则表达式匹配到的内容当作 位置参数 传递给后面的视图函数
"""
```

#### 有名分组

```python
"""
(?P<name>) 可以给正则表达式 分组并起一个别名
有名分组 就是将括号内正则表达式匹配到的内容当作 关键字参数 传递给后面的视图函数
"""
url(r'^testadd/(?P<year>\d+)', v2.testadd) 
# 起别名后 报错 TypeError 缺少关键字参数
# testadd() got an unexpected keyword argument 'year'

# 增加关键字对应的形参
def testadd(request, years):
    print(years)
    return HttpResponse('testadd')
```

#### 不能混用

```python
url(r'^index/(\d+)/(?P<year>\d+)/', v2.index)

def index(request, args, year):
    print(args, year)
    return HttpResponse("index")

>>> TypeError at /index/12313/32131313/
>>> index() missing 1 required positional argument: 'args'
    
# 总结： 不能混用 但是 同一个分组可以使用N多次
url(r'^index/(\d+)/(\d+)/(\d+)/', views.index),
url(r'^index/(?P<year>\d+)/(?P<month>\d+)/(?P<day>\d+)/', views.index),

def index(request, *args, **kwargs):
    print(args)    # ('12', '3123', '42341')
    print(kwargs)  # {'year': '12', 'month': '3123', 'day': '42341'}
    return HttpResponse("<h1>index</h1>")
```

#### 反向解析

```python
"""
本质：通过一些方法得到一个结果 该结果可以访问到对应的url 从而触发对应视图函数的运行

步骤：
  1. 先给路由与视图函数起一个别名
  url(r'^func_minho/', views.func, name='alias_func'),
  
  2. 反向解析
    2.1 后端反向解析
    from django.shortcuts import reverse, HttpResponseRedirect
    url = reverse('alias_func')
    return HttpResponseRedirect(url)
        
    2.2 前端反向解析 (模板语法)
    <a href="{% url 'alias_func' %}">111</a>
"""
```

- **最简单的请况 url第一个参数里面没用正则符号**

```python
# url
url(r'^home/', views.home, name='home_page'),

"""
给url函数添加一个 name参数(**唯一**) 起一个名字 类似于别名 
注意: **别名唯一 别名不能冲突**
我可以访问到这个名字(url正则可以随意修改) 然后获取到对应的url 从而触发视图函数的运行

1. 前端
  {% url 'alias_name' %}    
  alias_name对应url函数里面的 name参数对应的字符串

2. 后端
   reverse()
"""
```

- **无名分组反向解析**

```python
# url
url(r'^index/(\d+)/', views.index, name='alias_index'),

# 前端
<a href="{% url 'alias_index' 123 %}">111</a

# 后端
reverse('alias_index', args=(1,))  # /index/1/

"""
这个数字写代码的时候应该放什么?
  - 数字 一般情况下放的是数据的主键值 来做数据的编辑和删除
  # urls.py
  url(r'^edit/(\d+)/', views.edit, name='xxx')
  
  # views.py
  def edit(request, edit_id):
      reverse('xxx', args=(edit_id,))
  
  # html
  {% for user in user_queryset %}
  <a href='{% url 'xxx' user.id %}'>编辑</a>
  {% endfor %}
  
"""
```

- **有名分组反向解析**

```python
# url
url(r'^func/(?P<year>\d+)', views.func, name='alias_func')

# 前端
<a href="{% url 'alias_func' year=2009 %}">111</a>
<a href="{% url 'alias_func' 2021 %}">222</a>

# 后端
1. 有名分组写法一
reverse('alias_func', kwargs={'year': 2010})

2. 有名分组 简便写法
reverse('alias_func', args=(2021,))
```

### 路由分发

```python
"""
特点：django 每一个应用 都可以有自己的templates文件夹 urls.py static文件夹
  正是基于上述特点 django能够非常好的做到分组开发(每个人只写自己的app)

  多人开发完之后 只需要将每个人书写的app全部拷贝到一个新的django项目中 然后在配置文件里面注册所有的app 再利用路由分发的特点将所有的app整合起来
  
  当一个django项目中的url特别多的时候 总路由urls.py代码非常冗余 不好维护
  解决: 利用路由分发来减轻总路由的压力
"""

# 路由分发
"""
利用路由分发之后 总路由不在处理 路由与视图函数的直接对应关系 而是做一个分发处理
处理：识别当前url是属于哪个应用下的 然后直接分发给对应的应用处理
"""
```

**路由分发实现**

```python
from django.conf.urls import url, include
from app01 import urls as app01_urls
from app02 import urls as app02_urls

# 总路由
"""
只要url是app01开头就会自动将url中app01后面的路径交给 app01下的urls.py去做匹配
"""
urlpatterns = [
    # 路由分发
    # 只要url前缀是app01开头的 全部交给app01处理
    url(r'^app01/', include(app01_urls)), 
    # 只要url前缀是app02开头的 全部交给app02处理
    url(r'^app02/', include(app02_urls)),  
]

# 子路由 app01.urls
from django.conf.urls import url
from app01 import views
urlpatterns = [
    url(r'^reg/', views.reg)
]

# 子路由 app02.urls
from django.conf.urls import url
from app02 import views
urlpatterns = [
    url(r'^reg/', views.reg)
]


# 总路由 推荐写法 简便
url(r'^app01/', include('app01.urls')),
url(r'^app02/', include('app02.urls')),
# 注意事项：总路由的url 千万不能加$结尾 加了之后 无法继续向下匹配
```

### 名称空间

**解决url别名 命名冲突的问题 了解即可 可以不用 保证别名不同即可**

```python
# 当多个应用出现了相同的别名 反向解析是否会自动识别对应的前缀

"""
正常请况下的反向解析 是没有办法自动识别前缀的
"""

# 解决：利用名称空间

# 总路由加名称空间：namespace
urlpatterns = [
    url(r'^app01/', include('app01.urls', namespace='app01')),
    url(r'^app02/', include('app02.urls', namespace='app02')),
]

# 子路由 别名一致
"""
# app01.urls
urlpatterns = [url(r'^reg/', views.reg, name='reg')]

# app02.urls
urlpatterns = [url(r'^reg/', views.reg, name='reg')]
"""

# 解析的时候
  - 前端
    <a href="{% url 'app01:reg' %}">111</a>
    <a href="{% url 'app02:reg' %}">222</a>
    
  - 后端
    reverse('app01:reg')
    reverse('app02:reg')
```

**总结：其实只要保证别名不冲突 就没有必要使用名称空间**

```python
"""
一般情况下 有多个app的时候 我们在起别名的时候 会加上对应app的前缀
这样的话 就能够确保多个app之前名字不冲突的问题
"""
```

### 伪静态

**了解即可**

```python
"""
静态网页
  数据是写死的 万年不变
  
伪静态
  将一个动态网页 伪装为静态网页
  
  为什么要伪装呢?
    e.g.: 博客园
    伪装的目的 
      - 在于增大本网站的seo查询力度 
      - 并且增加搜索引擎收藏本网站的概率
      
  搜索引擎：本质就是一个巨大的爬虫程序
  总结：无论怎么优化 还是搞不过搜索竞价
 

"""
# 实现 把url设置为.html结尾 乍一看 以为是静态(伪静态)
urlpatterns = [
    url(r'^reg.html/', views.reg, name='app01_reg'),
]
```

## 视图层

**视图函数必须返回HttpResponse对象**

```python
# 否则报错
The view app01.views.index didn't return an HttpResponse object. It returned None instead.
```

### 三板斧

```python
# HttpResponse
返回字符串类型
class HttpResponse(HttpResponseBase):
    pass

# render
返回html页面 并且返回之前还可以给html文件传值
def render(request, template_name, context=None, content_type=None, status=None, using=None):
    """
    Returns a HttpResponse whose content is filled with the result of calling
    django.template.loader.render_to_string() with the passed arguments.
    """
    content = loader.render_to_string(template_name, context, request, using=using)
    return HttpResponse(content, content_type, status)

# redirect
重定向 也是继承的HttpResponse
```

### render简单的内部原理

```python
def index(request):
    from django.template import Template, Context
    res = Template('<h1>{{ user }}</h1>')
    con = Context({'user': {'username': 'minho', 'password': 123}})
    ret = res.render(con)
    print(ret)
    return HttpResponse(ret)

>>> <h1>{&#39;username&#39;: &#39;minho&#39;, &#39;password&#39;: 123}</h1>
```

### JsonResponse

```python
"""
Q: json格式的数据有什么用?
A: 前后端数据交互需要使用到json作为过渡 实现跨语言传输数据

前端序列化：
  - JSON.stringify()      json.dumps()
  - JSON.parse()          json.loads()
"""

# 任务：给前端浏览器返回一个json格式的字符串
```

- **原生json模块实现**

```python
# 原生json模块实现
import json
def ab_json(request):
    user_dict = {'username': 'Minho这是中文', 'password': '123123', 'hobby': 'study'}
    # 先转成json格式字符串
    json_str = json.dumps(user_dict, ensure_ascii=False)
    return HttpResponse(json_str)
```

- **JsonResponse实现**

```python
from django.http import JsonResponse
def ab_json(request):
    user_dict = {'username': 'Minho这是中文', 'password': '123123', 'hobby': 'study'}
    lst = [111, 222, 333, 444]
    
    # 读源码 掌握用法 如何传参
    # 序列化 字典
    return JsonResponse(user_dict, json_dumps_params={'ensure_ascii': False})
    
    # 序列化非字典对象 需要加safe参数 通过报错信息得知
    # In order to allow non-dict objects to be serialized set the safe parameter to False
    return JsonResponse(lst, safe=False)

# JsonResponse 默认只能序列化字典 序列化其他数据(其他可以被序列化的数据 不是所有的数据都能被序列化) 需要加safe参数
```

------

### Django自带的序列化组件

**DRF铺垫**

```python
# 需求：在前端给我获取到后端用户表里面的所有数据 需要是列表套字典

# json格式化校验 
https://www.bejson.com/  
```

- **JsonResponse**

**json.dumps 不再演示**

```python
from app02 import models
def user_ser(request):
    # 返回 [{}, {}, {}, {}]
    user_queryset = models.User.objects.values()
    # data = [user for user in user_queryset]
    data = list(user_queryset)
    return HttpResponse(JsonResponse(data, safe=False))


# OR 自定构造字典结构
from app02 import models
def user_ser(request):
    # 返回 [{}, {}, {}, {}]
    user_queryset = models.User.objects.all()
    data = []
    for user_obj in user_queryset:
        tmp = {
            'pk': user_obj.pk,
            'name': user_obj.name,
            'age': user_obj.age,
            'gender': user_obj.get_gender_display(),
        }
        data.append(tmp)
    return HttpResponse(JsonResponse(data, safe=False))

"""
前后端分离的项目：
  - 作为后端 只需要写代码将数据处理好
  - 序列化返回给前端即可
  - 再写一个接口文档 告诉前端每个字段代表的意思
"""
```

- **serializers**

```python
from app02 import models
from django.core import serializers
def user_ser(request):
    user_queryset = models.User.objects.all()
    """
    会自动帮你将数据变成json格式的字符串 并且内部非常的全面
    写接口 就是利用序列化组件渲染数据 然后写一个接口文档
    """
    res = serializers.serialize('json', user_queryset)
    return HttpResponse(res)

"""
使用序列化组件 封装程度更高 需要你写的代码更少
"""
```

### Form表单上传文件及后端如何操作

```python
"""
form表单上传文件类型的数据
  1. method必须指定成post
  2. enctype必须换成formdata
"""

def ab_file(request):
    if request.method == 'POST':
        # print(request.POST)  # 只能获取普通键值对数据 文件不行
        # print(request.FILES)  # <MultiValueDict: {'file': [<TemporaryUploadedFile: python-3.8.7-amd64.exe (application/x-msdownload)>]}>
        file_obj = request.FILES.get('file')  # 文件对象
        print(file_obj.name)
        with open(file_obj.name, 'wb') as f:
            for line in file_obj.chunks():  # 推荐加上chunks
                f.write(line)
    return render(request, 'form.html')
```

### request对象方法

```python
"""
request.method
request.POST
request.GET
request.FILES
"""

"""
request.path
request.path_info
request.get_full_path()  # 能获取完整的rul及问号后面的参数
"""
URL: http://127.0.0.1:8000/app01/ab_file/?username=minho
            
requesst.path           -> /app01/ab_file/
request.path_info       -> /app01/ab_file/

# 既能拿路由 也能拿参数 上面两个方法不能拿参数
request.get_full_path() -> /app01/ab_file/?username=minho


"""
request.body  # 原生的 浏览器发过来的二进制数据
  - GET      b''
  - POST      
"""
```

### FBV与CBV

**视图函数既可以是函数也可以是类**

#### FBV

**基于函数的视图**

```python
# FBV
def index(request):
    return HttpResponse('index')
```

#### CBV

- **基于类的视图**

```python
# CBV路由 urls.py写法
# as_view() CBV路由写法和FBV有点不一样(其实本质相同)
url(r'^login/', views.MyLogin.as_view()),

# 类视图
from django.views import View

class MyLogin(View):
    """只要是处理业务逻辑的视图函数 形参里面肯定要有request"""
    def get(self, request):
        return render(request, 'form.html')

    def post(self, request):
        return HttpResponse('post方法')
    
"""
FBV和CBV各有千秋

CBV特点：
  能够直接根据请求方式的不同 直接匹配到对应的方法执行
 
  思考：内部如何实现的?  -- **非常重要** 学习DRF必备的知识点
"""
```

- **CBV源码剖析**

```python
# 千万不要随意修改源码 出BUG很难找
```

**突破口**

```python
# CBV源码剖析 突破口在urls.py
from django.conf.urls import url
from app02 import views

urlpatterns = [
    url(r'^index/', views.index),
    url(r'^login/', views.LoginView.as_view()),
]
"""
函数/方法 加括号执行优先级最高 
as_view() 立即执行
猜测 as_view()可以通过我们自定义的类LoginView类直接调用 可能是下面2种情况
  - 1. @staticmethod 修饰的静态方法
  - 2. @classmethod 修饰的类方法
 
1. as_view()本质
  url(r'^login/', views.LoginView.as_view())
  上述代码在启动django的时候就会立刻执行as_view()方法
  as_view()方法会返回一个函数 view
  => url(^'^login/', views.view)  => 变形后 和FBV一样
  CBV与FBV在路由匹配上 本质是一样的 都是路由 对应 函数(函数内存地址)
  
2. 浏览器访问/login/ 触发views.view view具体做了什么? 
"""
```

**as_view()本质**

```python
@classonlymethod
def as_view(cls, **initkwargs):
    def view(request, *args, **kwargs):
        self = cls(**initkwargs)
        # cls是我们自己写的类 调用的时候自动注入
        # self = LoginView(**initkwargs) 产生一个我们自己写的类的实例
        if hasattr(self, 'get') and not hasattr(self, 'head'):
            self.head = self.get
        self.request = request
        self.args = args
        self.kwargs = kwargs
        return self.dispatch(request, *args, **kwargs)
        """
        以后 经常会需要看源码 但是在看Python源码的时候 一定要时刻提醒自己 面向对象属性方法查找顺序 mro
        总结：看源码 只要看到了self.xxx 一定要问自己 当前这个self到底是谁
        """
    return view
```

**dispatch CBV精髓**

```python
# CBV的精髓
def dispatch(self, request, *args, **kwargs):
    # Try to dispatch to the right method; if a method doesn't exist,
    # defer to the error handler. Also defer to the error handler if the
    # request method isn't on the approved list.
    # 获取当前请求的小写格式 然后比对当前请求方式是否合法
    # get请求为例
    if request.method.lower() in self.http_method_names:
       """
       反射：通过字符串来操作对象的属性或方法 运行时获取类型定义信息
       handler = getattr(自己写的类产生的对象, 'get', 当找不到get属性或者方法的时候就会用第三个参数)
       handler = 我们自己写的类里面的get方法
       """
        handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
    else:
        handler = self.http_method_not_allowed
    # 自动调用get方法
    return handler(request, *args, **kwargs)
```

**反射补充**

```python
运行时：runtime 区别于编译时 指的是程序被加载到内存中执行的时候
反射：reflection 执行是运行是获取类型定义信息

"""
一个对象能够在运行时 像照镜子一样 反射出其类型信息
简单说 在Python中 能够通过一个对象 找出其type class attribute或method的能力 成为反射或者自省

Python源码里 使用最频繁的其实就是反射
"""

具有反射能力的函数有:
  - type() 
  - isinstance()
  - callable() 
  - dir()
  - getattr() 等等...
    
反射相关的魔术方法：
__getattr__
__setattr__
__delattr__

__getattribute__  # 特殊 尽量不用
```

| 内建函数                         | 意义                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| getattr(object, name[, default]) | 通过name返回object的属性值 当属性不存在 将使用default返回 如果没有default 则抛出AttributeError name必须为**字符串** |
| setattr(object, name, value)     | object的属性存在则覆盖 不存在则新增                          |
| hasattr(object, name)            | 判断对象是否具有这个名字的属性 name必须为**字符串**          |

### CBV如何添加装饰器

```python
CBV中 django不建议你直接给类的方法加装饰器
无论该装饰器能否正常工作 都不建议增加
```

#### 加在CBV视图的具体方法上

```python
from django.views import View
from django.utils.decorators import method_decorator

class MyLogin(View):
    """
    CBV中 django不建议你直接给类的方法加装饰器
    无论该装饰器能否正常工作 都不建议增加
    """
    # @login_auth  # 不能直接使用装饰器
    @method_decorator(login_auth)  # 方式1: 指名道姓
    def get(self, request):
        return HttpResponse('GET请求')
    @method_decorator(login_auth)
    def post(self, request):
        return HttpResponse('POST请求')
```

#### 加在类视图上

```python
from django.views import View
from django.utils.decorators import method_decorator

# 方式2 可以添加多个 针对不同的方法加不同的装饰器
@method_decorator(login_auth, name='get')  
@method_decorator(login_auth, name='post')
class MyLogin(View):
    """
    CBV中 django不建议你直接给类的方法加装饰器
    无论该装饰器能否正常工作 都不建议增加
    """
    def get(self, request):
        return HttpResponse('GET请求')

    def post(self, request):
        return HttpResponse('POST请求')
```

#### 加在dispath方法上

```python
from django.views import View
from django.utils.decorators import method_decorator

class MyLogin(View):
    """
    CBV中 django不建议你直接给类的方法加装饰器
    无论该装饰器能否正常工作 都不建议增加
    """
    # 看CBV源码可以得出 CBV里面的所有方法在执行之前都需要先经过dispath方法(该方法你可以看成是一个分发方法)
    # 方式3: 重写dispath方法 给dispath方法装 会直接作用于当前类里面所有的方法
    @method_decorator(login_auth)  
    def dispatch(self, request, *args, **kwargs):
        return super().dispatch(request, *args, **kwargs)

    def get(self, request):
        return HttpResponse('GET请求')

    def post(self, request):
        return HttpResponse('POST请求')
```

## 模板层

### 模板语法

#### 传值

**模板语法传值**

```jinja2
{# #}    # 注释
{{ }}    # 变量相关
{% %}    # 逻辑相关
```

**后端传值示例**

```python
def index(request):
    # 模板语法可以传递的后端Python数据类型
    n = 123
    f = 11.11
    b = True
    s = 'name is minho'
    lst = ['kimi', 'huahua', 'minho']
    tup = ('11', '22', 33)
    dic = {'username': 'minho', 'age':20}
    se = {'按级别的', '密码'}
    def func():
        print('我被执行了')
        return 'func return res'
    class MyClass:
        def get_self(self):
            return 'self'
        
        @staticmethod
        def get_static():
            return 'static method'
        
        @classmethod
        def get_cls(cls):
            return 'cls'
        
        # 对象被展示到html页面上 就类似于执行了打印操作也会触发__str__方法
        def __str__(self):
            return 'MyClass obj'
    obj = MyClass()
    return render(request, 'index.html', locals())
```

**前端展示**

```jinja2
<p>整型：{{ n }} </p>
<p>浮点型：{{ f }} </p>
<p>字符串：{{ s }} </p>
<p>布尔值：{{ b }} </p>
<p>列表：{{ lst }}</p>
<p>元组：{{ tup }}</p>
<p>字典：{{ dic }}</p>
<p>集合：{{ se }}</p>
{# 传递函数名会自动加括号调用 但是模板语法不支持给函数传额外的参数 如果有参数 不执行也不报错 #}
<p>函数：{{ func }}</p>
{# 传类名的时候也会自动加括号调用(实例化) #}
<p>类：{{ MyClass }}</p>
<p>类对象(ins)：{{ obj }}</p>
{# Django模板语法 能够自动判断出当前的变量名是否可以加括号调用 如果可以就会自动执行 针对的是函数名和类名 #}
<p>普通方法：{{ obj.get_self }}</p>
<p>静态方法：{{ obj.get_static }}</p>
<p>类方法：{{ obj.get_cls }}</p>
```

#### 取值

```html
django模板语法的取值 是固定的格式 只能用"句点符" .
即可点键 也可以点索引  也可以两者混用

<p>username: {{ dic.username }}</p>
<p>list取值: {{ lst.0 }}</p>
<p>info: {{ dic.hobby.3.info}}</p>
```

#### 过滤器

```python
# 过滤器 就类似于是模板语法内置的 内置方法
# django内置有60多个过滤器 了解10个左右就差不多 后面遇到再去了解
```

- 基本语法

```jinja2
{{ 数据|过滤器:参数 }}
```

- 常用过滤器

```jinja2
<h1>过滤器</h1>
<p>统计长度：{{ s|length }}</p>

{# 第一个参数布尔值为True 就展示第一个参数的值 否则使用default的值 类似 dict.get(d, default) #}
<p>默认值：{{ b|default:'啥也不是' }}</p>

<p>文件大小：{{ file_size|filesizeformat }}</p>
<p>日期格式化：{{ current_time|date:'Y-m-d H:i:s' }}</p>
<p>切片操作(支持步长)：{{ lst|slice:'0:4:3' }}</p>
<p>切取字符(包含3个.)：{{ info|truncatechars:9 }}</p>
<p>切取单词(不包含3个. 只按照空格切)：{{ eng|truncatewords:5 }}</p>
<p>移除特定的字符：{{ eng|cut:'model' }}</p>
<p>拼接操作：{{ lst|join:'$' }}</p>
<p>拼接操作(加法): {{ n|add:'10' }}</p>
<p>拼接操作(加法): {{ s|add:eng }}</p>

{# hhh默认不转义标签 为了安全 不执行恶意scripts代码#}
<p>转义：{{ hhh|safe }}</p>
<p>转义：{{ sss|safe }}</p>
<p>转义：{{ res }}</p>
```

- **转义**

```python
# 模板语法 默认不转义标签 为了安全 不执行恶意scripts代码

# 转义标签用法 safe参数
# 前端
{{ var|safe }}

# 后端
from django.utils.safestring import mark_safe
res = mark_safe('<h1>新新</h1>')

"""
以后你在写全栈项目的时候 前端代码不一定非要在前端页面书写
也可以先在后端写好 然后再传递给前端页面
"""
```

#### 标签

**不要被名字干扰 标签就是一堆逻辑**

```jinja2
{# for循环 #}
{% for l in lst %}
    <p>{{ forloop }}</p>
    <p>{{ item }}</p>    # 一个个元素
{% endfor %}

{# forloop需要掌握它的参数 #}
{
'parentloop': {}, 
'counter0': 0, 
'counter': 1, 
'revcounter': 6, 
'revcounter0': 5, 
'first': True, 
'last': False
}

{# if判断 #}
{% if b %}
    <p>True res</p>
{% elif s %}
    <p>elif res</p>
{% else %}
    <p>False res</p>
{% endif %}

{# for if混合使用 #}
{% for item in lst_empty %}
    {% if forloop.first %}
        <p>第一次循环</p>
    {% elif forloop.last %}
        <p>最后一次循环</p>
    {% else %}
        <p>{{ item }}</p>
    {% endif %}
    {% empty %}
        <p>for循环的可迭代对象 内部没有元素 根本没法循环</p>
{% endfor %}

{# 可以使用字典的方法 #}
{% for item in dic.keys %}
    <p>{{ item }}</p>
{% endfor %}
{% for item in dic.items %}
    <p>{{ item }}</p>
{% endfor %}
{% for value in dic.values %}
    <p>{{ value }}</p>
{% endfor %}
{% for item in lst %}
    <p>{{ item }}</p>
{% endfor %}

{# with起别名 #}
{% with dic.hobby.3.info as nb %}
    {# 在with语法内 就可以通过as后面的别名快速的使用到前面非常复杂获取数据的方式 #}
    <p>{{ nb }}</p>
{% endwith %}
```

### 自定义

**过滤器/标签及inclusion_tag**

- **自定义过滤器**

```python
"""
先三步走：
  1. 在应用下创建一个名字**必须**叫 templatetags文件夹
  2. 在该文件夹内 创建任意名称的py文件 e.g: mytag.py
  3. 在该py文件内 **必须**先书写下面两句固定的代码 (单词一个都不能错)
    form django import template
    register = template.Library()
"""

from django import template
register = template.Library()
# 自定义过滤器(参数最多只有2个)
# 主要使用 name="" 后面的名字 也就是过滤器名字 函数的名字无所谓 随便取
@register.filter(name='myfilter')
def my_sum(x, y):
    return x + y

# html使用
<h1>自定义的使用(过滤器 最多只能有2个参数)</h1>
{% load mytags %}
<p>{{ n|myfilter:666 }}</p>
```

- **自定义标签**

**类似自定义函数**

```python
# 自定义标签(参数可以有多个)
@register.simple_tag(name='plus')
def index(a, b, c, d):
    return "{}-{}-{}-{}".format(a, b, c, d)

# 使用
{% load mytags %}
{# 标签多个参数之间 空格隔开 #}
<p>{% plus 'minho' 123 123 9988 %}</p>
```

- **自定义inclusion_tag**

```python
"""
内部原理：
  - 先定义一个方法
  - 在页面上调用该方法 并且可以传值
  - 该方法会生成一些数据 然后传递给一个html页面
  - 之后 将渲染好的结果放到调用的位置
  
"自动生成某一个页面的局部部分"
"""
# 自定义inclusion_tag
@register.inclusion_tag('left_menu.html')
def left(n):
    data = ['第{}项'.format(i) for i in range(n)]
    # inclusion_tag 两种给作用页面传值的方式
    # 第一种
    # return {'data': data}
    # 第二种
    return locals()  # 将data传递给left_menu.html

# 局部页面内容
<ul>
    {% for item in data %}
        <li>{{ item }}</li>
    {% endfor %}
</ul>

# 使用
"""
总结：当html页面某一个地方的页面需要传参数才能够动态的渲染出来 并且在多个页面上都需要使用到该局部 那么就考虑将该局部页面做成inclusion_tag形式
"""
{% load mytags %}
{% left 5 %}
```

### 模板继承

```python
"""
有一些网站 这些网站页面整体都大差不差 只是某一些局部在做变化
同一个html页面 想重复使用大部分样式 只是局部的修改
"""

# 模板的继承 你自己先选好一个你想要继承的模板页面
{% extends 'home.html' %}

# 继承了之后 子页面跟模板页面长的一摸一样的 你需要在模板页面上提前划定可以被修改的区域
{% block content %}
   模板内容
{% endblock %}

# 子页面 就可以声明想要修改哪块划定的区域
{% block content %}
  子页面内容
  子页面除了可以自己写自己的之外 还可以继续使用模板的内容
  {{ block.super }}
{% endblock %}

# 一般情况下 模板页面上应该至少有三块可以被修改的区域
# 这样划分之后 每一个子页面 就都可以有自己独有的css代码 html代码 js代码
  1. html区域
    {% block content %}
     子页面自己的html内容
    {% endblock %}
  2. css区域
    {% block css %}
      子页面自己的css
    {% endblock %}
  3. js区域
    {% block js %}
      子页面自己的js
    {% endblock %}

"""
一般情况下 模板的页面上 划定的区域越多 那么该模板的扩展性就越高
但是如果太多 那还不如自己写(没必要划分太多)
利用模板的继承 能够让自己的页面更加的好维护
"""
```

### 模板导入

```python
"""
将页面的某一个局部(e.g：一个好看的表单/导航栏等等) 当成模块的形式
哪个地方需要就可以直接导入使用即可
"""

{% include 'ok.html' %}
```

## 模型层

**重要：跟数据打交道**

### 测试脚本

**参考ORM部分得出的测试脚本**

```python
import os
import django

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'cmdb.settings')
# 加载django配置
django.setup(set_prefix=False)

# 在下面继续写代码 就可以测试django里面的单个py文件
from app.model import User  # 不能拿到最上面 所有的代码都必须等待环境准备完毕之后才能书写
...
```

#### 单表查询

```python
# django自带的sqlite3数据库对日期格式不是很敏感 处理的时候 容易出错

# 单表环境准备 modes.py
class User(models.Model):
    name = models.CharField(max_length=32)
    age = models.IntegerField()
    reg_time = models.DateField()
    """
    DateField
    DateTimeField
    这2个字段有2个重要参数：
      auto_now: 每次操作数据的时候 该字段会自动将当前时间更新
      auto_now_add: 在每次创建数据的时候 会自动将当前创建时间记录下来 之后只要不人为的修改 那么就一直不变
    """
```

#### 增

```python
# 1. create() 推荐 
# create方法 返回当前增加对象本身
res = User.objects.create(name='minhooo', age='18', register_time='2002-10-20')
print(res)

# 2. save()
import datetime
ctime = datetime.datetime.now()  
# 日期字段可以接受字符串 也可以接受datatime对象
user_obj = User(name='kimi', age=25, register_time=ctime)
user_obj.save()
```

#### 删

```python
# 删
"""
pk会自动查找到当前表的主键字段 指代的就是当前表的主键字段
用了pk之后 你就不需要指定当前表的主键字段到底叫什么
e.g: uid pid sid ...
"""
# 1. delete() 推荐
res = User.objects.filter(pk=3).delete()
print(res)

# 2. user_obj的delete() 先查再删 两条sql语句
user_obj = User.objects.filter(pk=6).first()
user_obj.delete()
```

#### 改

```python
# 改 
# 1. update()
User.objects.filter(pk=4).update(name='lomen')

# 获取user_obj 然后修改属性 调用save()更新
user_obj = User.objects.get(pk=4)  # 不推荐使用get
user_obj = User.objects.filter(pk=6)
print(user_obj)
user_obj.name = 'pomno'
user_obj.save()
"""
get方法返回的直接就是当前数据对象
但是不推荐使用该方法:
  - get() 一旦数据不存在 该方法会直接报错
  - filter() 不会报错 返回空QuerySet 还是使用filter
"""
```

#### 必知必会13条

```python
1. all()         # 查询所有数据
2. filter()      # 带有过滤条件的查询   WHERE
3. get()         # 直接拿数据对象 但是条件不存在直接报错
4. first()       # 拿QuerySet对象的第一个元素
5. last()        # 拿QuerySet对象的最后一个元素

# 6 7 重要 经常使用
6. values()  # 指定获取数据字段 select name, age from ...
res = User.objects.values('name', 'age')
print(res)   # 返回值类似 列表套字典
>>> <QuerySet [{'name': 'Minho', 'age': 18}, {'name': 'pomno', 'age': 29}]>

7. values_list()  # 用法同values 返回值类似 列表套元组

8. distinct()  # 去重
res = User.objects.values('name').distinct(
"""
去重一定要是一模一样的数据 如果带有主键 那么肯定不一样
你在往后的查询中 一定不要忽略主键
"""

9. order_by()  # 排序
res = User.objects.order_by('age')  # 默认升序
res = User.objects.order_by('-age')  # 前面加- 降序

10. reverse()  # 反转 反转的前提是 数据已经排过序
# 需要用在order_by之后 所以用的不多
res1 = User.objects.order_by('age').reverse()
    
11. count()  # 统计
res = User.objects.count()
    
12. exclude()  # 排除在外 WHERE NOT
res = User.objects.exclude(name='Minho')
    
13. exists()  # 基本用不到 返回布尔值
# 直接拿前面的数据就可以进行布尔判断 随意基本用不到
res = User.objects.filter(pk=10).exists()

"""
查看内部封装的SQL语句的方式
  - 方式1：QuerySet_obj.query 只能用于queryset对象
  - 方式2：所有的sql语句都能查看 配置文件配置 详见参考上面ORM详解
"""
```

#### 双下划线查询

```python
# age 大于35 __gt
res = User.objects.filter(age__gt=35)

# age 小于35 __lt
res = User.objects.filter(age__lt=35)

# 小于等于__lte 大于等于__gte
res = User.objects.filter(age__lte=29)
res = User.objects.filter(age__gte=29)

# 成员查询 age是 18 or 29 or 55 
res = User.objects.filter(age__in=[18, 29, 55])

# 范围查询 age是18到40之间 首尾都要
res = User.objects.filter(age__range=[18, 55])

# 查询名字里面含有n的数据 模糊查询
# __contains LIKE BINARY
res = User.objects.filter(name__contains='n')

# 忽略大小写 __contains LIKE
res = User.objects.filter(name__icontains='N')

# __startswith __endswith
res = User.objects.filter(name__startswith='M')
res = User.objects.filter(name__endswith='o')

# 查询出注册时间是 2020.1月份的数据
# 按照月份 或者年份 等 查找数据
res = User.objects.filter(reg_time__month='1')
res = User.objects.filter(reg_time__year='2020')
```

### 多表操作

```python
# models.py 模型准备
# 创建表关系 先将基表创建出来 然后再添加外键字段
class Book(models.Model):
    title = models.CharField(max_length=32)
    price = models.DecimalField(max_digits=8, decimal_places=2)  # 小数总共8位 小数点后面占2位
    publish_date = models.DateTimeField(auto_now_add=True)
    """
    图书和出版社是一对多 并且书是多的一方 所以外键字段放在book表里面
    如果字段对应的是ForeignKey 那么ORM会自动在字段后面加_id -> publish_id
    后面定义ForeignKey的时候 不要自己加_id
    """
    publish = models.ForeignKey(to='Publish', on_delete=models.CASCADE)  # 默认就是与出版社表的主键字段做外键关联 可以通过to_field设置
    """
    图书和作者是多对多关系 外键字段建在任意一方均可 但是推荐创建再查询频率较高的一方
    authors是一个虚拟字段 主要是用来高速ORM 书籍表 和 作者表 是多对多关系 让ORM自动帮你创建第三张表关系
    
    django 1.x 自动级联更新删除 2.x 3.x需要加参数
    """
    authors = models.ManyToManyField(to='Author')

class Publish(models.Model):
    name = models.CharField(max_length=32)
    address = models.CharField(max_length=64)
    email = models.EmailField()
    # 本质还是varchar(254) 该字段类型不是给models看的 而是给我们后面会学到的校验型组件看的
    def __str__(self):
        return "<Publish {}>".format(self.name)

class Author(models.Model):
    name = models.CharField(max_length=32)
    age = models.IntegerField()
    """
    作者与作者详情是一对一关系 外键字段建在任意一方都可以 但是推荐创建在查询频率较高的表中
    OneToOneField 也会自动给字段加_id后缀
    """
    author_detail = models.OneToOneField(to='AuthorDetail', on_delete=models.CASCADE)

class AuthorDetail(models.Model):
    phone = models.BigIntegerField()  # 电话号码用BigIntergerField或者直接用CharField
    address = models.CharField(max_length=32)
```

#### 一对多

**一对一 一对多外键的增删改查**

- **增**

```python
# 1. 直接写外键实际字段 放fk_id
models.Book.objects.create(title='三国演义', price=123.23, publish_id=1)

# 2. 虚拟字段 放对象
publish_obj = models.Publish.objects.filter(pk=2).first()
models.Book.objects.create(title='红楼梦', price=666.32, publish=publish_obj)
```

- **删**

```python
# 删
models.Publish.objects.filter(pk=1).delete()  # django 1.x 默认级联删除
```

- 改

```python
# 修改
# 1. 同增加
models.Book.objects.filter(pk=2).update(publish_id=4)

# 2. 同增加
publish_obj = models.Publish.objects.filter(pk=2).first()
models.Book.objects.filter(pk=2).update(publish=publish_obj)
```

#### 多对多

**多对多的 增删改查 就是在操作第三张表**

- **增**

```python
# 如何给书籍添加作者
book_obj = models.Book.objects.filter(pk=4).first()
print(book_obj.authors)  
# 第三张表不是我们创建 无法获取model
# book_obj.authors 就类似于你已经取到了第三张表

# add() 传主键id
book_obj.authors.add(1)   # 书籍ID为2的书籍 绑定一个主键为1的作者
book_obj.authors.add(2, 3)   # 添加多个作者

# add() 传对象
author_obj = models.Author.objects.filter(pk=1).first()
author_obj2 = models.Author.objects.filter(pk=2).first()
author_obj3 = models.Author.objects.filter(pk=3).first()
book_obj.authors.add(author_obj2, author_obj3)
"""
add给第三张关系表添加数据
括号内 既可以传对象 也可以传数字(id) 并且都支持传多个值
"""
```

- **删**

```python
# 删
book_obj = models.Book.objects.filter(pk=4).first()

# 传id
book_obj.authors.remove(2)
book_obj.authors.remove(1, 3)

# 传对象
author_obj = models.Author.objects.filter(pk=3).first()
author_obj2 = models.Author.objects.filter(pk=2).first()
book_obj.authors.remove(author_obj, author_obj2)
```

- 改

```python
# 改
book_obj = models.Book.objects.filter(pk=2).first()

# 传id
book_obj.authors.set((1, 2))  # 括号内必须给一个可迭代对象
book_obj.authors.set((3,))  # 先删除数据 再增加

# 传对象
author_obj = models.Author.objects.filter(pk=3).first()
author_obj2 = models.Author.objects.filter(pk=2).first()
book_obj.authors.set([author_obj, author_obj2])
"""
set()
  括号内必须传一个可迭代对象 该可迭代对象内 既可以传对象 也可以传数字(id) 都支持多个
"""
```

- **清空**

```python
# 清空
# 在第三张关系表中 清空某个书籍与作者的绑定关系
# DELETE FROM `app02_book_authors` WHERE `app02_book_authors`.`book_id` = 2
book_obj = models.Book.objects.filter(pk=2).first()
book_obj.authors.clear()
"""
clear()
  括号内不要加任何参数
"""
```

### 跨表查询(重点)

```python
# 正反向的概念
看外键字段在哪儿
外键字段在我手上 我查你就是正向
否则 外键字段不在我手上 我查你就是反向

# 正向 反向
假设：
一本书对应一个出版社 
一个出版社可以出版多本书
那么 外键字段在 书 这边
正向：由 书 查询 出版社
反向：由 出版社 查询 书

book    >>> 外键字段在书哪儿(正向) >>> publish
publish >>> 外键字段在书哪儿(反向) >>> book

一对一和多对多的正反向判断也是如此

"""
正向查询按外键字段
反向查询按表名小写
         _set.all()
"""

"""
多表操作
  1. 子查询  
    ORM --> 基于对象的跨表查询
    - 先拿到一个数据对象
    - 对象点点点 就能拿到对应的字段
  
  2. 联表查询
    ORM --> 基于双下划线的跨表查询
    
    inner join
    left join
    right join
    union
"""

1. 基于对象的跨表查询
  # 正向 
  book_obj.publish
  book_obj.authors.all()
  author_obj.author_detail
    
  # 反向
  publish_obj.book_set  # App01.Book.None
  publish_obj.book_set.all()
  author_obj.book_set.all()
  author_detail_obj.author

2. 基于双下划线的跨表查询
  # 等价的一对正向和反向查询
  models.Book.objects.filter(pk=1).values('title', 'publish__name')
  models.Publish.objects.filter(book__id=1).values('book__title', 'name')
    
  # 其他字段同理
  # 利用双下划线的跨表查询 可以帮助你跨N多张表 只要有外键字段
  models.Book.objects.filter(pk=1).values('authors__author_detail__phone')
```

#### 子查询

**基于对象的跨表查询**

- **正向**

```python
# 基于对象的跨表查询 先拿到关系字段在的对象

# 1. 查询书籍主键为2的出版社名称
book_obj = models.Book.objects.filter(pk=2).first()
# 书查出版社 正向 按字段
res = book_obj.publish
print(res, res.name, res.address)

# 2. 查询书籍主键为2的作者
book_obj = models.Book.objects.filter(pk=4).first()
# 书查作者 正向
# res = book_obj.authors  # app02.Author.None 发现结果类似这样 ORM可能没错 少了 .all()
res = book_obj.authors.all()  # <QuerySet [<Author: Author object (2)>, <Author: Author object (3)>]>
print(res)

# 3. 查询作者minho的电话号码
author_obj = models.Author.objects.filter(name='minho').first()
res = author_obj.author_detail
print(res, res.phone, res.address)

"""
在书写ORM语句的时候 跟写sql语句一样的
不要试图一次将orm语句写完 如果比较复杂 就写一点看一点

正向什么时候需要加 .all()
  当你的结果可能有多个的时候 就需要加 .all()
  如果是一个结果 则直接能拿到数据对象
    book_obj.publish
    book_obj.authors.all()
    author_obj.author_detail
"""
```

- **反向**

```python
# 4. 查询出版社是东方出版社出版的书
publish_obj = models.Publish.objects.filter(name='东方出版社').first()
# 出版社查书 反向
res = publish_obj.book_set  # app02.Book.None
res = publish_obj.book_set.all()
print(res)

# 5. 查询作者是minho写过的书
author_obj = models.Author.objects.filter(name='minho').first()
# 作者查书 反向
res = author_obj.book_set  # app02.Book.None
res = author_obj.book_set.all()
print(res)

# 5. 查询手机号是110的作者姓名
author_detail_obj = models.AuthorDetail.objects.filter(phone='110').first()
res = author_detail_obj.author
print(res, res.name)
"""
基于对象 反向查询的时候
  当你的查询结果可以有多个的时候 就必须加 _set.all()  一对多 多对多 反向查询
  当你的结果只有一个的时候 不需要加 _set.all()  一对一反向查询
"""
```

#### 联表查询

**基于双下划线的跨表查询**

```python
# 1. 查询minho的手机号
res = models.Author.objects.filter(name='minho').values('author_detail__phone', 'name')
print(res)
# 反向
res = models.AuthorDetail.objects.filter(author__name='minho').values('phone', 'author__name')  # 拿作者姓名是minho的作者详情
print(res)

# 2. 查询书籍为2的出版社名称 和 书的名称
res = models.Book.objects.filter(pk=2).values('publish__name', 'title')
print(res)
# 反向
res = models.Publish.objects.filter(book__id=2).values('name', 'book__title')
print(res)

# 3. 查询书籍主键为3的作者姓名
res = models.Book.objects.filter(pk=3).values('authors__name')
print(res)
# 反向
res = models.Author.objects.filter(book__id=3).values('name')
print(res)


# 查询书籍主键是2的作者的手机号
# 涉及：book author authordetail
res = models.Book.objects.filter(pk=3).values('authors__author_detail__phone')
print(res)
"""
只要掌握了正反向的概念 以及双下划线查询
那么你就可以无限制的跨表
"""
```

### 聚合查询

```python
# 原生：max min sum count avg
# orm: aggregate

# 聚合查询 aggregate
"""
聚合查询通常情况下 都是配合分组一起使用的
只要是跟数据库相关的模块
  基本上都在 django.db.models里面
  如果上述没有那么应该在 django.db里面
"""
from django.db.models import Max, Min, Avg, Count, Sum

# 1. 书的平均价格
res = models.Book.objects.aggregate(Avg('price'))
print(res)
# 2. 上述方法一次性使用
res = models.Book.objects.aggregate(Max('price'), Min('price'), Sum('price'), Count('pk'), Avg('price'))
print(res)
```

### 分组查询

```python
# 原生：group by
# orm: annotate
# 分组查询 annotate
"""
MySQL分组查询都有哪些特点
  分组之后 默认只能获取到分组的依据 组内其他字段 都无法直接获取
  严格模式：ONLY_FULL_GROUP_BY
"""
# 1. 统计每一本书的作者个数
from django.db.models import Max, Min, Avg, Count, Sum
res = models.Book.objects.annotate()  # models后面点什么 就是按什么分组
# res = models.Book.objects.annotate(author_num=Count('authors')).values('title', 'author_num')   # 与下面等价 不用加__id
res = models.Book.objects.annotate(author_num=Count('authors__id')).values('title', 'author_num')
"""
author_num 是我们自己定义的字段(别名) 用来存储统计出来的每本书对应的作者个数
"""
print(res)

# 2. 统计每个出版社卖得最便宜的书的价格
res = models.Publish.objects.annotate(min_price=Min('book__price')).values('name', 'min_price')
print(res)

# 3. 统计不止一个作者的图书
# 先按着图书分组 然后过滤出不止一个作者的图书
res = models.Book.objects.annotate(author_num=Count('authors')).filter(author_num__gt=1).values('title', 'author_num')
"""
只要你的orm语句得出的结果 还是一个queryset对象
那么它就可以继续无限制的使用点queryset对象封装的方法
"""
print(res)

# 4. 查询每个作者出的书的总价格
res = models.Author.objects.annotate(sum_price=Sum('book__price')).values('name', 'sum_price')
print(res)

"""
如果我想按照指定的字段分组 该如何处理?
  models.Book.objects.values('price').annotate()
  如果 annotate()方法前面 有values指定字段 那么就按照指定字段分组
  否则 按照models后面指定的表分组
  
  如果出现分组查询报错的情况 你需要修改数据库严格模式
"""
```

### F与Q查询

- **F查询**

```python
# 1. 查询卖出数大于库存数的书籍
# F查询
"""
能够帮助你直接获取到表中某个字段对应的数据 当作查询条件
"""
from django.db.models import F
res = models.Book.objects.filter(maichu__gt=F('kucun'))
print(res)

# 2. 将所有书籍的价格提升50块
models.Book.objects.update(price=F('price') + 500)

# 3. 将所有书的名称后面加上爆款两个字
"""
在操作字符类型的数据的时候 F不能够直接做到字符串的拼接
"""
from django.db.models.functions import Concat
from django.db.models import Value
models.Book.objects.update(title=Concat(F('title'), Value('爆款')))
# models.Book.objects.update(title=F('title') + '爆款')  # 所有的名称会全部变为空白
```

- **Q查询**

```python
# Q 实现查询的与 或 非

# Q查询
# 1. 查询卖出数大于100或者价格小于400的书籍
"""
filter括号内 多个参数是and关系
"""
res = models.Book.objects.filter(maichu__gt=100, price__lt=400)  # filter 多个参数默认是AND关系
from django.db.models import Q
res = models.Book.objects.filter(Q(maichu__gt=100), Q(price__lt=400))  # Q包裹 逗号分割 还是AND关系
res = models.Book.objects.filter(Q(maichu__gt=100) | Q(price__lt=400))  # | OR关系
res = models.Book.objects.filter(~Q(maichu__gt=100) | Q(price__lt=400))  # ~ NOT关系
print(res)

# Q的高阶用法 能够将查询条件的左边也变成字符串的形式 而不是变量
q = Q()
q.connector = 'or'  # Q对象链接 children 默认是AND关系 .connector修改
q.children.append(('maichu__gt', 100))
q.children.append(('maichu__lt', 600))
res = models.Book.objects.filter(q)  # filter括号内支持直接放Q对象 默认还是AND关系 可以修改
print(res)
```

### django中如何开启事务

```python
"""
事务 ACID
  A 原子性
    不可分割的最小单位
  
  C 一致性
    跟原子性是相辅相成的
  
  I 隔离性
    事务之间户型不干扰
  
  D 持久性
    事务一旦确认永久生效
    
事务的回滚
  rollback
  
事务的确认
  commit
"""

# Django中如何简单的开启事务
from django.db import transaction

try:
    with transaction.atomic():
        # sql1
        # sql2
        ...
        # 在with代码块内书写的所有orm操作 都是同一个事务
        orm...
except Exception as err:
    print(err)
print('执行其他操作')
```

### orm中常用字段及参数

[DjangoORM字段类型参考](https://docs.djangoproject.com/zh-hans/3.1/ref/models/fields/)

[博客参考](https://www.cnblogs.com/Dominic-Ji/p/9203990.html)

```python
AutoField
  主键字段 primary_key=True
    
CharField  ->  varchar
  verbose_name  # 字段的注释
  max_length    # 长度 必须提供

InterField  ->  int
BigInterField   ->  bigint

DecimalField
  max_digits=8      # 8位数字
  decimal_places=2  # 2位小数

EmailField  ->  varchar(254)

DateField  ->  date
DateTimeField  ->  datetime
  auto_now      # 每次修改数据的时候 都会自动更新当前时间
  auto_now_add  # 只在创建数据的时候记录创建时间 后续不会自动修改

BooleanField(Field) - 布尔值类型
  该字段传布尔值(False/True) 数据库里面存0/1

TextField(Field) - 文本类型
  该字段可以用来存大段内容(文章 博客) 没有字数限制
      
FileField(Field) - 字符类型
  upload_to = "/data"  # 给该字段传一个文件对象 会自动将文件保存到/data目录下 然后将文件路径保存到数据库中 /data/a.txt
    
...

# 外键字段及参数
# 你在用ForeingKey(unique=True)字段创建一对一关系 orm会有一个提示 orm推荐你使用OneToOneField()
unique=True
  ForeignKey(unique=True) == OneToOneField()
    
db_index
  如果db_index=True 则代表为此字段设置**索引**
    
to_field
  要关联的表的字段 默认不写关联的就是另外一张表的主键字段

on_delete
  当删除关联表中的数据时 当前表与其关联的行的行为
  """
  django2.x及以上版本时 需要你自己指定外键字段的级联更新 级联删除
  """
db_constraint
  是否在数据库中创建外键约束 默认为True

...
# django除了给你提供了很多字段类型之外 还支持你自定义字段
```

- **自定义字段**

```python
# 自定义
class MyCharField(models.Field):
    def __init__(self, max_length, *args, **kwargs):
        self.max_length = max_length
        # 一定要是关键字传参 按位置传参会出错(看源码)
        super().__init__(max_length=max_length, *args, **kwargs)

    def db_type(self, connection):
        """
        返回真正的数据类型及各种约束条件
        """
        return 'char({})'.format(self.max_length
                                 
# 使用
myfield = MyCharField(max_length=16, null=True)
```

### choices参数

**数据库字段设计常见**

**choicesc参数 在实际项目中 使用场景是非常广泛的**

[Field.choices](https://docs.djangoproject.com/zh-hans/3.1/ref/models/fields/#choices)

```python
"""
用户表
  - 性别
  - 学历
  - 工作经验
  - 是否结婚
  - 客户来源
  ...
  
针对某个可以列举完全的可能性字段 我们应该如何存储?
总结：只要某个字段的可能性是可以列举完全的 那么一般情况下 都会采用choices参数
"""

score_choices = [
    ('A', '优秀'),
    ('B', '良好'),
    ('C', '及格'),
    ('D', '不及格')
]
# 保证字段类型 跟列举出来的元组的第一个数据类型一致即可
score = models.CharField(choices=score_choices)
```

- **model准备**

```python
# model准备
class User(models.Model):
    name = models.CharField(max_length=32)
    age = models.IntegerField()
    # 性别字段
    # 存布尔值 不是特别合理 没有对应关系 True/Flase分别对应什么性别?
    # gender = models.BooleanField()
    gender_choices = [
        (1, '男'),
        (2, '女'),
        (3, '其他'),
    ]
    gender = models.IntegerField(choices=gender_choices)  # 看choices里面元组的第一个元素是什么类型 就是什么字段
    """
    该gender字段存的还是数字 但是如果存的数字在上面元组列举的范围之内
    那么就可以非常轻松的获取到数字对应的真正的内容
    
    1. gender字段存的数字 不在上述元组列举的范围内容?
    
    2. 如果在 如何获取对应的中文信息?
    """
    def __str__(self):
        return "<User {}>".format(self.name)
```

- **测试**

```python
# 增
# 存的时候 没在choices列举的范围之内的数字 也能存进去
# 存的数据范围 还是按照字段类型决定 IntegerField
models.User.objects.create(name='mm', age=18, gender=1, register_time='2020-10-1')
models.User.objects.create(name='nn', age=67, gender=2, register_time='2020-10-1')
models.User.objects.create(name='bb', age=22, gender=3, register_time='2020-10-1')
models.User.objects.create(name='vv', age=9, gender=4, register_time='2020-10-1')

# 取
user_obj = models.User.objects.filter(pk=10).first()
# print(user_obj.gender)
# 只要是choices参数的字段 如果你想要获取对应信息 固定写法: get_字段名_display()
print(user_obj.get_gender_display())

# 如果没有对应关系 字段是什么还是展示什么
user_obj = models.User.objects.filter(pk=12).first()
print(user_obj.get_gender_display())
```

### 数据库查询优化

```python
"""
ORM语句的特点：惰性查询
  如果你仅仅只是书写了ORM语句 在后面根本没有用到该语句查询出来的参数
  那么ORM会自动识别 直接不执行
  要用到数据的时候 才会走数据库
"""

# only 与 defer
# select_related 与 prefetch_related
```

#### only与defer

```python
# only
# 获取user表中 所有用户的名字
res = User.objects.values('name')
for d in res:
    print(d.get('name'))

# 实现 获取到的是一个数据对象 然后 点name就能拿到名字 并且没有其他字段
# only使用
res = User.objects.only('name')
for d in res:
    print(d.name)  # 点only括号内的字段 不会走数据库
    print(d.age)   # 点only括号内没有的字段 会重新走数据库字段 而all()不需要
    
# defer使用
res = User.objects.defer('name')  # 还是返回数据对象 除了没有name属性之外 其他的都有
for i in res:
    print(i.name)  # 5条SQL
    print(i.age)   # 1条SQL

"""
defer与only刚好相反
  defer括号内放的字段不在查询出来的对象里面 查询该字段需要重新走数据库
  而如果查询的是非括号内的字段 则不需要走数据库
  
总结：
  - only  只会缓存指定字段的结果 后面循环去取指定字段 只会有1条SQL 取其他字段 N条SQL
  - defer 会缓存除指定字段以外的全部字段结果 后面循环取指定字段 N条SQL 取其他字段 1条SQL
"""
```

#### select_related与prefetch_related

**跟跨表操作有关**

```python

res = models.Book.objects.all()
for i in res:
    print(i.publish.name)  # 每循环一次 就走一次数据库查询

# select_related
"""
select_related 内部直接先将book与publish连起来 
然后一次性将大表里面的所有数据 全部封装给查询出来的对象
这个时候对象无论是点击book表的数据 还是publish的数据 都无需再走数据库查询了

注意事项：
  select_related括号内 只能放外键字段：
    - 一对一 一对多 可以
    - 多对多 不行 抛出异常

"""
res = models.Book.objects.select_related('publish')  # INNER JOIN 联表
for i in res:
    print(i.publish.name)
    
# prefetch_related 
res = models.Book.objects.prefetch_related('publish')  # 子查询
"""
prefetch_related 该方法内部其实就是子查询
  将子查询查询出来的所有结果也给你封装到对象中
  给你的感觉好像也是一次性搞定的
"""
for i in res:
    print(i.publish.name)
    
# select_related与prefetch_related各有优缺点 不一定谁一定好 根据实际请况来看
```

### 批量插入数据

```python
# 循环一次次插入数据
from . import models

def ab_pl(request):
    # 先给book插入10000条数据
    for i in range(10000):
        models.Book.objects.create(title='第{}本书'.format(i))
    # 再将所有的数据查询并展示到前端页面
    book_queryset = models.Book.objects.all()
    return render(request, 'ab_pl.html', locals())

    # 这个时候访问指向该视图函数的url的时候 页面会一致转圈 卡顿很久(根据数据量决定) - 慢 甚至超时

def ab_pl(request):
    """
    当你想要批量插入数据的时候 使用orm给你提供的bulk_create能够大大的减少操作时间
    INSERT INTO `app01_book` (`title`) VALUES ('第0本书'), ('第1本书'), ...
    """
    # 批量插入 极快
    book_list = [models.Book(title="第{}本书".format(i)) for i in range(10000)]
    models.Book.objects.bulk_create(book_list)
    book_queryset = models.Book.objects.all()
    return render(request, 'ab_pl.html', locals())
```

## Forms组件

[使用表单](https://docs.djangoproject.com/zh-hans/3.1/topics/forms/)

### 前戏

```python
"""
写一个注册功能
  获取用户名和密码 利用form表单提交数据
  在后端判断用户名和密码 是否符合一定的条件
    - 名字中不能含有 金瓶梅
    - 密码不能少于三位
  如果符合条件 需要你将提示信息展示到前s端页面
"""
```

- **V1版本 自己实现**

```python
# VIEW
def ab_form(request):
    back_dic = {'username': '', 'password': ''}
    """
    无论是post请求还是get请求 页面都能够获取到字典
    只不过get请求来的时候 字典值都是空的
    而post请求来之后 字典可能有值
    """
    if request.method == 'POST':
        username = request.POST.get('username')
        password = request.POST.get('password')
        if '金瓶梅' in username:
            back_dic['username'] = '不符合社会主义核心价值观'
        if len(password) < 3:
            back_dic['password'] = '密码不能太短'
    return render(request, 'ab_form.html', locals())

# HTML
<form action="" method="post">
    <p>username:
        <input type="text" name="username">
        <span style="color: red">{{ back_dic.username }}</span>
    </p>
    <p>password:
        <input type="password" name="password">
        <span style="color: red">{{ back_dic.password }}</span>
    </p>
    <input type="submit">
</form>
```

- **V2版本 使用form组件**

```python
"""
V1版本中 需要自己做下面三件事情
  1. 手动书写前端获取用户数据的html代码  -> 渲染html代码
  2. 后端对用户数据进行校验            -> 校验数据
  3. 对不符合要求的数据进行前端提示      -> 展示提示信息
  
而form组件 可以把这三件事情 全部搞定
  - 校验规则 可以直接由form组件完成 不需要频繁请求后端
  - 把校验规则 提前告诉form组件
  
为什么数据校验非要去后端 不能在前端利用js直接完成呢?
  - 数据校验 前端可有可无 但是后端必须要有
  - 前端的校验是弱不禁风的 你可以直接修改 或者利用爬虫程序绕过前端页面直接朝后端提交数据
 
  e.g:
    购物网站 - 选取货物之后 计算一个价格发送给后端 如果后端不做价格校验
      实际是获取用户选择的所有商品的主键值 然后在后端查询出所有商品的价格 再次计算一遍
      如果跟前端一致 完成支付 如果不一致直接拒绝
"""
```

### 基本使用

```python
from django import forms
class MyForm(forms.Form):
    # username字符串最小3位 最大8位
    username = forms.CharField(min_length=3, max_length=8)
    # password字符串最小3位 最大8位
    password = forms.CharField(min_length=3, max_length=8)
    # email字段必须符合邮箱格式
    email = forms.EmailField()
```

### 校验数据

**环境准备及基本使用**

```python
"""
1. 测试环境的准备 可以自己拷贝代码准备
2. 其实在pycharm里面已经帮你准备了一个测试环境
    - 直接点击pycharm底部的 PythonConsole 它会帮你准备测试环境 导入相关模块
"""

>>> from app02 import views

# 1. 将待校验的数据组织成字典的形式传入(传递给需要验证的Form)
>>> form_obj = views.MyForm({'username':'minho', 'password':'123', 'email':'123'})

# 2. 验证数据是否合法 该方法只有在所有数据全部合法的请况下才会返回True
>>> form_obj.is_valid()    
False

# 3. 查看所有校验通过的数据
>>> form_obj.cleaned_data
{'username': 'minho', 'password': '123'}

# 4. 查看所有不符合校验规则以及不符合的原因
# 返回字典 key:验证字段 value:列表(报错信息不唯一 可能有多个)
>>> form_obj.errors    
{
    'email': ['Enter a valid email address.']
}

# 5. 校验数据只校验类中出现的字段 多传不影响 多传字段直接忽略
>>> form_obj = views.MyForm({'username':'minho', 'password':'123', 'email':'123@qq.com', 'hobby':'study'})
>>> form_obj.is_valid()
True
>>> form_obj.errors
{}

# 6. 校验数据 默认请况下 类里面所有的字段都必须传值
>>> form_obj = views.MyForm({'username':'minho', 'password':'123'})
>>> form_obj.is_valid()
False
>>> form_obj.errors
{'email': ['This field is required.']}

"""
校验数据的时候 默认情况下：数据可以多传但是绝不能少传
"""
```

### 渲染标签

**forms组件只会自动帮你渲染用户输入的标签(input select radio checkbox) 不会帮你渲染提交按钮**

```python
# view
def index(request):
    # 1. 先产生一个空对象
    form_obj = MyForm()
    # 2. 直接将该空对象传递给html页面
    return render(request, 'index.html', locals())

# 前端利用空对象做操作
<form action="" method="post">
    <p>第1种渲染方式: 代码书写极少 封装程度太高 不便于后续扩展 一般情况下只在本地测试使用</p>
    {{ form_obj.as_p }}
    {{ form_obj.as_ul }}
    {{ form_obj.as_table }}
    
    <p>第2种渲染方式：可扩展性很强 但是书写代码太多 一般情况下不用</p>
    <p>{{ form_obj.username.label }}: {{ form_obj.username }}</p>
    <p>{{ form_obj.password.label }}: {{ form_obj.password }}</p>
    <p>{{ form_obj.email.label }}: {{ form_obj.email }}</p>
    
    <p>第3种渲染方式：推荐使用 代码书写简单 并且扩展性也高</p>
    {% for form in form_obj %}
        <p>{{ form.label }}:{{ form }}</p>
    {% endfor %}
</form>

"""
label属性默认展示的是Form类中字段首字母大写的形式
也可以自己修改 直接给字段对象加label属性即可
  username = forms.CharField(min_length=3, max_length=8, label='用户名')
"""
```

### 展示提示信息

```python
"""
浏览器会自动帮你校验数据 但是前端的校验弱不禁风
如何让浏览器不做校验?
  - form标签加一个 novalidate属性
"""

# view
def index(request):
    # 1. 先产生一个空对象
    form_obj = MyForm()
    if request.method == 'POST':
        # 2. 获取用户数据并检验
        """
        1. 数据获取繁琐
        2. 校验数据需要构造成字典的格式传入才行
        注意：request.POST可以看成就是一个字典
        """
        # 3. 校验数据
        form_obj = MyForm(request.POST)
        # 4. 判断数据是否合法
        if form_obj.is_valid():
            # 5. 如果合法 操作数据库存储数据库
            return HttpResponse('ok')
        # 5. 不合法 有错误
    # 2. 直接将该空对象传递给html页面
    return render(request, 'index.html', locals())


# html
<form action="" method="post" novalidate>
    {% for form in form_obj %}
        <p>
            {{ form.label }}:{{ form }}
            <span style="color: red">{{ form.errors.0 }}</span>
        </p>
    {% endfor %}
    <input type="submit">
</form>

"""
1. 必备的条件 get请求和post请求 传给html页面的对象变量名必须一样
2. form组件当你的数据不合法的情况下 会保留你上次的数据 让你基于之前的结果进行修改 更加的人性化
"""

# 自定义错误提示信息 error_messages参数
class MyForm(forms.Form):
    username = forms.CharField(min_length=3, max_length=8, label='用户名',
                               error_messages={
                                   'min_length': '用户名最少3位',
                                   'max_length': '用户名最大8位',
                                   'required': '用户名不能为空'
                               })
    password = forms.CharField(min_length=3, max_length=8, label='密码',
                               error_messages={
                                   'min_length': '密码最少3位',
                                   'max_length': '密码最大8位',
                                   'required': '密码不能为空'
                               })
    email = forms.EmailField(label='邮箱', error_messages={
        'invalid': '邮箱格式不正确',
        'required': '邮箱不能为空'
    })
```

### 钩子函数(hook)

```python
"""
在特定的节点自动触发完成响应操作
钩子函数在form组件中 就类似于第二道关卡 能够让我们自定义校验规则

在forms组件中有2类钩子
  1. 局部钩子
    当你需要给单个字段增加校验规则的时候可以使用
  
  2. 全局钩子
    当你需要给多个字段增加校验规则的时候可以使用
"""

# 实际案例
1. 校验用户名中不能含有666    # 只是校验username字段 局部钩子

2. 检验密码和确认密码是否一致  # password confirm两个字段 全局钩子

# 钩子函数 在类里面书写方法即可
class MyForm(forms.Form):
    username = forms.CharField(...)
    password = forms.CharField(...)
    confirm_password = forms.CharField(...)
    email = forms.EmailField(...)

    # 钩子函数
    # 局部钩子
    def clean_username(self):
        # 获取到用户名
        username = self.cleaned_data.get('username')
        if '666' in username:
            # 展示错误信息
            self.add_error('username', '不能包含666')
        # 将钩子函数钩出去的数据(单个数据)再放回去
        return username

    # 全局钩子
    def clean(self):
        password = self.cleaned_data.get('password')
        confirm_password = self.cleaned_data.get('confirm_password')
        if not password == confirm_password:
            self.add_error('confirm_password', '两次密码不一致')
        # 将钩子函数钩出去的数据(全部数据)再放回去
        return self.cleaned_data

```

### 重要参数

[Django官网表单字段详解](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/)

**forms组件 其他参数及补充知识点**

```python
label              自定义字段名
error_messages     自定义报错信息
initial            默认值
required           是否必填 默认是True


"""
问题：
1. 字段没有样式

2. 针对不同类型的input如何修改
  - text
  - password
  - date
  - radio
  - checkbox
  ...


# 处理文本输入的部件 widget参数解决
https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/
  widget=forms.PasswordInput

# 调整样式 attrs参数解决
# 多个属性值 直接空格隔开即可
https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.Widget.attrs
  widget=forms.TextInput(attrs={'class':'form-contral c1 c2'})
"""

validators      第一道关卡还支持正则校验
参考：https://docs.djangoproject.com/zh-hans/3.1/ref/validators/
validators=[RegexValidator(r'^[0-9]+$', '请输入数字'),
            RegexValidator(r'^159[0-9]+$', '数字必须以159开头')]
```

### 其他字段类型

[选择器和复选框部件](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#selector-and-checkbox-widgets)

### Forms组件源码

```python
"""
切入点：
  form_obj.is_valid()
"""
def is_valid(self):
    """Return True if the form has no errors, or False otherwise."""
    return self.is_bound and not self.errors
    """
    如果is_vaild要返回True的话 那么：
      - self.is_bound()要为True
      - self.errors()要为False
    """
    
# 只要传值 那么self.is_bound就为True
self.is_bound = data is not None or files is not None

@property
def errors(self):
    """Return an ErrorDict for the data provided for the form."""
    if self._errors is None:
        self.full_clean()
    return self._errors

# forms组件所有的功能基本都出自于该方法
def full_clean(self):
    """
    Clean all of self.data and populate self._errors and self.cleaned_data.
    """
    self._errors = ErrorDict()  # 创建类空字典
    if not self.is_bound:  # Stop further processing.
        return
    self.cleaned_data = {}
    # If the form is permitted to be empty, and none of the form data has
    # changed from the initial data, short circuit any validation.
    if self.empty_permitted and not self.has_changed():
        return

    # 这三行 整个forms组件精髓所在
    self._clean_fields()    # 检验字段源码 + 局部钩子
    self._clean_form()      # 全局钩子
    self._post_clean()      # An internal hook
```

### 校验字段/局部钩子源码解读

```python
def _clean_fields(self):
    for name, field in self.fields.items():
        # value_from_datadict() gets the data from the data dictionaries.
        # Each widget type knows how to retrieve its own data, because some
        # widgets split data over several HTML fields.
        # 获取字段对应的用户数据
        if field.disabled:
            value = self.get_initial_for_field(field, name)
        else:
            value = field.widget.value_from_datadict(self.data, self.files, self.add_prefix(name))
        try:
            if isinstance(field, FileField):
                initial = self.get_initial_for_field(field, name)
                value = field.clean(value, initial)
            else:
                value = field.clean(value)
            self.cleaned_data[name] = value  # 将合法的字段添加到cleaned_data
            if hasattr(self, 'clean_%s' % name):  # 利用反射获取局部钩子函数
                value = getattr(self, 'clean_%s' % name)()  # ()调用 局部钩子函数需要有返回值
                self.cleaned_data[name] = value
        except ValidationError as e:
            self.add_error(name, e)  # 添加提示报错信息
            
# 得出钩子函数第二种验证字段不通过的报错方式
# 局部钩子
def clean_username(self):
    # 获取到用户名
    username = self.cleaned_data.get('username')
    if '666' in username:
        # 展示错误信息
        # 报错方式1
        # self.add_error('username', '不能包含666')
        # 报错方式2 看源码得出 较为繁琐一般不用
        raise ValidationError('不能包含未允许的数字')
    # 将钩子函数钩出去的数据(单个数据)再放回去
    return username
```

### 全局钩子源码解读

```python
def _clean_form(self):
    try:
        cleaned_data = self.clean()  # 全局钩子需要一个返回值 self.cleaned_data
    except ValidationError as e:
        self.add_error(None, e)
    else:
        if cleaned_data is not None:
            self.cleaned_data = cleaned_data
```

#### _post_clean

```python
"""内部预留钩子 An internal hook"""
def _post_clean(self):
    """
    An internal hook for performing additional cleaning after form cleaning
    is complete. Used for model validation in model forms.
    """
    pass
```

## Cookie和Session

```python
"""
发展史：
  1. 网站都没有保存用户功能的需求 所有用户访问返回的结果都是一样的
    e.g: 新闻 博客 文章 ...
   
  2. 出现了一些需要保存用户信息的网站
    e.g: 淘宝 支付宝 京东 ...  (需要知道你是谁)
    
    以登陆功能为例：如果不保存用户登陆状态 也就意味着用户每次访问网站都需要重复的输入用户名和密码(你觉得这样的网站 你还想用吗?)  
    
    当用户第一次登陆成功之后 将用户的 用户名密码 返回给用户浏览器 让用户浏览器保存在本地 之后访问网站的时候浏览器自动将保存在浏览器上的用户名和密码发送给服务端 服务端获取之后自动校验
    早期这种方式具有非常大的安全隐患(能够直接看到)
    
    优化：
      当用户登陆成功之后 服务端产生一个随机字符串(在服务端保存数据 用K:V键值对的形式) 交由客户端浏览器保存
      随机字符串1：用户1相关信息
      随机字符串2：用户2相关信息
      随机字符串3：用户3相关信息
      ...
      之后访问服务端的时候 都带着该随机字符串 服务端去数据库中比对是否有对应的随机字符串从而获取到对应的用户信息
      但是如果你拿到了(截获)该随机字符串 那么你就可以冒充当前用户 真实还是有安全隐患的
    

**在WEB领域 没有绝对的安全 也没有绝对的不安全**
  基本上防御措施都需要程序员自己写代码完善 并且只能完善 没法杜绝
"""

# cookie
概念：服务端设置保存在客户端浏览器上的键值对(只要符合前面定义的都可以叫cookie)
  - 服务端保存在客户端浏览器上的信息都可以称之为cookie
  - 它的表现形式一般都是K:V键值对(可以有多个)
  - 浏览器可以选择不保存(但是会导致网页无法工作 比如无法登录)
    
# session
概念：存储在服务端上的键值对(一般情况都跟用户信息有关 用来标识当前用户)
  - 数据是保存在服务端的 
  - 并且它的表现形式一般也是K:V键值对(可以有多个)
  - 需要基于cookie才能工作(其实大部分的保存状态的实现都需要基于cookie来做)

# token
  - session虽然是保存在服务端的 但是经不住数据量大
  - 服务端不再保存数据 
    - 登陆成功之后 将一段信息用自己定制的加密方式(加密算法别人不知道)对内容进行加密处理 
    - 然后将加密结果拼接在这段信息后面 整体返回给用户浏览器保存 [[信息内容][加密内容]]
    - 浏览器下次访问的时候带着该信息 服务端自动切去前面一段信息再次使用自己的加密算法 跟浏览器尾部的密文进行比对
    
# jwt认证
  三段信息
    
总结：
  1. cookie就是保存在客户端浏览器上的信息
  2. session就是保存在服务端上的信息
  3. session是基于cookie工作的(其实大部分的保存用户状态的操作 都需要使用到cookie)
```

### Cookie操作

```python
"""
虽然cookie是服务端告诉客户端浏览器需要保存的内容 但是客户端浏览器可以选择拒绝保存 如果禁止掉 那么只要是需要记录用户状态的网站 登陆功能都无法使用
"""

# 视图函数的返回值
return HttpResponse()
return render()
return redirect()

# 如果你想要操作cookie 你就不得不利用HttpResponse对象
obj1 = HttpResponse()
操作cookie
return obj1

obj2 = render()
操作cookie
return obj2

obj3 = redirect()
操作cookie
return obj3
```

#### 设置cookie

```python
# obj in [obj1, obj2, obj3]
  obj.set_cookie(key, value)

# 在设置cookie的时候 可以添加一个超时时间
obj.set_cookie('username', 'minho123123', max_age=5/expires=5)
  - max_age
  - expires
  这两个参数都是设置超时时间的 并且都是以秒为单位
  需要注意的是 针对IE浏览器 需要使用expires

# 设置键值对的时候 可以加盐处理
obj.set_signed_cokkie(key, value, salt='盐')
```

#### 获取cookie

```python
# obj in [obj1, obj2, obj3]
  request.COOKIES
  request.COOKIES.get(key)

# 获取加盐cookie
request.get_signed_cookie(key, salt='盐')
```

#### 删除cookie

```python
# 如何主动删除cookie - 退出登录|注销
@login_auth
def logout(request):
    obj = redirect('/app03/login/')
    obj.delete_cookie('username')  # 删除cookie 实现退出登录注销功能
    return obj
```

#### cookie版登录验证

```python
from django.shortcuts import render, HttpResponse, redirect, reverse

def login(request):
    if request.method == 'POST':
        username = request.POST.get('username')
        password = request.POST.get('password')
        if username == 'minho' and password == '123':
            # 保存用户登陆状态
            obj = redirect(reverse('app03_home'))
            # 让浏览器记录cookie
            obj.set_cookie('username', 'minho123123')
            """
            浏览器不单单会帮你存
            而且后面每次访问你的时候 还会带着它过来
            """
            # 跳转到一个需要用户登陆之后再能看的界面
            return obj
    return render(request, 'login.html')

def home(request):
    # 获取cookie信息 判断你有没有
    if request.COOKIES.get('username') == 'minho123123':
        return HttpResponse('我是home页面 只有登陆的用户才能进来')
    # 没有登陆应该跳转到登陆页面
    return redirect(reverse('app03_login'))
```

- **登陆检验简单装饰器**

```python
"""
问题1：如果需要验证登陆的视图有非常多 这样写很冗余 -> 装饰器
解决：校验用户是否登陆的简单装饰器
"""
def login_auth(wrapped):
    def wrapper(request, *args, **kwargs):
        if request.COOKIES.get('username'):
            return wrapped(request, *args, **kwargs)
        return redirect(reverse('app03_login'))
    return wrapper
```

- **完整登陆代码示例**

```python
"""
问题2：
  有下面多个视图的情况下 我访问/index 会被重定向到/login
  但是 我输入用户名密码登陆之后 自动跳转到/home 而不是我想访问的/index/
  
  用户如果在没有登陆的情况下 想访问一个需要登陆的页面
  那么先跳转到登陆页面 当用户输入正确的用户名和密码之后
  应该跳转到用户之前想要访问的页面去 而不是直接写死
  
解决：见下面稍微完整的登陆示例代码
"""
from functools import wraps
from django.shortcuts import render, HttpResponse, redirect, reverse

def login_auth(wrapped):
    @wraps(wrapped)
    def wrapper(request, *args, **kwargs):
        # print(request.path)
        # print(request.get_full_path())  # 能够获取到用户上一次想要访问的url
        target_url = request.get_full_path()
        if request.COOKIES.get('username'):
            return wrapped(request, *args, **kwargs)
        # return redirect('/login/?next={}'.format(target_url))
        return redirect(reverse('app03_login') + '?next={}'.format(target_url))
    return wrapper

def login(request):
    if request.method == 'POST':
        username = request.POST.get('username')
        password = request.POST.get('password')
        if username == 'minho' and password == '123':
            # 获取用户上一次想要访问的url
            target_url = request.GET.get('next')  # 可能为None - 用户直接访问/login的情况
            if target_url:
                obj = redirect(target_url)
            else:
                # 保存用户登陆状态
                obj = redirect(reverse('app03_home'))
            # 让浏览器记录cookie
            obj.set_cookie('username', 'minho123123')
            """
            浏览器不单单会帮你存
            而且后面每次访问你的时候 还会带着它过来
            """
            # 跳转到一个需要用户登陆之后再能看的界面
            return obj
    return render(request, 'login.html')

@login_auth
def home(request):
    return HttpResponse('我是home页面 只有登陆的用户才能进来')

@login_auth
def index(request):
    return HttpResponse('Index Page, Need Login')

@login_auth
def users(request):
    return HttpResponse('Users Page, Need Login')
```

### Session操作

[如何使用会话](https://docs.djangoproject.com/zh-hans/3.1/topics/http/sessions/)

```python
"""
session数据是保存在服务端的(存到哪儿? 数据库 -> session表) 给客户端返回的是一个随机字符串
  客户端cookie -> sessionid:随机字符串

1. 在默认情况下操作session的时候 需要一张表来存 django默认提供一张django_session表
   django会默认创建很多张表 django_session就是其中一张(No such table:django_session)
   
2. django默认session的过期时间是14天
   但是你也可以人为的修改它
   
3. django_session表中的数据是取决于浏览器的
   同一个计算机上(IP地址) 同一个浏览器 只会有一条数据生效(当session过期的时候 可能会出现多条数据对应一个浏览器 但是这个数据没意义- 无效数据;但是该现象不会持续很久 内部会自动识别过期的数据清除 你也可以通过代码请除) 
   目的 -> 主要为了节省服务端资源
   
   通过一个唯一的cookieID 服务端就知道来的用户是谁 然后服务端根据不同的cookieID 在服务端上保存一段时间的私密资料(如账号密码等) 这就是服务端session保存
   
4. session是保存在服务端的 但是session的保存位置可以有多种选择
   4.1 MySQL
   4.2 文件
   4.3 redis/memcache...缓存
   4.4 其他
   ...
   
5. session还可以对保存的随机字符串做加盐处理

设置session
  - session可以设置多个 但是参考上面第3点
  - session也可以设置过期时间
  request.session['key'] = value    # 把request.session当作一个字典
   
获取session
  request.session.get('key')
  
清除session
  request.session.delete()    # 只删服务端当前session
  request.session.flush()     # 浏览器和服务端都清空(推荐使用)
"""
```

#### 设置session

```python
# views.py
def set_session(request):
    """
    设置session内部发生了哪些事：
      1. django内部会自动帮你生成一个随机字符串
      2. django内部自动将随机字符串和对应的数据存储到django_session表中 这一步不是直接生效的 分为下面两步
        2.1 先在内存中产生操作数据的缓存
        2.2 在响应结果经过django中间件的时候 才真正操作数据库
      3. 将产生的随机字符串返回给客户端浏览器保存(设置cookie) 
    """
    request.session['hobby'] = 'girl'
    return HttpResponse('Session Set')

# session设置过期时间
request.session.set_expire()
  括号内可以放4种类型的参数
    1. 整数                            多少秒只会失效
    2. 日期对象(datetime/timedelta)     到指定日期之后失效
    3. 0                               一旦当前浏览器关闭立刻失效
    4. None                            失效时间取决于django内部全局session默认的失效时间(14天)  
    
# 测试过期是否是生效
def set_session(request):
    request.session['hobby'] = 'girl'
    request.session['hobby1'] = 'girl1'
    request.session['hobby2'] = 'girl2'
    request.session.set_expiry(0)
    return HttpResponse('Session Set')

def get_session(r):
    # print(request.session.get('hobby'))
    if r.session.get('hobby'):
        print(r.session)  # 封装为一个对象 <django.contrib.sessions.backends.db.SessionStore object at 0x...>
        print(r.session.get('hobby1'))
        print(r.session.get('hobby2'))
        return HttpResponse('Session Get')
    return HttpResponse('No Session')  # 浏览器关闭后 重新打开则失效
```

- **django_session Table数据**

| session_key                      | session_data                                                 | expire_date         |
| -------------------------------- | ------------------------------------------------------------ | ------------------- |
| 0mzz5pprookey7rjd4m3pl6onoa5jdtm | NjRhZjljYzM1NDNkNzhkNmVkOWVkM2QzZGY2MGE1OWNhYzQwODBmNTp7ImhvYmJ5IjoiZ2lybCJ9 | 2021-03-10 01:59:31 |

- **客户端cookie**

| Name      | Value                            |
| --------- | -------------------------------- |
| sessionid | 0mzz5pprookey7rjd4m3pl6onoa5jdtm |

#### 获取session

```python
def get_session(request):
    """
    获取session内部发生了那些事：
      1. 自动从浏览器请求中获取sessionid对应的随机字符串
      2. 拿着该随机字符串去django_session中查找对应的数据
      3. 比对数据
        3.1 如果比对上 则将对应得数据取出并以字典得形式封装到request.session中
        3.2 如果比对不上 则request.session.get()返回的是None - dict.get()方法获取不到key 默认返回None
    """
    print(request.session.get('hobby'))
    return HttpResponse('Session Get')
```

#### 清除session

```python
def del_session(request):
    request.session.delete()    # 只删除服务端的有效session
    request.session.flush()     # 服务端和浏览器都删(推荐使用)
    return HttpResponse('Session Delete')
```

#### django中session配置

```python
1. 数据库Session
SESSION_ENGINE = 'django.contrib.sessions.backends.db'   # 引擎（默认）

2. 缓存Session
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'  # 引擎
SESSION_CACHE_ALIAS = 'default'                            # 使用的缓存别名（默认内存缓存，也可以是memcache），此处别名依赖缓存的设置

3. 文件Session
SESSION_ENGINE = 'django.contrib.sessions.backends.file'    # 引擎
SESSION_FILE_PATH = None                                    # 缓存文件路径，如果为None，则使用tempfile模块获取一个临时地址tempfile.gettempdir() 

4. 缓存+数据库
SESSION_ENGINE = 'django.contrib.sessions.backends.cached_db'        # 引擎

5. 加密Cookie Session
SESSION_ENGINE = 'django.contrib.sessions.backends.signed_cookies'   # 引擎

其他公用设置项：
SESSION_COOKIE_NAME ＝ "sessionid"                       # Session的cookie保存在浏览器上时的key，即：sessionid＝随机字符串（默认）
SESSION_COOKIE_PATH ＝ "/"                               # Session的cookie保存的路径（默认）
SESSION_COOKIE_DOMAIN = None                             # Session的cookie保存的域名（默认）
SESSION_COOKIE_SECURE = False                            # 是否Https传输cookie（默认）
SESSION_COOKIE_HTTPONLY = True                           # 是否Session的cookie只支持http传输（默认）
SESSION_COOKIE_AGE = 1209600                             # Session的cookie失效日期（2周）（默认）
SESSION_EXPIRE_AT_BROWSER_CLOSE = False                  # 是否关闭浏览器使得Session过期（默认）
SESSION_SAVE_EVERY_REQUEST = False                       # 是否每次请求都保存Session，默认修改之后才保存（默认）
```

#### session版登录验证

```python
from functools import wraps

def check_login(func):
    @wraps(func)
    def inner(request, *args, **kwargs):
        next_url = request.get_full_path()
        if request.session.get("user"):
            return func(request, *args, **kwargs)
        else:
            return redirect("/login/?next={}".format(next_url))
    return inner

def login(request):
    if request.method == "POST":
        user = request.POST.get("user")
        pwd = request.POST.get("pwd")

        if user == "alex" and pwd == "alex1234":
            # 设置session
            request.session["user"] = user
            # 获取跳到登陆页面之前的URL
            next_url = request.GET.get("next")
            # 如果有，就跳转回登陆之前的URL
            if next_url:
                return redirect(next_url)
            # 否则默认跳转到index页面
            else:
                return redirect("/index/")
    return render(request, "login.html")

@check_login
def logout(request):
    # 删除所有当前请求相关的session
    request.session.delete()
    return redirect("/login/")

@check_login
def index(request):
    current_user = request.session.get("user", None)
    return render(request, "index.html", {"user": current_user})


"""
总结：
  django默认给我们提供了一张 django_session表
  学了session之后 我们发现 我们通过 request.session.get(key) 可以拿到我们存储的数据 而且可以在任意视图直接使用

高阶用法：
  有时候 如果多个视图函数都需要使用到一些数据的话 你也可以考虑将该数据存储到django_session表中 方便后续的使用
  e.g: 登录验证码校验
"""
```

## Django中间件

### 中间件

```python
django自带七个中间件 每个中间件都有各自对应的功能 并且django还支持程序员自定义中间件
你在用django开发项目的时候 只要是涉及到 全局 相关的功能 都可以使用中间件方便的完成

e.g:
    - 全局用户身份校验
    - 全局用户权限校验
    - 全局访问频率校验(反爬虫措施之一 -- 破解：IP代理池)
    - ...(其他涉及全局的功能)
    
"""
django中间件是django的门户(请求来和走 两个方向都要经过)
  1. 请求来的时候 需要先经过中间件才能到达真正的django后端
  2. 响应走的时候 也需要经过中间件才能发送出去
"""

- 仔细理解django请求生命周期流程图
- 研究django中间件代码规律
```

#### 自带七个中间件

```python
# 列表里面其实不是字符串 是在导入模块(字符串方式导入模块) 模块路径的字符串形式
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# 看源码几个中间件的代码组织结构
"""
django支持程序员自定义中间件 并且暴露给程序员五个可以自定义的方法
  1. 第一类 必须掌握
    - process_request
    - process_response
  
  2. 了解即可
    - process_view 
    - process_template_response
    - process_exception
"""
class SessionMiddleware(MiddlewareMixin):
    def process_request(self, request):
        csrf_token = self._get_token(request)
        if csrf_token is not None:
            # Use same token next time.
            request.META['CSRF_COOKIE'] = csrf_token

    def process_response(self, request, response):
        return response
   
class CsrfViewMiddleware(MiddlewareMixin):
    def process_request(self, request):
        csrf_token = self._get_token(request)
        if csrf_token is not None:
            # Use same token next time.
            request.META['CSRF_COOKIE'] = csrf_token

    def process_view(self, request, callback, callback_args, callback_kwargs):
        return self._accept(request)
    
    def process_response(self, request, response):
        return response

class AuthenticationMiddleware(MiddlewareMixin):
    def process_request(self, request):
        request.user = SimpleLazyObject(lambda: get_user(request))
```

#### 自定义中间件

```python
"""
固定步骤：
  1. 在项目名或者应用名下创建以一个任意名称的文件夹
  2. 在该文件夹内创建一个任意名称的py文件
  3. 在该py文件内需要书写类(这个类必须继承MiddlewareMixin)
     3.1 在这个类里面 就可以自定义这5个方法
     3.2 这5个方法并不是全部都需要书写 用到几个写几个
  4. 需要将类的路径以字符串的形式注册到配置文件才能生效
  
# 注册中间件
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    '你自己写的中间件路径1',
    '你自己写的中间件路径2',
    '你自己写的中间件路径3',
]
"""
```

#### 中间件方法详解

```python
"""
django支持程序员自定义中间件 并且暴露给程序员五个可以自定义的方法
  1. 第一类 必须掌握
     process_request(self, request)
     process_response(self, request, response)
  
  2. 了解即可
     process_view(self, view_name, *args, **kwargs)
     process_template_response(self, request, response)
     process_exception(self, request, exception)
"""
```

##### process_request

**使用频率最高 最好用的 重点**

```python
1. 请求来的时候 需要经过每一个中间件里面的process_request方法 
   结果的顺序是按照配置文件中注册的中间件顺序 从上往下 依次执行
    
2. 如果中间件里面没有定义该方法 那么直接跳过执行下一个中间件

3. 如果该方法返回了HttpResponse对象 那么请求将不再继续往后执行 而是直接原路返回(e.g:校验失败不允许访问 反爬...)

总结：process_request方法 就是用来做全局相关的所有限制功能

"""
研究：
  如果在第一个process_request方法 就已经返回了HttpResponse对象 那么响应走的时候 是经过所有的中间件里面的process_response还是有其他情况?
  
  结果：其他情况 会直接走同级别(同一个中间件)的process_response 然后依次返回
  
  Flask框架也有类似的中间件 但是它的规律与django不一样：Flask中间件只要返回数据了就必须经过所有中间件里面的类似于process_response方法
"""
```

##### process_response

**重点**

```python
1. 响应走的时候 需要经过每一个中间件的process_response方法
   该方法有两个额外的参数 process_response(request, response)
   response: 默认就是后端视图函数返回给浏览器的内容
    
2. 该方法必须返回一个HttpResponse对象
   2.1 默认返回的就是形参response
   2.2 你也可以返回自己的内容

3. 顺序是按照配置文件中注册了的中间件 从下往上 依次经过
   如果你没有定义的话 直接跳过执行下一个
```

##### process_view

```python
def process_view(self, request, view_name, *args, **kwargs):
    print(view_name, args, kwargs)
    print('自定义中间件里面的process_view')
        
路由匹配成功之后 执行视图函数之前 会自动执行中间件里面的process_view方法
顺序是按照配置文件中注册的中间件顺序 从上往下 依次执行
```

##### process_template_response

```python
def index(request):
    print('我是视图函数index')
    obj = HttpResponse('index')
    def render():
        print('内部的render')
        return HttpResponse('ojbk')
    obj.render = render
    return obj

返回的HttpResponse对象有render属性的时候才会触发
顺序是按照配置文件中注册了的中间件 从下往上 依次经过
```

##### process_exception

```python
当视图函数中出现异常的情况下触发
顺序是按照配置文件中注册了的中间件 从下往上 依次经过
```

### 编程思想

```python
# 基于django中间件的一个重要的编程思想
```

#### importlib简单介绍

```python
# 有以下目录结构
Projects/
  - myfile/
    - b.py
  - a.py

# b.py
name = 'bbbbbb'

"""
问题：如何在a.py中使用b.py里面的变量?
"""
```

- **模块导入**

```python
# 方法1
from myfile import b

print(b.name)
```

- **importlib使用**

```python
# 方法2
import importlib

res = 'myfile.b'
ret = importlib.import_module(res)  # 内部处理：from myfile import b
print(ret)

# 该方法最小只能到模块(py文件名) 不能到py文件下面的变量(类 函数 变量...)
```

#### importlib进阶使用

1. **配置文件注册功能**
2. **importlib模块**
3. **字符串切割rsplit maxsplit参数**
4. **反射 getattr()...**
5. **面向对象鸭子类型**

```python
# notify.py 里面封装的三个通知函数
def wechat(content):
    print('微信通知：%s' % content)

def qq(content):
    print('qq通知：%s' % content)

def email(content):
    print('邮箱通知：%s' % content)

"""  
我在其他地方要使用这三个通知函数 可以全部导入 然后进行使用
问题：如果我很多地方都使用了这三个通知函数 现在我需要把其中一种方式取消掉 怎么办?
     1. 找到所有使用的地方 注释掉对应通知函数 - 可以解决 使用地方多的话 比较麻烦
     2. importlib高阶使用
"""
from notify import *

def send_all(content):
    wechat(content)
    # qq(content)
    email(content)
    
if __name__ == '__main__':
    send_all('啥时候放长假')
    
===============================
# 解决上述问题 把notify通知的方法封装为类到一个包里面的三个模块
notify/
  - __init__.py  # 包文件
  - email.py
  - qq.py
  - wechat.py

# email.py
class Email:
    def __init__(self):
        pass  # 发送邮件前需要做的前期准备工作

    def send(self, content):    # 其他通知封装类似 都提供一个send方法
        print('邮箱通知：%s' % content)
        
# 然后提供一个配置文件settings.py - django启发
NOTIFY_LIST = [
    'notify.email.Email',
    # 'notify.qq.QQ',
    'notify.wechat.Wechat',
]

# 重点：在notify包的__init__.py文件中做一下操作
import settings    # 导入配置文件
import importlib   # 导入importlib

def send_all(content):
    for path_str in settings.NOTIFY_LIST:
        module_path, class_name = path_str.rsplit('.', maxsplit=1)  # 从右往左切 只切一次
        # 拿到模块名和类名
        # module_path = 'notify.email' class_name = 'Email'
        # 1. 利用字符串导入模块
        module = importlib.import_module(module_path)  # from notify import email
        # 2. 利用反射获取类名
        cls = getattr(module, class_name)  # 拿到类名 Email QQ Wechat
        # 3. 生成类的对象(实例化)
        obj = cls()
        # 4. 利用鸭子类型直接拿调用send方法
        obj.send(content)
        
# 最后使用 start.py
"""
这个时候 可扩展性非常高
如果我们不想用哪个方法 直接去配置文件注释掉对应的配置即可
需要加通知方法 也很方便
"""
import notify

notify.send_all('通知')
```

### csrf跨站请求伪造

```python
"""
钓鱼网站(e.g: 大学英语4 6级 登录缴费钓鱼网站)
  我搭建一个跟正规网站一摸一样的界面(中国银行)
  用户不小心进入到了我们的网站 用户给某个人转钱
  打钱的操作确确实实提交给了中国银行的系统 用户的钱也确实减少了 唯一的不同就是目标账户不是用户想要转钱的账户 变成了一个莫名其妙的账户
  
内部本质：
  我们在钓鱼网站的页面 针对目标账号 只给用户提供一个没有name属性的普通input框
  然后我们在内部隐藏一个已经写好name和value的input框 提交的时候 action url写正常网站的url
  
=========
如何规避上述问题 -- csrf跨站请求伪造
  网站在给用户返回一个具有提交数据功能页面的时候 会给这个页面加一个唯一标识(不同的页面不一样 永远不重复)
  当这个页面朝后端发送post请求的时候 我的后端会先校验这个唯一标识 如果这个唯一标识不对 直接拒绝(403 forbidden) 如果成功则正常执行(csrf中间件帮我们搞定)
"""
```

#### Form表单如何校验

```html
<form action="" method="post">
  {% csrf_token %}
  <input type="hidden" name="csrfmiddlewaretoken" value="唯一的随机字符串">
  ...
</form>

<!-- hidden属性 隐藏 -->
<input type="hidden" name="csrfmiddlewaretoken" value="JCVNo1kxO4Z1WSUSmFkLBTFsO6XIYAl3Gr9VUfSQoWo6gBJn4jJxhn8V6YCWJxdQ">
```

#### Ajax如何校验

- **方式一**

```javascript
$.ajax({
  url: "/cookie_ajax/",
  type: "POST",
  data: {
    "username": "Tonny",
    "password": 123456,
    "csrfmiddlewaretoken": $("[name = 'csrfmiddlewaretoken']").val()  // 使用JQuery取出csrfmiddlewaretoken的值 拼接到data中
    // "csrfmiddlewaretoken": '{{ csrf_token }}'  // 或者利用模板语法提供的快捷书写
  },
  success: function (data) {
    console.log(data);
  }
})
```

- **方式二**

```javascript
$.ajax({
  url: "/cookie_ajax/",
  type: "POST",
  headers: {"X-CSRFToken": $.cookie('csrftoken')},  // 从Cookie取csrf_token 并设置ajax请求头
  data: {"username": "Q1mi", "password": 123456},
  success: function (data) {
    console.log(data);
  }
})
```

- **方式三(通用)**

```javascript
// 将这段代码配置到你的Django项目的静态文件中 直接导入该文件即可自动帮我们解决ajax提交post数据时校验csrf_token的问题(依赖jQuery)

function getCookie(name) {
    var cookieValue = null;
    if (document.cookie && document.cookie !== '') {
        var cookies = document.cookie.split(';');
        for (var i = 0; i < cookies.length; i++) {
            var cookie = jQuery.trim(cookies[i]);
            // Does this cookie string begin with the name we want?
            if (cookie.substring(0, name.length + 1) === (name + '=')) {
                cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                break;
            }
        }
    }
    return cookieValue;
}
var csrftoken = getCookie('csrftoken');

function csrfSafeMethod(method) {
  // these HTTP methods do not require CSRF protection
  return (/^(GET|HEAD|OPTIONS|TRACE)$/.test(method));
}

$.ajaxSetup({
  beforeSend: function (xhr, settings) {
    if (!csrfSafeMethod(settings.type) && !this.crossDomain) {
      xhr.setRequestHeader("X-CSRFToken", csrftoken);
    }
  }
});
```

[django跨站请求伪造保护官方文档](https://docs.djangoproject.com/zh-hans/3.1/ref/csrf/)

#### csrf相关装饰器

[装饰器方法](https://docs.djangoproject.com/zh-hans/3.1/ref/csrf/#module-django.views.decorators.csrf)

[装饰类](https://docs.djangoproject.com/zh-hans/3.1/topics/class-based-views/intro/#id1)

```python
"""
1. 网站整体都不校验csrf 就单单几个视图函数需要校验
2. 网站整体都校验csrf 就单单几个视图函数不需要校验

  csrf_exempt  - 忽视校验
  csrf_protect - 需要校验 
"""

# 解决:Django提供的装饰器

from django.views.decorators.csrf import csrf_exempt, csrf_protect

# FBV直接加对应装饰器即可
# @csrf_exempt
@csrf_protect
def index(request):
    pass

# CBV
csrf_protect - 全局忽视 局部需要校验 -> 使用CBV装饰器的三种方法都可以
csrf_exempt  - 全局校验 局部忽视校验 -> 只可以加给dispath方法 才有效

@method_decorator(csrf_exempt, name='dispath')
class MyIndex(View):
    # @method_decorator(csrf_exempt)
    def dispath(self, request, *args, **kwargs):
        return super().dispath(request, *args, **kwargs)
    
    def post(self, request):
        return HttpResponse('POST')
```

## Auth模块

```python
# 只要是跟用户相关的登录、注册、检验、修改密码、注销、验证用户是否登录 都能用该模块实现
"""
其实我们在创建好一个django项目之后 直接执行数据库迁移命令 会自动生成很多表
  django_session
  auth_user  # 用户表
  
django在启动之后 就可以直接访问admin路由 需要输入用户名和密码 数据参考的就是auh_user表 并且还必须是管理员才能进入

创建超级用户(管理员)
   eatesuperuser
   
依赖于auth_user表完成用户相关的所有功能

使用auth模块 要用就用全套 封装的很好
"""
```

### 常用方法总结

```python
1. 比对用户名和密码是否正确
# 括号内必须同时传入用户名和密码
user_obj = auth.authenticate(request, username=username, password=password)
print(user_obj)  # 用户对象 数据不符合返回None
print(user_obj.username)  # minho
print(user_obj.password)  # 密文

2. 保存用户状态
auth.login(request,user_obj)
# 类似于reqeust.session[key] = user_obj
# 只要执行了该方法 你就可以在任何地方通过request.user就能获取到当前登录的用户对象
# 它自动去django_session里面查找对应的用户对象 给你封装到request.user中

3. 判断当前用户是否登录
request.user.is_authenticated()

4. 获取当前登录用户
# 获取用户对象 未登录：AnonymousUser
request.user

5. 校验用户是否登录 - 装饰器
from django.contrib.auth.decorators import login_required
# 两种使用方式
# 局部配置
@login_required(login_url='/login/')
# 全局配置
LOGIN_URL = '/login/'  # settings.py

6. 比对原密码
request.user.check_password(old_password)  # 自动加密对比密码

7. 修改密码
request.user.set_password(new_password)  # 这步不会影响数据库 仅仅是在修改对象的属性
request.user.save()  # 真正的操作数据库

8. 注销
auth.logout(request)  # request.session表找到登录的对象 类似：request.session.flush

9. 注册
User.objects.create(username=username, password=password)  # 写入数据 不能使用create 密码没有加密处理 后面使用密码比对 request.user.check_password 会出错

# 创建普通用户
User.objects.create_user(username=username, password=password)

# 创建超级用户(了解): 使用代码创建查超级用户 邮箱必填 命令行创建可以不填
User.objects.create_superuser(username=username, password=password, email='1234567@qq.com')
```

### 完整代码示例

```python
from django.shortcuts import render, HttpResponse, redirect
from django.contrib import auth
from django.contrib.auth.decorators import login_required
from django.contrib.auth.models import User

from hmac import compare_digest

"""
使用auth模块 要用就用全套 否则会出一些奇怪的错误
"""
def login(request):
    if request.method == 'POST':
        username = request.POST.get('username')
        password = request.POST.get('password')
        # 去用户表中校验数据
        # 1. 表如何获取
        # 2. 密码如何比对
        user_obj = auth.authenticate(request, username=username, password=password)
        print(user_obj)  # 用户对象 数据不符合返回None
        # print(user_obj.username)
        # print(user_obj.password)
        if user_obj:
            # 保存用户状态
            auth.login(request,user_obj)
            # 类似于reqeust.session[key] = user_obj
            # 只要执行了该方法 你就可以在任何地方通过request.user就能获取到当前登录的用户对象
            # 它自动去django_session里面查找对应的用户对象 给你封装到request.user中
        """
        authenticate
        1. 自动查找auth_user标签
        2. 自动给密码加密再比对
        该方法注意事项：括号内必须同时传入用户名和密码 不能只传用户名(一步直接筛出用户对象 否则还需要通过用户名再去查找密码 再比对)
        """
    return render(request, 'login.html')


# @login_required(login_url='/app03/login/')  # 局部配置: 用户没有登录跳转到login_user后面指定的网址
@login_required
def home(request):
    """用户登录之后才能看home"""
    # 获取用户对象 未登录：AnonymousUser
    print(request.user)
    # 判断用户是否登录
    print(request.user.is_authenticated())
    return HttpResponse('OK')


"""
如果局部和全局都有 局部优先

局部和全局哪个好?
  全局的好处在于 无需重复写代码 但是跳转的页面却很单一
  局部的好处在于 不同的视图函数在用户没有登录的情况下可以跳转到不同的页面
"""

# @login_required(login_url='/app03/login/')
@login_required
def index(request):
    return HttpResponse('index')


@login_required
def set_password(request):
    if request.method == 'POST':
        old_password = request.POST.get('old_password')
        new_password = request.POST.get('new_password')
        confirm_password = request.POST.get('confirm_password')
        # 先检验2次密码是否一致
        if new_password == confirm_password:
            # 校验老密码是否对不对
            is_right = request.user.check_password(old_password)  # 自动加密对比密码
            if is_right:
                # 修改密码
                request.user.set_password(new_password)  # 直到这步不会影响数据库 仅仅是在修改对象的属性
                request.user.save()  # 真正的操作数据库
        return redirect('/app03/login/')

    return render(request, 'set_password.html', locals())


@login_required
def logout(request):
    auth.logout(request)  # request.session表找到登录的对象 类似：request.session.flush
    return redirect('/app03/login/')


def register(request):
    if request.method == 'POST':
        username = request.POST.get('username')
        password = request.POST.get('password')
        confirm_password = request.POST.get('confirm_password')
        # 操作auth_user表 写入数据
        if compare_digest(password, confirm_password):
            # User.objects.create(username=username, password=password)  # 写入数据 不能使用create 密码没有加密处理
            # 创建普通用户
            User.objects.create_user(username=username, password=password)
            # 创建超级用户(了解): 使用代码创建查超级用户 邮箱必填 命令行创建可以不填
            # User.objects.create_superuser(username=username, password=password, email='1234567@qq.com')
    return render(request, 'register.html')
```

### 如何扩展auth_user表

```python
from django.db import models
from django.contrib.auth.models import User, AbstractUser

# 扩展auth_user表的两种方式
# 第一种：一对一关系 不推荐
class UserDetail(models.Model):
    phone = models.BigIntegerField()
    user = models.OneToOneField(to='User')


# 第二种：利用面向对象的继承
class UserInfo(AbstractUser):
    """
    如果继承了AbstractUser
    那么在执行数据库迁移命令的时候 auth_user表就不会再创建出来了
    而UserInfo表中会出现 auth_user所有的字段外加自己扩展的字段
    这么做的好处：在于你能够直接点击你自己的表更加快速的完成操作及扩展

    前提：
       1.在继承之前没有执行过数据库迁移命令
          auth_user没有被创建出来 - 在数据库设计阶段就要明确需不需要扩展该字段
          如果当前库已经创建了 那么你就重新换一个库
       2.继承的类里面 不要覆盖AbstractUser里面的字段名
          表里面有的字段不能动 只能额外扩展字段
       3.需要在配置文件中告诉django 你要用UserInfo替换auth_user ***
          AUTH_USER_MODEL = 'app03.UserInfo'  # 应用名.表名 不要加models
    """
    phone = models.BigIntegerField()
    
"""
你如果自己写表替代了auth_user
那么 auth模块的功能还是照常使用 参考
的表也由原来的auth_user变成了UserInfo
"""
```

## Django版本区别

```python
1.x 2.x 3.x区别
2.x 3.x 差不多
```

### 路由层

```python
"""
1. django 1.x 路由层使用的是url方法
   django 2.x 3.x 路由层使用的是path方法
    - url() 第一个参数支持正则
    - path() 第一个参数是不支持正则的 写什么就匹配什么

如果你不习惯使用path 也提供了另一个方法 re_path
from django.urls import re_path
from django.conf.urls import url  # 可以继续使用url 不推荐

2.x 3.x 里面的re_path() 等价于1.x里面的url

2. 虽然path不支持正则 但是它的内部支持5种转换器
"""
```

#### path转换器

[博客参考](https://www.cnblogs.com/xiaoyuanqujing/articles/11642628.html)

- **示例**

```python
# 将第二个路由里面的内容先转成整型 然后以关键字的形式传递给后面的视图函数
path('index/<int:id>/', views.index)
```

- **5种转换器**

```python
# 5种转换器
str    匹配除了路径分隔符（/）之外的非空字符串，这是默认的形式
int    匹配正整数，包含0
slug   匹配字母、数字以及横杠、下划线组成的字符串
uuid   匹配格式化的uuid，如 075194d3-6885-417e-a8a8-6c931e272f00
path   匹配任何非空字符串 包含了路径分隔符（/）（不能用？）
```

- **自定义转换器**

```python
# 除了有默认的5个转换器之外 还支持自定义转换器
class MonthConverter:
    regex='\d{2}' # 属性名必须为regex
    
    def to_python(self, value):
        return int(value)
    
    def to_url(self, value):
        # 匹配的regex是两个数字，返回的结果也必须是两个数字
        return value 
```

- **自动转换器使用**

```python
from django.urls import path,register_converter
from app01.path_converts import MonthConverter

# 注册
register_converter(MonthConverter,'mon')  

# 使用
from app01 import views
urlpatterns = [
    path('articles/<int:year>/<mon:month>/<slug:other>/', views.article_detail, name='aaa'),
]
```

### 模型层

```python
"""
模型层里面1.x外键默认都是级联更新删除的
但是到了2.x和3.x中 需要你自己手动配置on_delete参数
"""

# 1.x
publish = models.ForeignKey(to='Publish')

# 2.x 3.x
# 具体参数参考官网文档
publish = models.ForeignKey(to='Publish', on_delete='xxx')
```

**on_delete参数补充**

[ForeignKey.on_delete](https://docs.djangoproject.com/zh-hans/3.1/ref/models/fields/#django.db.models.ForeignKey.on_delete)

```python
on_delete=models.CASCADE,     # 删除关联数据,与之关联也删除
on_delete=models.DO_NOTHING,  # 删除关联数据,什么也不做
on_delete=models.PROTECT,     # 删除关联数据,引发错误ProtectedError
on_delete=models.SET_NULL,    # 删除关联数据,与之关联的值设置为null（前提该字段需要设置为可空,一对一同理）
on_delete=models.SET_DEFAULT, # 删除关联数据,与之关联的值设置为默认值（前提FK字段需要设置默认值,一对一同理）
on_delete=models.SET(),       # 删除之后执行一个函数
on_delete=RESTRICT,           # New in Django 3.1.
```
