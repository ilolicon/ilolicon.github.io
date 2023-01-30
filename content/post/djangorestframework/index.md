---
title: DjangoRESTframework
description: Django REST framework is a powerful and flexible toolkit for building Web APIs.
date: 2021-02-22 23:00:00
image: drf.png
categories: 
- PYTHON
tags:
- Python
- Web
- RESTful
- API
---

# RESTFramework

[RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)

[DjangoRESTframework官网](https://www.django-rest-framework.org/)

[DjangoRESTframework 中文教程](https://q1mi.github.io/Django-REST-framework-documentation)

## RESTful API

API接口： 通过网络 规定了前后台信息交互规则的url链接 也就是前后台信息交互的**媒介**

### 简介

```python
REST 全称：Representational State Transfer 中文意思的表述(通常译为：表征性状态转移)

RESTful是一种定义Web API接口的设计风格 尤其适用于前后端分离的应用模式中
这种风格的理念认为：后端开发任务就是提供数据的 对外提供的是数据资源的访问接口 所以在定义接口时 客户端访问的URL路径就表示这种要操作的数据资源

事实上 我们可以使用任何一个框架都可以实现符合restful规范的接口
```

### 规范

**restful**10条规范

1.**协议**

```python
URL链接一般都采用https协议进行传输 注：采用https协议 可以提高数据交互过程中的安全性
```

2.**接口特征性表现(一看就知道是API接口)**

用api关键字表示接口url

```python
- https://api.example.com
- https://example.org/api/      
# 如果确定API很简单 不会有进一步扩展 可以考虑放在主域名之下
```

3.**多版本共存**

应该将API的版本放入URL(一种资源有多版本请况下)

```python
- https://api.example.com/v1/
- https://api.example.com/v2/

# 另一种做法是 将版本号放在HTTP头信息中 但不如放入URL方便和直观
```

4.**数据即是资源 均使用名字(复数) - 重要**

接口一般都是完成前后台数据的交互 交互的数据我们称之为资源

```python
- https://api.example.com/v1/zoos
- https://api.example.com/v1/animals
- https://api.example.com/v1/employees
    
注：一般提倡用资源的复数形式 在URL链接中不要出现操作资源的动词
  
# 特殊的接口可以出现动词 因为这些接口一般没有一个明确的资源 或是动词就是接口的核心含义
  - https://api.baidu.com/place/search
  - https://api.baidu.com/login
```

5.**资源操作由请求方式决定(method) - 重要**

操作资源一般都会设计到增删改查 我们提供请求方式来标识这些动作

```python
- https://api.baidu.com/books  - GET请求：获取所有书
- https://api.baidu.com/books/1  GET请求：获取主键为1的书
- https://api.baidu.com/books    POST请求：新增一本书
- https://api.baidu.com/books/1  PUT请求：整体修改主键为1的书
- https://api.baidu.com/books/1  PATCH请求：局部修改主键为1的书
- https://api.baidu.com/books    DELETE请求：删除主键为1的书
```

常用的HTTP动词有下面五个(括号里面是对应的SQL命令)

```python
- GET(SELECT):    从服务器取出资源(一项或多项资源)
- POST(CREATE):   在服务器新建一个资源
- PUT(UPDATE):    在服务器更新资源(客户端提供改变后的完整资源)
- PATCH(UPDATE):  在服务器更新资源(客户端提供改变的属性)
- DELETE(DELETE): 从服务器删除资源
```

还有两个不常用的HTTP动词

```python
- HEAD: 获取资源的元数据
- OPTIONS: 获取信息 关于资源的哪些属性是客户端可以改变的
```

6.**过滤信息**

```python
# 如果记录数量很多 服务器不可能都将它们返回给用户
# API应该提供参数 过滤返回结果
  - ?limit=10   指定返回记录的数量
  - ?offset=10  指定返回记录的开始位置
  - ?page=2&per_page=100  指定第几页，以及每页的记录数
  - ?sortby=name&order=asc  指定返回结果按照哪个属性排序，以及排序顺序
  - ?animal_type_id=1  指定筛选条件
```

7.**状态码**

[状态码完全列表](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)

服务器向用户返回的状态码和提示信息

```python
# 常见的有以下一些(方括号中是该状态码对应的HTTP动词)
200 OK - [GET]：服务器成功返回用户请求的数据 该操作是幂等的（Idempotent）
201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功
202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
204 NO CONTENT - [DELETE]：用户删除数据成功

301 Moved Permanently 永久重定向
302 Found 暂时重定向

400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误 服务器没有进行新建或修改数据的操作 该操作是幂等的
401 Unauthorized - [*]：表示用户没有权限(令牌、用户名、密码错误)
403 Forbidden - [*] 表示用户得到授权(与401错误相对) 但是访问是被禁止的
404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录 服务器没有进行操作 该操作是幂等的
406 Not Acceptable - [GET]：用户请求的格式不可(比如用户请求JSON格式 但是只有XML格式)
410 Gone -[GET]：用户请求的资源被永久删除 且不会再得到的
422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时 发生一个验证错误

500 INTERNAL SERVER ERROR - [*]：服务器发生错误 用户将无法判断发出的请求是否成功
```

8.**错误处理**

```python
# 如果状态码是4xx 就应该向用户返回出错信息
# 一般来说 返回的信息中将error作为键名 出错信息作为键值即可
{
    error: "Invalid API key"
}
```

9.**返回结果**

针对不同操作 服务器向用户返回的结果应该符合以下规范

```python
GET    /collection： 返回资源对象的列表(数组)
GET    /collection/resource： 返回单个资源对象
POST   /collection： 返回新生成的资源对象
PUT    /collection/resource： 返回完整的资源对象
PATCH  /collection/resource： 返回完整的资源对象
DELETE /collection/resource： 返回一个空文档
```

10.**Hypermedia API**

```python
RESTful API最好做到Hypermedia 
即返回结果中提供链接 连向其他API方法 使得用户不查文档 也知道下一步应该做什么

Hypermedia API的设计被称为HATEOAS
Github的API就是这种设计 访问api.github.com会得到一个所有可用API的网址列表
{
  "current_user_url": "https://api.github.com/user",
  "authorizations_url": "https://api.github.com/authorizations",
  // ...
}
```

其他

```python
1. API的身份认证应该使用OAuth 2.0框架

2. 服务器返回的数据格式 应该尽量使用JSON 避免使用XML
```

## DRF初识

### 安装

```python
pip install djangorestframework==3.12.2
```

### CBV源码

```python
# ModelViewSet继承自View(django远程的View)
# ModelViewSet -> APIView -> View
```

#### View

- CBV简单实现

```python
# urls.py
urlpatterns = [
    path('books/', views.Books.as_view()),
]

# views.py
from django.views import View
from django.http import JsonResponse

class Books(View):
    def get(self, request):
        return JsonResponse({'c++': 111})
    
"""
127.0.0.1:8000/books/  GET  通过CBV实现GET请求方法
"""
```

- **as_view()**

```python
"""
源码解读：切入口：as_view()
本质还是FBV: 
    views.Books.as_view() ->  as_view()返回一个函数对象 -> views.Books.view
    
如果请求过来 路径匹配上 会执行 views.Books.view的 view内存地址指向的函数对象 并把request参数传入(当次请求的request对象) -> view(request) 
"""

# as_view()本质
@classonlymethod
def as_view(cls, **initkwargs):
    def view(request, *args, **kwargs):
        self = cls(**initkwargs)
        # cls是我们自己写的类 调用的时候自动注入(谁调用 注入谁)
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

# CBV的精髓 dispath
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

#### APIView

- **CBV简单实现**

```python
# urls.py
# 也是as_view() 只不过被APIView(继承View)重写
path('books_apiview/', views.BooksAPIView.as_view())

# views.py
from rest_framework.views import APIView

class BooksAPIView(APIView):
    def get(self, request):
        return JsonResponse({'APIView': 666})
```

- **as_view()**

```python
"""
源码解读：切入口还是 as_view() -> APIView的as_view() -> view 函数内存地址
"""
@classmethod  # 类的绑定方法
def as_view(cls, **initkwargs):
    view = super().as_view(**initkwargs)  # 调用父类的as_view方法
    view.cls = cls
    view.initkwargs = initkwargs
    # 继承APIView的视图类会禁用csrf认证(drf会有自己的认证)
    # 添加装饰器的另一种方法 @方式是语法糖 本质就是传入一个被装饰函数给装饰器函数当作实参
    return csrf_exempt(view)

# 使用csrf装饰器 URL视图还可以这么写 
path('test/', csrf_exempt(as_view))

"""
发起请求 -> 路由匹配成功 -> view(request) -> 调用dispath() -> mro属性查找顺序 self.dispath会执行到APIView的dispath方法 而不再走View类的dispath -> 把请求方法转为小写 -> 通过反射去对象中查找 有没有改方法定义的属性 有则传入request并执行
"""

# APIView的disapth
def dispatch(self, request, *args, **kwargs):
    self.args = args
    self.kwargs = kwargs
    # request参数是当次请求的request对象
    # 请求过来 wsgi注入一个envrion字段 django把它包装成一个request对象
    
    # 然后重新赋值的request 是初始化处理后的一个新的Request对象
    request = self.initialize_request(request, *args, **kwargs)
    
    # 现在视图函数拿到的request 已经不是django原生给我们封装的request
    # 而是drf自己定义的request对象(mro) 以后视图函数再使用的request对象 就是这个新的drf定义的对象
    self.request = request  # 将新的request对象赋值给self.request 你视图函数里面 其他的方法 也可以从这个里面获取相关数据
    self.headers = self.default_response_headers  # deprecate?

    try:
        # 三大认证模块
        self.initial(request, *args, **kwargs)

        # Get the appropriate handler method
        if request.method.lower() in self.http_method_names:
            handler = getattr(self, request.method.lower(),
                              self.http_method_not_allowed)
        else:
            handler = self.http_method_not_allowed
   # 响应模块
        response = handler(request, *args, **kwargs)

    except Exception as exc:
       # 异常模块
        response = self.handle_exception(exc)
 
    # 渲染模块
    self.response = self.finalize_response(request, response, *args, **kwargs)
    return self.response

# 封装drf自己的request对象
def initialize_request(self, request, *args, **kwargs):
    parser_context = self.get_parser_context(request)
    # django原来的request 被封装到Request里面 
    # self._request 原生的request
    # self.request  drf封装的request
    
    # 返回DRF封装的Request类实例化之后的对象
    return Request(
        request,    # 原生的request对象
        # 获取解析类
        parsers=self.get_parsers(),
        authenticators=self.get_authenticators(),
        negotiator=self.get_content_negotiator(),
        parser_context=parser_context
    ) 

"""
针对这个赋值：
self.request = request  # 将新的request对象赋值给self.request 你视图函数里面 其他的方法 也可以从这个里面获取相关数据(就是drf封装的当次请求的request)

class MyCBV(VPIView):
 def get(self, request):
     ..
 def foo(self):
     print(self.request)  # 视图函数中 其他方法 不用传参 也可以直接使用该对象     
"""
```

- **Request**

```python
"""
from rest_framework.request import Request

只要继承了APIView 视图类中的request对象 都是新的
也就是上面导入路径这个Request的对象(实例)

使用使用新的request对象 就像使用之前的request是一模一样的(因为重写了__getattr__方法)

def __getattr__(self, attr):
    try:
        return getattr(self._request, attr)
    except AttributeError:
        return self.__getattribute__(attr)
"""

class Request:
    """
    Wrapper allowing to enhance a standard `HttpRequest` instance.

    Kwargs:
        - request(HttpRequest). The original request instance.
        - parsers(list/tuple). The parsers to use for parsing the
          request content.
        - authenticators(list/tuple). The authenticators used to try
          authenticating the request's user.
    """

    def __init__(self, request, parsers=None, authenticators=None,
                 negotiator=None, parser_context=None):
        assert isinstance(request, HttpRequest), (
            'The `request` argument must be an instance of '
            '`django.http.HttpRequest`, not `{}.{}`.'
            .format(request.__class__.__module__, request.__class__.__name__)
        )
        # self._request 原生的request
        self._request = request
        self._data = Empty
  ...
        
        
# restframework封装的request对象
"""
django封装的request对象没有.data {}
它是一个字典 
  - post请求不管使用什么编码(form/urlencoded/json) 传过来的数据 都在request.data
  - get请求
"""
request.data    
```

- **取POST请求数据**

|            | django原生request(request.POST) | drf封装后的Request对象(request.data) |
| ---------- | ------------------------------- | ------------------------------------ |
| form       | 有数据 QueryDict                | 有数据 QueryDict                     |
| urlencoded | 有数据 QueryDict                | 有数据 QueryDict                     |
| json       | 无数据                          | 有数据 普通字典                      |

- **取GET请求url中的数据**

```python
# drf Request类方法
# self._request.GET = request.query_params
# django原来取： request.GET
# drf取： request.query_params

@property
def query_params(self):
    return self._request.GET
```

- **APIView的initial方法**

```python
# dispath中调用 initial方法
# request参数 已经是新的request
def initial(self, request, *args, **kwargs):
 ...
    # Ensure that the incoming request is permitted
    """
    perform_authentication
    认证组件：检验用户 - 游客、合法用户、非法用户
      游客： 代表检验通过 直接进入下一步校验(权限检验)
      合法用户：代表校验通过 将用户存储再request.user中 再进入下一步校验(权限校验)
      非法用户：代表校验失败 抛出异常 返回403权限异常结果
    """
    self.perform_authentication(request)
    """
    check_permissions
    权限组件：校验用户权限 - 必须登录、所有用户、登录读写游客只读、自定义用户角色
      认证通过：可以进入下一步校验(频率认证)
      认证失败：抛出异常 返回403权限异常结构
    """
    self.check_permissions(request)
    
    """
    check_throttles
    频率组件：限制视图接口被访问的频率次数 - 限制的条件(IP/id/唯一键)、频率周期时间(s/m/h)、频率的次数(3/s)
    没有达到限制：正常访问接口
    达到限制：限制时间之内不能访问 限制时间达到后 可以重新访问
    """
    self.check_throttles(request)
```

- **补充**

```python
def add(x, y):
    return x + y

# 由于Python中一切皆对象 所以都可以设置属性 取出属性
# Python源码中使用非常多
add.xyz = 'xyz111'  

print(add(5, 5))
print(add.xyz)
```

## 序列化器

[drf-Serializer](https://www.django-rest-framework.org/api-guide/serializers/)

[drf-Serializer-fileds](https://www.django-rest-framework.org/api-guide/fields/)

[drf-Serializer-relations](https://www.django-rest-framework.org/api-guide/relations/)

- Serializer

```python
"""
序列化：序列化器会把模型对象转换为字典 经过response以后变成json字符串 
       model对象 -> json
       
反序列化：把客户端发送过来的数据 经过request以后变成字典(框架封装) 序列化器可以把字典转成模型
       json -> model对象
       反序列化另一个作用：完成数据的校验功能(类似forms组件)  
"""

1. Serializer类 必须写一个类继承它 想序列化什么字段就在里面写字段(source可以修改序列化出来的字段名 获取取属性 .跨表)

2. 序列化queryset(列表)对象和真正(单个)的对象 many=True的作用 instance=要序列化的对象(模型类对象 也就是模型类实例)

3. 反序列化 instance=要序列化的对象 data=request.data(新增没有instance参数)

4. 字段验证 序列化类中 给字段加属性(字段参数) 局部和全局钩子函数|字段参数的validaores=[] 
   视图函数中调用 序列化对象.is_valid(raise_exception=True)进行验证 raise_exception参数默认False 如果为True 验证不通过直接抛出异常
    
5. 反序列化数据保存
   5.1 修改保存 -> 调用序列化对象的.save() -> 触发序列化类的.update方法
   5.2 新增保存 -> 调用序列化对象的.save() -> 触发序列化类的.create方法

   备注：
   如果自定义序列化类不是继承的ModelSerializer 需要重写 .update方法和.create方法(可以很复杂)
   如果反序列的数据中有需要保存到第三张表的 需要在重写的方法额外处理
    
6. ModelSerializer 跟Model做了对应 写法固定 字段参数直接用 extra_kwargs
   其余用法和继承Serializer一摸一样(包括全局/局部钩子函数)
    
7. many=True 可以序列化多条的原因

8. 接口 鸭子类型
           
```

### 序列化

#### 字段类型

[SerializerFields](https://www.django-rest-framework.org/api-guide/fields/)

[序列化器字段](https://www.w3cschool.cn/lxraw/lxraw-u4kq35ot.html)

```python
# 完整字段见上面参考链接
CharField
IntergerField
DateField
...
```

#### 字段参数

```python
# 检验功能 非常类似forms组件
  1. 自带字段选项校验(更多参考字段链接详细内容)
     - read_only
     - required  默认为True
     - max_length 
     - validators
         验证器功能列表 应将其应用于输入字段输入 并引发验证错误或简单地返回
       ...        
  2. 局部钩子函数
  3. 全局钩子函数
    
# 通用字段参数
read_only        表明该字段仅用于序列化输出 默认False
write_only       表明该字段仅用于反序列化输入 默认False
required         表明该字段在反序列化时必须输入 默认True
default          反序列化时使用的默认值
allow_null       表明该字段是否允许传入None 默认Flase
validators       该字段使用的验证器
error_messages   包含错误编号与错误信息的字典
label            用于HTML展示API页面时 显示的字段名称
help_text        用于HTML展示API页面时 显示的字段帮助提示信息
```

### 反序列化

#### 修改数据

```python
"""
PUT 127.0.0.1:8000/books/1 修改书籍主键为1的书籍信息
  1. 写一个序列化的类 继承Serializer类
  2. 在类中写想要反序列化的字段
  3. 在视图类中使用 导入自定义序列化类 -> 实例化得到的序列化类对象(把要修改的对象 和 修改的数据传入)
      # instance=要修改的对象
      # data=修改的数据
      serializer = BookSerializer(instance=book, data=request.data)
   
   4. 数据校验
      # raise_exception参数默认为False
      # 如果为True 验证不通过直接抛出异常
      serializer.is_valid(raise_exception=True)
      
      4.1 如果校验通过 就保存
          serializer.save()  # 是序列化器的save()方法
      4.2 如果校验不通过
          返回错误信息
          
   5. 如果字段的校验规则不够 可以自己写钩子函数(局部和全局 类似forms)
"""

class BooksDetailView(APIView):
    def put(self, request, pk):
        response_msg = {'status':100, 'msg':'成功'}
        # 找到这个对象
        book = Book.objects.filter(pk=pk).first()
        # 直接用request.data的数据来修改原来的对象 -> 反序列化
        serializer = BookSerializer(book, request.data)
        # 一定要数据验证(类似form表单验证)
        if serializer.is_valid():
            # 直接调用报错 需要重写update方法 接口规范了子类的行为 鸭子类型
            # NotImplementedError: `update()` must be implemented.
            serializer.save()  # 验证通过则返回
            response_msg['data'] = serializer.data
            # return Response(serializer.data)
        else:
            response_msg['status'] = 1001
            response_msg['msg'] = '数据校验失败'
            response_msg['data'] = serializer.errors
        return Response(response_msg)
    
# 直接继承Serializer没有update方法 需要重写
class BookSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    name = serializers.CharField(max_length=16, min_length=4)
    price = serializers.DecimalField(max_digits=5, decimal_places=2)
    author = serializers.CharField()
    publish = serializers.CharField()

    def update(self, instance, validated_data):
        # instance是book这个对象
        # validated_data是校验后的数据
        instance.name = validated_data.get('name')
        instance.price = validated_data.get('price')
        instance.author = validated_data.get('author')
        instance.publish = validated_data.get('publish')
        instance.save()  # django ORM提供的
        return instance
```

#### 数据校验钩子函数

```python
先自己字段参数验证 
-> 如果该字段有局部钩子函数 走该函数验证该字段
-> 所有字段按照该顺序验证完成 看是否有全局钩子验证函数
-> 全局钩子函数验证(此时数据已经经过第一次校验 validated_data)
-> 验证完成
```

- **局部钩子**

```python
from rest_framework import serializers
from rest_framework.exceptions import ValidationError

class BookSerializer(serializers.Serializer):
    ...

    def validate_price(self, data):
        """
        validate_字段名(通过反射获取该方法) 接收一个参数
        :param data: 就是price 可以对任意单一字段自定义校验
        :return: data
        """
        # print(data)
        # print(type(data))
        if data > 10:
            return data
        # 校验失败 抛出异常
        raise ValidationError('价格太低')
```

输出示例

```json
{
    "status": 1001,
    "msg": "数据校验失败",
    "data": {
        "price": [
            "价格太低"
        ]
    }
}
```

- **全局钩子**

```python
from rest_framework import serializers
from rest_framework.exceptions import ValidationError

class BookSerializer(serializers.Serializer):
    ...
    
    def validate(self, attrs):
        """
        全局钩子：校验多个字段
        :param attrs: 校验通过的数据(validated_data)
        :return: attrs 
        """
        print(attrs)
        author = attrs.get('author')
        publish = attrs.get('publish')
        if author == publish:
            raise ValidationError('作者名字和出版社一致')
        return attrs
```

输出示例

```json
{
    "status": 1001,
    "msg": "数据校验失败",
    "data": {
        "non_field_errors": [
            "作者名字和出版社一致"
        ]
    }
}
```

- **参数validators(非钩子函数 字段选项验证器 较少使用)**

```python
def check_author(data):
    if data.startswith('sb'):
        raise ValidationError('作者名字不能以sb开头')
    return data

class BookSerializer(serializers.Serializer):
    ...
    # 使用字段参数：验证器
    author = serializers.CharField(validators=[check_author])

"""
加上上面的钩子函数和自带的字段验证 总共有三种验证数据的方式：
  1. 字段参数自带的验证参数们
  2. 局部钩子/全局钩子函数
  3. 参数：validators 验证器
  
抛出无效数据的异常
  .is_valid()方法使用可选的raise_exception标志 如果存在验证错误将会抛出一个serializers.ValidationError异常
  这些异常由REST framework提供的默认异常处理程序自动处理 
  默认情况下将返回HTTP 400 Bad Request响应
"""

# 如果数据无效就返回400响应
serializer.is_valid(raise_exception=True)
```

输出示例

```json
{
    "status": 100,
    "msg": "数据校验失败",
    "data": {
        "author": [
            "作者名字不能以sb开头"
        ]
    }
}
```

#### 序列化反序列化字段处理

```python
"""
问题：我们序列化的时候和反序列化的时候 使用的字段数目不全部一致
     比如主键id 序列化展示 反序列化不应该输入

解决：
  1. 写两个serializer类 一个用于序列化 一个用于反序列化 -> 冗余度非常高 麻烦
  2. 字段参数解决
      read_only    表明该字段仅用于序列化输出 默认False
      write_only   表名该字段仅用于反序列化输入 默认False
"""
class BookSerializer(serializers.Serializer):
    """
    read_only=True
      - 序列化输出 有该字段
      - 反序列化传入数据 不需要传该字段 错误的输入所有的read_only字段都将被忽略
    write_only
      - 序列化输出 没有该字段
      - 反序列化传入数据 必须传入该字段 否则校验失败
    """
    id = serializers.IntegerField(read_only=True)
    publish = serializers.CharField(max_length=64, min_length=2, write_only=True)
```

#### Choices字段显示名称处理

[DRF处理模型中的选项字段](https://segmentfault.com/a/1190000017572896)

```python
"""
drf反序列化的时候 无法取到get_fieldname_display()返回的可读字符串
目前会三种解决方法：
  - source参数
  - serializers.SerializerMethodField() 字段
  - 重写序列化器的 to_representation() 方法
  
注意：前面2种方法 等于重新定义了一个新的字段 此时无法反序列化
"""

# 1. source参数
gender = serializers.CharField(source='get_gender_display')  # 该字段变只读

# 2. SerializerMethodField字段

# 3. 重写to_representation方法 不影响反序列化
class NodeSerializer(serializers.ModelSerializer):
    .../
    
    def to_representation(self, instance):
        data = super().to_representation(instance)
        data.update(node_type=instance.get_node_type_display())
        data.update(product_line=instance.get_product_line_display())
        data.update(isp=instance.get_isp_display())
        return data
```

### 其余API操作

#### 查询所有数据

```python
# URL
path('books/', views.Books.as_view())

# View
class Books(APIView):
    def get(self, request):
        books = Book.objects.all()
        # 序列化多条 需要加参数：many=True
        serializer = BookModelSerializer(instance=books, many=True)
        return Response(serializer.data)
```

#### 新增数据

```python
# URL
path('books/', views.Books.as_view())

# models.py
class Books(APIView):
    def post(self, repost):
        # 修改才有instance 新增没有instance 只有data
        # 必须关键字传参 位置传参会给到第一个位置参数:instance
        book_ser = BookModelSerializer(data=repost.data)
        # 校验字段
        if book_ser.is_valid():
            book_ser.save()
            return Response(book_ser.data)
        return Response(book_ser.errors)
    
# serializers.py 需要重写create()方法
class BookSerializer(serializers.Serializer):
    def create(self, validated_data):
        """
        :param validated_data: dict正好可以**解构
        :return: 
        """
        # 如果validated_data的数据 不是全部需要 
        # 就需要拿出来额外处理 不能直接双**解构
        return Book.objects.create(**validated_data)
```

#### 删除数据

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from .models import Book

# 删除无需序列化器
class BooksDetail(APIView):
    def delete(self, request, pk):
        """删除一个数据"""
        Book.objects.filter(pk=pk).delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

### 自定义响应内容

```python
# utils.py
class MyResponse:
    def __init__(self):
        self.status = 100
        self.mgs = 'success'

    @property
    def get_dict(self):
        return self.__dict__

# 使用的时候 实例化再修改属性即可
if __name__ == '__main__':
    res = MyResponse()
    res.status = 101
    res.msg = '数据校验失败'
    res.data = {'name': 'Minho'}
    print(res.get_dict)
```

### 模型类序列化器

```python
class BookModelSerializer(serializers.ModelSerializer):
    # ModelSerializer 也可以使用source参数 用法一样
    pce = serializers.CharField(source='publish.price')
    class Meta:
        model = Book                  # 对应models.py中的表模型(要序列化哪个表的数据)
        # fields = '__all__'          # __all__标识序列化所有字段
        # fields = ['name', 'price']  # 只序列化指定字段
        exclude = ['name']            # 跟fields不能都写 写哪个字段标识排除哪个字段
        read_only_fields = ['price']
        extra_kwargs = {
            'author': {'write_only': True}
        }

"""
write_only_fields 弃用 -> 使用extra_kwargs解决
  可以通过使用extra_kwargs选项快捷地在字段上指定任意附加的关键字参数
  类似于：Serializer的 name = serializers.CharField(max_length=16, min_length=4)
  
示例：
  extra_kwargs = {'password': {'write_only': True},
                  'author': {'write_only': True},}
                  
备注：
  继承ModelSerializer之后 不用再自己重写 .update() 和 .create()方法
  其他使用方式和继承Serializer 一摸一样
"""        
```

输出示例

```json
[
    {
        "nid": 1,
        "pce": "129.11",
        "name": "红楼梦",
        "price": "129.11",
        "publish": "北京出版社"
    },
    {
        "nid": 3,
        "pce": "192.45",
        "name": "python",
        "price": "192.45",
        "publish": "xx出版社"
    },
]
```

### many=True源码分析

```python
# 序列化多条数据的时候 需要传 many=True
class Books(APIView):
    def get(self, request):
        # 序列化单条
        book = Book.objects.all()
        books_one_ser = BookModelSerializer(instance=book)
        # 序列化多条
        books = Book.objects.all()
        books_ser = BookModelSerializer(instance=books, many=True)
        print(type(books_one_ser))
        print(type(books_ser))
        return Response(books_ser.data)
 
# many=True时 返回的结果跟不传 不是同一个类的对象
单条类型： <class 'drftutorial.book.serializer.BookModelSerializer'>
多条类型： <class 'rest_framework.serializers.ListSerializer'> 

"""
对象的生成：先调用类的__new__方法 生成空对象
实例化：类名(参数) 调用类的__init__()方法

类的__new__方法 控制对象的生成 -> 由此猜测BookModelSerializer类的__new__方法做了处理 根据是否有many=True参数 生成不同的类对象
"""
class BaseSerializer(Field):
    ...
    def __new__(cls, *args, **kwargs):
        # We override this method in order to automatically create
        # `ListSerializer` classes instead when `many=True` is set.
        if kwargs.pop('many', False):
            return cls.many_init(*args, **kwargs)
        # 没有传many=True 正常实例化
        return super().__new__(cls, *args, **kwargs)
    
    @classmethod
    def many_init(cls, *args, **kwargs):
  ...
        """
        如果传入many=True 调用该方法 生成新得类对象 里面就是一个个得Serializer对象
        """
        return list_serializer_class(*args, **list_kwargs)
```

### Serializer高级用法

[关联字段](https://www.django-rest-framework.org/api-guide/relations/)

#### 一对多关系字段

- **source参数**

```python
source参数就是指定序列化对象的属性
该属性可以是模型类的字段属性
也可以是该模型类的方法(类方法本质也是类属性 本质一样）
            
"""
source的使用 下面三个功能：
  1. 可以改字段名字  xxx = serializers.CharField(max_length=32, source='title')
  2. 可以.跨表取属性 publish = serializers.CharField(source='publish.email')
  3. 可以执行方法 publish_date = serializers.DateTimeField(source='test')  # test不是model类对应字段 而是自定义函数
"""
```

- **source=对象属性**

```python
"""
一对多关系字段：
  1. 模型类__str__魔术方法
     publish = serializers.CharField() -> book.publish -> 获取publish的__str__方法返回值
  
  2. source参数用法
     publish = serializers.CharField(source='publish.email')
     source参数可以直接取模型类的字段
       2.1 更换序列化器显示的字段名称(隐藏真正的数据库字段) 用source与实际的数据库模块字段对应 
       xxx = serializers.CharField(max_length=32, source='title')       
       2.2 一对多关系中 可以取到关联关系表中的其他字段(跨表)
       publish = serializers.CharField(source='publish.email')
"""

from rest_framework import serializers

class BookSerializer(serializers.Serializer):
    # 这部分字段相当于隐藏了模型前缀 相当于取book得属性(包括方法)
    # book.xxx | book.price | book.publish_data...
    xxx = serializers.CharField(max_length=32, source='title')
    price = serializers.DecimalField(max_digits=8, decimal_places=2)
    publish_date = serializers.DateTimeField()
    # book.publish 外键关联字段 直接写 显示结果会显示__str__方法的结果
    publish = serializers.CharField(source='publish.email')
    # book.authors None -> book.authors.all()   
```

- **source=对象方法**

```python
# serializers.py
from rest_framework import serializers

class BookSerializer(serializers.Serializer):
    xxx = serializers.CharField(max_length=32, source='title')
    publish_date = serializers.DateTimeField(source='test')
    
# models.py
class Book(models.Model):
    title = models.CharField(max_length=32)
    
    def test(self):
        return 'book.test method'

"""
source指定的方法的return内容就是序列化的内容
"""
```

输出示例

```json
{
    "xxx": "红楼梦",
    "publish_date": "book.test method",
}
```

#### 多对多关系字段

- **SerializerMethodField**

```python
from rest_framework import serializers

class BookSerializer(serializers.Serializer):
 ...
    authors = serializers.SerializerMethodField()

    def get_authors(self, obj):
        # obj: book对象
        authors = obj.authors.all()  # 跨表拿出所有作者
        return [{'name': author.name, 'age': author.age} for author in authors]

"""
serializer通过字段取出多对多关系表数据：
  SerializerMethodField(method_name=None)
  这是一个只读字段 它通过在附加的序列化器类上调用一个方法来获取其值 它可以用于将任何类型的数据添加到对象的序列化表示中
  该字段会绑定一个方法 如果没有指定method_name  则默认为：get_<field_name>(self, obj): pass
  该方法返回什么 序列化的字段就显示什么内容
"""
```

输出示例

```python
{
    "xxx": "红楼梦",
    "price": "123.45",
    "publish_date": "2021-01-05T20:43:26Z",
    "publish": "南京出版社",
    "authors": [
        {
            "name": "minho",
            "age": 25
        },
        {
            "name": "kimi",
            "age": 46
        }
    ]
}
```

## 请求与响应

[MDN HTTP教程](https://developer.mozilla.org/zh-CN/docs/Web/HTTP)

[django request-response](https://docs.djangoproject.com/zh-hans/3.1/ref/request-response/)

### Request

[drf-request](https://www.django-rest-framework.org/api-guide/requests/)

[assert断言 补充](https://www.runoob.com/python3/python3-assert.html)

```python
# Rqeust类并没有继承django的
# CBV源码有部分解析
# 位置：from rest_framework.request import Request

# Request部分关键源码
class Request:
    """
    Wrapper allowing to enhance a standard `HttpRequest` instance.

    Kwargs:
        - request(HttpRequest). The original request instance.
        - parsers(list/tuple). The parsers to use for parsing the
          request content.
        - authenticators(list/tuple). The authenticators used to try
          authenticating the request's user.
    """
    def __init__(self, request, parsers=None, authenticators=None,
                 negotiator=None, parser_context=None):
        assert isinstance(request, HttpRequest), (
            'The `request` argument must be an instance of '
            '`django.http.HttpRequest`, not `{}.{}`.'
            .format(request.__class__.__module__, request.__class__.__name__)
        )
        # 赋值django封装的request
        self._request = request
      
    # 获取属性处理
    # 如果自己没有的 问django原生封装的request要 这个时候是self._request
    def __getattr__(self, attr):
        """
        If an attribute does not exist on this instance, then we also attempt
        to proxy it to the underlying HttpRequest object.
        """
        try:
            return getattr(self._request, attr)
        except AttributeError:
            return self.__getattribute__(attr)
        
    # request.data
    @property
    def data(self):
        if not _hasattr(self, '_full_data'):
            self._load_data_and_files()
        return self._full_data
        
# __getattribute__
def __getattribute__(self, *args, **kwargs): # real signature unknown
    """ Return getattr(self, name). """
    pass
```

#### request.data

- 取POST请求数据

```python
请求对象.data -> request.data
前端以三种编码方式传入的数据 都可以取出来(结果可能是QueryDict 或 字典)

"""
实现：
  from/urlencode 直接去取django原生封装的request取
  json 去request.boby取出来 然后反序列化返回字典
  
  json模块是否支持反序列化bytes格式
    3.6以后才支持 否则要decode成字符串在反序列化
"""
```

|            | django原生request(request.POST) | drf封装后的Request对象(request.data) |
| ---------- | ------------------------------- | ------------------------------------ |
| form       | 有数据 QueryDict                | 有数据 QueryDict                     |
| urlencoded | 有数据 QueryDict                | 有数据 QueryDict                     |
| json       | 无数据                          | 有数据 普通字典                      |

#### requset.query_params

- 取GET请求url中的数据

```python
# drf Request类方法
# self._request.GET = request.query_params
# django原来取： request.GET
# drf取： request.query_params

@property
def query_params(self):
    return self._request.GET
```

### Response

```python
Response 是一个类
要使用 就要实例化(需要传参)

Response(data=None, status=None, template_name=None, 
         headers=None, exception=False, conetnt_type=None)
```

[drf-response](https://www.django-rest-framework.org/api-guide/responses/)

```python
# 位置：from rest_framework.response import Response

class Response(SimpleTemplateResponse):
    """
    An HttpResponse that allows its data to be rendered into
    arbitrary media types.
    """
    # 实例化参数
    def __init__(self, data=None, status=None,
                 template_name=None, headers=None,
                 exception=False, content_type=None):
        """
        data: 响应数据 序列化器.data返回的字典 or 自定义返回字典内容
        status: 响应状态码
        template_name: drf默认提供的模板 可以修改(自定制)
        headers: 响应头
        exception: 异常处理
        content_type: 响应中 Content-Type标头告诉客户端实际返回的内容的内容类型
        """
        pass
    
"""
  RESTframework提供了一个响应类Response 使用该类构造响应对象时 响应的具体数据内容会被转换(render渲染)成符合前端需求的类型
  
  RESTframework提供了Renderer渲染器 用来根据请求头中的Accept(用户代理期望的MIME类型列表)来自动转换响应数据到对应格式 如果前端请求中未进行Accept声明 则会采用默认方式处理响应数据 我们可以通过配置来修改默认响应格式
"""
```

- **从修改默认响应渲染类看drf配置解析顺序**

```python
# 可以在rest_framework.settings查找所有的drf默认配置项
DEFAULTS = {
    # Base API policies
    'DEFAULT_RENDERER_CLASSES': [  # 默认响应渲染类
        'rest_framework.renderers.JSONRenderer',  # json渲染器
        'rest_framework.renderers.BrowsableAPIRenderer',  # 浏览API渲染器
    ]
    ...
}

"""
默认：浏览器响应成浏览器的格式 postman响应成json格式 - 通过配置实现
可以修改：
  1. 局部使用 - 对某个视图类有效
  2. 全局使用 - 对全部视图类有效
"""
注意：drf有默认的配置文件 即rest_framework.settings 
找的时候 先从项目的settings.py中找 -> 找不到采用自己默认的


# 全局：django项目settings.py配置
REST_FRAMEWORK = {  
    'DEFAULT_RENDERER_CLASSES': [
        'rest_framework.renderers.JSONRenderer', # 使浏览器访问也返回json数据
    ]
}

# 局部：视图函数中修改(类属性)
from rest_framework.renderers import JSONRenderer

class TestView(APIView):
    # 局部对视图类做配置(类属性)
    renderer_classes = ['rest_framework.renderers.JSONRenderer']
    def get(self, request):
        return Response({'msg': 'ok'})
    
# 总结：drf的配置信息查找顺序(源码：面向对象属性解析顺序)
先从自己类中找类属性配置 
-> django项目的settings.py中的REST_FRAMEWORK = [] 命名空间配置
-> drf自己的默认配置 rest_framework.settings

# django也有两套配置文件
# settings.py | django.conf.global_settings.py
思考：两套配置问价 如何 实现顺序的查找
     - 类的继承可以实现 但是django这里不是 它是单个的模块
     - django的实现：一个配置字典 先读取老的{key:value} 再读取新的{key:value}
       字典特性：key不重复 新的key的value会覆盖老的
```

#### 构造方式

```python
from rest_framework.views import APIView
from rest_framework.response import Response

class TestView(APIView):
    def get(self, request):
        print(request)
        # 返回用了默认templates渲染 如果使用浏览器访问 需要注册rest_framework
        # 否则浏览器会访问api.html 提示: TemplateDoesNotExist
        return Response({'name': 'minho'},
                        status=201,
                        headers={'token': 'xxx'})
```

- 响应示例

```json
HTTP 201 Created
Allow: GET, HEAD, OPTIONS
Content-Type: application/json
Vary: Accept
token: xxx

{
    "name": "minho"
}
```

- 常用属性

```python
1. Response().data
传给response对象的序列化后 但尚未render处理的数据

2. Response.status_code
状态码的数字

3. Response.content
经过render处理后的响应数据
```

#### 响应data

```python
"""
响应内容：
  data就是你要返回的数据 是一个字典
  可以是Serializer序列化后的字典 也可以是自定义的字典
"""
```

#### 响应status

```python
"""
响应状态码： 
  直接使用status定义的状态码 默认返回 200 OK
  它把所有使用到的状态码都定义成了常量 将数字对应了更明确的标识符
  return Response({'name': 'minho'},
                  status=status.HTTP_200_OK,
                  headers={'token': 'xxx'})
"""

from rest_framework import status

# status模块内容
def is_informational(code):
    return 100 <= code <= 199

def is_success(code):
    return 200 <= code <= 299

def is_redirect(code):
    return 300 <= code <= 399

def is_client_error(code):
    return 400 <= code <= 499

def is_server_error(code):
    return 500 <= code <= 599

HTTP_100_CONTINUE = 100
HTTP_101_SWITCHING_PROTOCOLS = 101
HTTP_200_OK = 200
HTTP_201_CREATED = 201
HTTP_202_ACCEPTED = 202
HTTP_203_NON_AUTHORITATIVE_INFORMATION = 203
HTTP_204_NO_CONTENT = 204
HTTP_205_RESET_CONTENT = 205
HTTP_206_PARTIAL_CONTENT = 206
HTTP_207_MULTI_STATUS = 207
HTTP_208_ALREADY_REPORTED = 208
HTTP_226_IM_USED = 226
HTTP_300_MULTIPLE_CHOICES = 300
HTTP_301_MOVED_PERMANENTLY = 301
HTTP_302_FOUND = 302
HTTP_303_SEE_OTHER = 303
HTTP_304_NOT_MODIFIED = 304
HTTP_305_USE_PROXY = 305
HTTP_306_RESERVED = 306
HTTP_307_TEMPORARY_REDIRECT = 307
HTTP_308_PERMANENT_REDIRECT = 308
HTTP_400_BAD_REQUEST = 400
HTTP_401_UNAUTHORIZED = 401
HTTP_402_PAYMENT_REQUIRED = 402
HTTP_403_FORBIDDEN = 403
HTTP_404_NOT_FOUND = 404
HTTP_405_METHOD_NOT_ALLOWED = 405
HTTP_406_NOT_ACCEPTABLE = 406
HTTP_407_PROXY_AUTHENTICATION_REQUIRED = 407
HTTP_408_REQUEST_TIMEOUT = 408
HTTP_409_CONFLICT = 409
HTTP_410_GONE = 410
HTTP_411_LENGTH_REQUIRED = 411
HTTP_412_PRECONDITION_FAILED = 412
HTTP_413_REQUEST_ENTITY_TOO_LARGE = 413
HTTP_414_REQUEST_URI_TOO_LONG = 414
HTTP_415_UNSUPPORTED_MEDIA_TYPE = 415
HTTP_416_REQUESTED_RANGE_NOT_SATISFIABLE = 416
HTTP_417_EXPECTATION_FAILED = 417
HTTP_418_IM_A_TEAPOT = 418
HTTP_422_UNPROCESSABLE_ENTITY = 422
HTTP_423_LOCKED = 423
HTTP_424_FAILED_DEPENDENCY = 424
HTTP_426_UPGRADE_REQUIRED = 426
HTTP_428_PRECONDITION_REQUIRED = 428
HTTP_429_TOO_MANY_REQUESTS = 429
HTTP_431_REQUEST_HEADER_FIELDS_TOO_LARGE = 431
HTTP_451_UNAVAILABLE_FOR_LEGAL_REASONS = 451
HTTP_500_INTERNAL_SERVER_ERROR = 500
HTTP_501_NOT_IMPLEMENTED = 501
HTTP_502_BAD_GATEWAY = 502
HTTP_503_SERVICE_UNAVAILABLE = 503
HTTP_504_GATEWAY_TIMEOUT = 504
HTTP_505_HTTP_VERSION_NOT_SUPPORTED = 505
HTTP_506_VARIANT_ALSO_NEGOTIATES = 506
HTTP_507_INSUFFICIENT_STORAGE = 507
HTTP_508_LOOP_DETECTED = 508
HTTP_509_BANDWIDTH_LIMIT_EXCEEDED = 509
HTTP_510_NOT_EXTENDED = 510
HTTP_511_NETWORK_AUTHENTICATION_REQUIRED = 511
```

#### 响应template_name

```python
"""
响应模板名：
  指定渲染的模板名字 可以自定义模板 基本用不到 了解即可
"""
```

#### 响应headers

[MDN HTTP消息头详解](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers)

```python
"""
响应头：
  可以往响应头放东西 就是一个字典 具体headers字段及含义 参考上面链接  
"""

# Django设置头字段
# 要设置或删除响应中的头字段 请像对待字典一样对待它
>>> response = HttpResponse()
>>> response['Age'] = 120
>>> del response['Age']
```

#### 响应content_type

[MDN Content-Type](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type)

[Media Types](https://www.iana.org/assignments/media-types/media-types.xhtml)

```python
"""
响应编码格式：
  请求也有 实体头部用于指示资源的MIME类型 media type
"""
```

### 封装Response对象

```python
# 自己封装Response对象
from rest_framework.response import Response

class APIResponse(Response):
    def __init__(
            self,
            code=100,
            msg='success',
            result=None,
            status=None,
            headers=None,
            **kwargs):
        dic = {'code': code, 'msg': msg}
        if data:
            dic['result'] = result
        dic.update(kwargs)
        # super() 对象来调用对应的绑定方法 会自动注入self
        super().__init__(data=dic, status=status, headers=headers)
        
        # 类来调用对象的绑定方法 这个方法就是一个普通函数 有几个参数传几个参数
        # Response.__init__(data=dic, status=status, headers=headers)
```

## 视图家族

### 两个视图基类

```python
APIView         # 继承django View

GenericAPIView  # 继承drf APIVIew 做了一些扩展 重要的如下 看源码重点关注
  类属性：
    - queryset = None
    - serializer_class = None
  类方法：
    - get_queryset()   常用
    - get_object()     获取一条数据
    - get_serializer() 常用
    - get_setrializer_class() 内部来用 外部会重写(视图类中用到多个序列化器的时候)

# get_object()源码解析
class GenericAPIView(views.APIView):
    ...
    def get_object(self):
        """
        Returns the object the view is displaying.

        You may want to override this if you need to provide non-standard
        queryset lookups.  Eg if objects are referenced using multiple
        keyword arguments in the url conf.
        """
        # 返回所有数据queryset对象 会有过滤
        queryset = self.filter_queryset(self.get_queryset())

        # Perform the lookup filtering.
        # lookup_field = 'pk'  
        # lookup_url_kwarg = None
        """
        url路径 对应有名分组的名字 默认叫 pk 不对应则报错
        如果需要修改：在自己的视图类 重写定义上面两个字段的其中一个
        这里的lookup_url_kwarg 就是pk(默认) 
        """
        lookup_url_kwarg = self.lookup_url_kwarg or self.lookup_field
  
        # {pk: 4} url有名分组 -> 关键字传参到对应的视图函数
        filter_kwargs = {self.lookup_field: self.kwargs[lookup_url_kwarg]}
        
        # 根据pk=4 去queryset对象中get单个对象
        obj = get_object_or_404(queryset, **filter_kwargs)
        
        # May raise a permission denied
        self.check_object_permissions(self.request, obj)
        return obj
```

#### APIView

- **继承APIView实现5个常用接口**

```python
# 前面视图类继承APIView写CBV 已经能比较快速的写出5个常用接口
# CBV代码如下：
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from .models import Book
from .serializers import BookSerializer

class BooksDetail(APIView):
    def get(self, request, pk):
        """获取一个数据"""
        book = Book.objects.filter(pk=pk).first()
        serializer = BookSerializer(instance=book)
        return Response(serializer.data)

    def put(self, request, pk):
        """修改一个数据"""
        book = Book.objects.filter(pk=pk).first()
        serializer = BookSerializer(book, request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk):
        """删除一个数据"""
        Book.objects.filter(pk=pk).delete()
        return Response(status=status.HTTP_204_NO_CONTENT)

class BooksList(APIView):
    def get(self, request):
        """获取数据列表(所有数据)"""
        books = Book.objects.all()
        serializer = BookSerializer(instance=books, many=True)
        return Response(serializer.data)

    def post(self, request):
        """新增一个数据"""
        serializer = BookSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

### GenericAPIVIew

- **源码分析**

```python
class GenericAPIView(views.APIView):
    queryset = None
    serializer_class = None
    ...
    
    def get_queryset(self):
        assert self.queryset is not None, (
            "'%s' should either include a `queryset` attribute, "
            "or override the `get_queryset()` method."
            % self.__class__.__name__
        )

        queryset = self.queryset
        if isinstance(queryset, QuerySet):
            # Ensure queryset is re-evaluated on each request.
            queryset = queryset.all()
        return queryset    
"""
看GenericAPIView源码发现 继承该类 需要覆盖2个类属性：
  - queryset -> QuertSet对象 - 指定使用哪个模型
  - serializer_class -> 序列化器 - 指定使用哪个序列化器来序列化上面模型类对应的数据
  
看get_queryset(self)方法知道 queryset可以写两种写法
  - queryset = Book.object.all()
  - queryset = Book.object  # 源码会自己判断
"""
```

- **继承GenericView实现上面APIView的接口功能**

```python
"""
基于GenericAPIView实现上面5个接口 其实非常简单
通过源码我们发现下面规律
"""
```

| 功能描述         | APIView                                                  | GenericView                                                  |
| ---------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| 获取所有数据     | `books = Book.objects.all()`                             | `books = self.get_queryset()`                                |
| 获取单个数据     | `book = Book.objects.filter(pk=pk).first()`              | `book = self.get_object()`                                   |
| 序列化所有数据   | `serializer = BookSerializer(instance=books, many=True)` | `serializer = self.get_serializer(instance=books, many=True)` |
| 序列化单个数据   | `serializer = BookSerializer(instance=book)`             | `serializer = self.get_serializer(instance=book)`            |
| 反序列化单个数据 | `serializer = BookSerializer(data=request.data)`         | `serializer = self.get_serializer(data=request.data)`        |
| 删除数据         | `Book.objects.filter(pk=pk).delete()`                    | `self.get_object().delete()`                                 |

- **发现以上规律后 使用GenericView重写上面5个接口则非常简单**

```python
from rest_framework.generics import GenericAPIView
from rest_framework.response import Response
from rest_framework import status
from .models import Book
from .serializers import BookSerializer

class BooksDetail(GenericAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

    def get(self, request, pk):
        book = self.get_object()
        serializer = self.get_serializer(instance=book)
        return Response(serializer.data)

    def put(self, request, pk):
        book = self.get_object()
        serializer = self.get_serializer(book, request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk):
        self.get_object().delete()
        return Response(status=status.HTTP_204_NO_CONTENT)

class BooksList(GenericAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

    def get(self, request):
        books = self.get_queryset()
        serializer = self.get_serializer(instance=books, many=True)
        return Response(serializer.data)

    def post(self, request):
        serializer = self.get_serializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
  
"""
继承GenericView重写这5个接口之后 有什么优势?
  我们只要把具体方法里面的之前表达业务字段的变量(book/books) 替换成 通用的没有具体含义的变量(如：obj)
  视图函数的代码实现就变得非常通用

如何通用?
  我们需要序列化其他模型类里面的数据的时候 只需要完成下面三步即可完成CBV视图函数的代码编写
  1. 赋值粘贴代码
  2. 修改CBV类名(如：BookList -> UserList)
  3. 重写赋值 queryset 和 serializer_class 两个类属性
"""

问题：
  这一步写完之后 虽然比较法方便 但是你会发现具体方法的代码既然都一样(冗余度大) 
  每次都要拷贝同样的代码 只是替换不同的数据模型以及对应模型的序列化器
  那我们如果把这些方法通用的代码都封装为不同的Mixin类 然后CBV视图直接使用多继承的方式 岂不是更加方便和简介
  

"""
drf提供了5个视图扩展Mixin类 封装了上面的通用功能
后面使用 只需要多继承这些Mixin类 即可使用其封装好的方法
  - 序列化所有数据 返回Response  self.list
  - 序列化单个数据 返回Response  self.retrieve
  - 反序列化一个数据(新增) 返回Response  self.create
  - 反序列化一个数据(修改) 返回Response  self.update
  - 删除一个数据 返回Response  self.destroy
"""
```

## 五个视图扩展Mixin类

```python
from rest_framework.mixins import *
```

### ListModelMixin

```python
class ListModelMixin:
    """
    List a queryset.
    """
    def list(self, request, *args, **kwargs):
        queryset = self.filter_queryset(self.get_queryset())

        page = self.paginate_queryset(queryset)
        if page is not None:
            serializer = self.get_serializer(page, many=True)
            return self.get_paginated_response(serializer.data)

        serializer = self.get_serializer(queryset, many=True)
        return Response(serializer.data)
```

### CreateModelMixin

```python
class CreateModelMixin:
    """
    Create a model instance.
    """
    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        self.perform_create(serializer)
        headers = self.get_success_headers(serializer.data)
        return Response(serializer.data, status=status.HTTP_201_CREATED, headers=headers)

    def perform_create(self, serializer):
        serializer.save()

    def get_success_headers(self, data):
        try:
            return {'Location': str(data[api_settings.URL_FIELD_NAME])}
        except (TypeError, KeyError):
            return {}
```

### RetrieveModelMixin

```python
class RetrieveModelMixin:
    """
    Retrieve a model instance.
    """
    def retrieve(self, request, *args, **kwargs):
        instance = self.get_object()
        serializer = self.get_serializer(instance)
        return Response(serializer.data)
```

### UpdateModelMixin

```python
class UpdateModelMixin:
    """
    Update a model instance.
    """
    def update(self, request, *args, **kwargs):
        partial = kwargs.pop('partial', False)
        instance = self.get_object()
        serializer = self.get_serializer(instance, data=request.data, partial=partial)
        serializer.is_valid(raise_exception=True)
        self.perform_update(serializer)

        if getattr(instance, '_prefetched_objects_cache', None):
            # If 'prefetch_related' has been applied to a queryset, we need to
            # forcibly invalidate the prefetch cache on the instance.
            instance._prefetched_objects_cache = {}

        return Response(serializer.data)

    def perform_update(self, serializer):
        serializer.save()

    def partial_update(self, request, *args, **kwargs):
        kwargs['partial'] = True
        return self.update(request, *args, **kwargs)
```

### DestroyModelMixin

```python
class DestroyModelMixin:
    """
    Destroy a model instance.
    """
    def destroy(self, request, *args, **kwargs):
        instance = self.get_object()
        self.perform_destroy(instance)
        return Response(status=status.HTTP_204_NO_CONTENT)

    def perform_destroy(self, instance):
        instance.delete()
```

### Mixin类使用

**基于GenericAPIView和5个Mixin类重写书籍管理接口**

```python
from rest_framework.mixins import CreateModelMixin, ListModelMixin
from rest_framework.mixins import DestroyModelMixin, RetrieveModelMixin, UpdateModelMixin
from rest_framework.generics import GenericAPIView
from .models import Book
from .serializers import BookSerializer

class BooksDetail(RetrieveModelMixin, UpdateModelMixin, DestroyModelMixin, GenericAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

    def get(self, request, pk):
        return self.retrieve(request, pk)

    def put(self, request, pk):
        return self.update(request, pk)

    def delete(self, request, pk):
        return self.destroy(request, pk)

class BooksList(ListModelMixin, CreateModelMixin, GenericAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

    def get(self, request):
        return self.list(request)

    def post(self, request):
        return self.create(request)
```

## 九个通用视图子类

```python
"""
上面使用多继承 继承对应方法的Mixin类和GenericView类重写几个接口 已经非常简介优雅
drf还提供了更方便的继承方法(drf自己完成对应的Mixin和GenericView的继承 你只需要按需继承drf提供的新的类即可)
"""
from rest_framework.generics import *
```

### CreateAPIView

```python
# 新增一个对象 POST
class CreateAPIView(mixins.CreateModelMixin,
                    GenericAPIView):
    """
    Concrete view for creating a model instance.
    """
    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)
```

### ListAPIView

```python
# 返回所有对象 GET
class ListAPIView(mixins.ListModelMixin,
                  GenericAPIView):
    """
    Concrete view for listing a queryset.
    """
    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)
```

### RetrieveAPIView

```python
# 返回一个对象 GET  retrieve: 取回
class RetrieveAPIView(mixins.RetrieveModelMixin,
                      GenericAPIView):
    """
    Concrete view for retrieving a model instance.
    """
    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)
```

### DestroyAPIView

```python
# 删除一个对象 DELETE
class DestroyAPIView(mixins.DestroyModelMixin,
                     GenericAPIView):
    """
    Concrete view for deleting a model instance.
    """
    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)
```

### UpdateAPIView

```python
# 更新一个对象 PUT/PATCH
class UpdateAPIView(mixins.UpdateModelMixin,
                    GenericAPIView):
    """
    Concrete view for updating a model instance.
    """
    def put(self, request, *args, **kwargs):
        return self.update(request, *args, **kwargs)

    def patch(self, request, *args, **kwargs):
        return self.partial_update(request, *args, **kwargs)
```

### ListCreateAPIView

```python
# List 返回所有对象|新增一个对象
class ListCreateAPIView(mixins.ListModelMixin,
                        mixins.CreateModelMixin,
                        GenericAPIView):
    """
    Concrete view for listing a queryset or creating a model instance.
    """
    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)
```

### RetrieveUpdateAPIView

```python
# 限制删除单个对象 返回一个对象|更新一个对象
class RetrieveUpdateAPIView(mixins.RetrieveModelMixin,
                            mixins.UpdateModelMixin,
                            GenericAPIView):
    """
    Concrete view for retrieving, updating a model instance.
    """
    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)

    def put(self, request, *args, **kwargs):
        return self.update(request, *args, **kwargs)

    def patch(self, request, *args, **kwargs):
        return self.partial_update(request, *args, **kwargs)
```

### RetrieveDestroyAPIView

```python
# 限制更新单个对象 返回一个对象|删除一个对象
class RetrieveDestroyAPIView(mixins.RetrieveModelMixin,
                             mixins.DestroyModelMixin,
                             GenericAPIView):
    """
    Concrete view for retrieving or deleting a model instance.
    """
    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)

    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)
```

### RetrieveUpdateDestroyAPIView

```python
# Detail 返回一个对象|更新一个对象|删除一个对象
class RetrieveUpdateDestroyAPIView(mixins.RetrieveModelMixin,
                                   mixins.UpdateModelMixin,
                                   mixins.DestroyModelMixin,
                                   GenericAPIView):
    """
    Concrete view for retrieving, updating or deleting a model instance.
    """
    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)

    def put(self, request, *args, **kwargs):
        return self.update(request, *args, **kwargs)

    def patch(self, request, *args, **kwargs):
        return self.partial_update(request, *args, **kwargs)

    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)
```

### 通用视图子类使用

**根据接口需求继承这9个视图子类重写接口**

```python
"""
这9个视图子类 都继承于GenericAPIView 然后根据不同的接口需求混入实现不同功能的Mixin类 并且实现对应的 get/post/put/patch/delete 方法 我们只需要根据我们接口需求 继承上面这9个视图子类的其中一个或几个即可 
然后仅仅只需提供GenericAPIView视图需要的 queryset 和 serializer_class 两个类属性即可十分简洁的完成接口的书写
"""

# 使用9个视图子类中的其中2个重写5个书籍管理接口
from rest_framework.generics import ListCreateAPIView
from rest_framework.generics import RetrieveUpdateDestroyAPIView
from .models import Book
from .serializers import BookSerializer

class BooksDetail(RetrieveUpdateDestroyAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

class BooksList(ListCreateAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    
"""
-> 发起请求 
-> 路由匹配 
-> CBV视图函数 
-> as_view()执行返回view函数地址
-> view函数执行dispath方法 完成请求分发 
-> dispath方法把此次请求的方法名转换为小写(request.method -> GET -> get -> getattr(self, get))
-> 通过反射getattr获取CBV视图函数的对应方法名称
-> 执行该方法

这个时候 我们继承的这9个视图子类 已经封装好了对应请求的方法
根据实例属性方法的查找是顺序 比如GET请求过来(request.method = GET)：
-> 自己写的CBV视图类找对应的get方法
-> 没有找到(这个时候不用再自己提供这些方法) 
-> 往父类查找
-> 最终会在这9个视图子类找到对应封装的get方法执行
-> 通用视图子类执行自己的get方法
-> 然后调用Mixin类封装的方法序列化/反序列化数据 并返回响应
"""
```

## 视图集ViewSet

```python
"""
使用上面通用视图子类 已经非常简洁的实现了5个接口
但是我们使用了2个CBV 而且对应的类属性都是一样的(queryset和serializer_calss)

我们是否可以直接合成一个?
问题：合成一个的问题在于 这两个CBV分别都有一个get方法
     - 一个get 是获取所有数据
     - 另一个get 是获取单个数据
如果能解决这个问题 那么就可以合并
如何解决?
"""
```

### ModelViewSet

**使用ModelViewSet编写5个接口 需要注意路由的变化**

```python
# urls.py
"""
注意路由变化 使用ViewSet actions参数必须传入
简单看下源码发现 此时的as_view方法 已经被ViewSetMixin类重写 

解决同一个get请求需要对应执行不同方法的问题
  - 获取所有数据 {'get': 'list'}
  - 获取单个数据 {'get': 'retrieve'}
"""
from django.urls import path
from . import views

urlpatterns = [
    # 使用ViewSet actions参数必须传 是一个字典 {请求小写: 执行的方法名}
    # 当路径匹配上 并且是get请求 会执行BookModelViewSet类的list方法 其余同理
    path('books/', views.BookModelViewSet.as_view(actions={'get': 'list', 'post': 'create'})),
    path('books/<int:pk>/', views.BookModelViewSet.as_view(actions={'get': 'retrieve', 'put': 'update', 'delete': 'destroy'})),
]

# views.py
"""合并BookList CBV 和BookDetail CBV"""
from rest_framework.viewsets import ModelViewSet
from .models import Book
from .serializers import BookSerializer

class BookModelViewSet(ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
```

### ViewSetMixin源码分析

```python
"""
ViewSetMixin重写了as_view方法 核心代码
只要路由中配置了对应关系 比如 {'get': 'list', 'post': 'create'}
当get请求过来 就会执行对象的list方法
当post请求过来 就会执行对象的create方法
  通过反射实现 把扩展Mixin类的 list create方法 拿过来赋值给当前ViewSet对象
"""
class ViewSetMixin:
    """
    This is the magic.

    Overrides `.as_view()` so that it takes an `actions` keyword that performs
    the binding of HTTP methods to actions on the Resource.

    For example, to create a concrete view binding the 'GET' and 'POST' methods
    to the 'list' and 'create' actions...

    view = MyViewSet.as_view({'get': 'list', 'post': 'create'})
    """

    @classonlymethod
    def as_view(cls, actions=None, **initkwargs):
  ...

        def view(request, *args, **kwargs):
            self = cls(**initkwargs)

            if 'get' in actions and 'head' not in actions:
                actions['head'] = actions['get']

            # {'get': 'list', 'post': 'create'}
            self.action_map = actions

            # Bind methods to actions
            # This is the bit that's different to a standard view
            for method, action in actions.items():
                # method: get | action: list
                # 反射获取属性：执行完handler赋值 handler就变成了list(Mixin类写好的)的内存地址
                handler = getattr(self, action)
                # 反射设置属性：当前视图对象.get = list 
                # 当get请求过来 CBV经过dispath分发 触发视图函数的.get方法
                setattr(self, method, handler)
                # for循环执行完毕 对象.get -> list 对象.post -> create

            self.request = request
            self.args = args
            self.kwargs = kwargs

            # And continue as usual
            return self.dispatch(request, *args, **kwargs)
  ...
        return csrf_exempt(view)
```

### 继承ViewSetMixin的视图类

```python
"""
通过上面源码的分析 我们知道使用ViewSet的时候 as_view需要一个actions参数
它会将请求方法(key)与你指定的执行函数(value)绑定
那么 继承ViewSetMixin的视图类 我们就可以改写路由 以及改写CBV提供的get/post等的方法名称

总结：继承ViewSetMixin视图类 配置好actions参数的映射关系 CBV方法名称可以任意
"""
urlpatterns = [
    path('books/', views.BookModelViewSet.as_view(actions={'get': 'get_all_book'}))
]

# ViewSetMixin一定要放在APIView前面 
# 多继承属性查找顺序 让as_view方法先找ViewSetMixin的
class BookModelViewSet(ViewSetMixin, APIView):
    def get_all_book(self, request):
        books = Book.objects.all()
        serializer = BookSerializer(books, many=True)
        return Response(serializer.data)
    
# drf内置提供了这些继承ViewSetMixin视图类的子类
```

#### ViewSet

```python
# 继承APIview 就是上一步我们实现的
# 没有提供任何actions方法 需要我们写的代码是最多的
class ViewSet(ViewSetMixin, views.APIView):
    """
    The base ViewSet class does not provide any actions by default.
    """
    pass
```

#### GenericViewSet

```python
# 继承GenericView 可以使用GenericView的各种方法
# 涉及到序列化/数据库操作 尽量使用GenericView 方便
class GenericViewSet(ViewSetMixin, generics.GenericAPIView):
    """
    The GenericViewSet class does not provide any actions by default,
    but does include the base set of generic view behavior, such as
    the `get_object` and `get_queryset` methods.
    """
    pass
```

#### ReadOnlyModelViewSet

```python
# 只读视图集 只提供获取所有数据 和获取单个数据的方法
class ReadOnlyModelViewSet(mixins.RetrieveModelMixin,
                           mixins.ListModelMixin,
                           GenericViewSet):
    """
    A viewset that provides default `list()` and `retrieve()` actions.
    """
    pass
```

#### ModelViewSet

```python
# 前面使用的ModelViewSet 提供了5个接口的实现方法
"""
get    -  list/retrieve
post   -  create
put    -  update
patch  -  partial_update
delete -  destroy
"""
class ModelViewSet(mixins.CreateModelMixin,
                   mixins.RetrieveModelMixin,
                   mixins.UpdateModelMixin,
                   mixins.DestroyModelMixin,
                   mixins.ListModelMixin,
                   GenericViewSet):
    """
    A viewset that provides default `create()`, `retrieve()`, `update()`,
    `partial_update()`, `destroy()` and `list()` actions.
    """
    pass
```

# 路由Routers

## 简单写法

**在urls.py中配置**

### **FBV**

```python
path('publishs/<int:pk>/', views.detail)
```

### **CBV**

```python
path('publishs/<int:pk>/', views.PublishDetail.as_view())
```

### ViewSetMixin

```python
# 一旦视图类继承ViewSetMixin之后 写法稍有不同
# as_view() 必须传入actions参数(位置或关键字传参)
# ViewSetMixin重写as_view 比如get：通过反射取'list'函数 赋值给'get'
path('books/', views.BookViewSet.as_view(actions={'get': 'list', 'post': 'create'}))
```

## Routers

```python
对于视图集ViewSet 我们除了可以手动指明请求的方式与动作action之间的对应关系以外
还可以使用Routers来帮助我们快速实现路由信息
```

**routers写法**

```python
# 继承自ModelViewSet的路由Routers写法

# 1. 导入routers模块
from rest_framework import routers

# 2. 该模块有两个类 实例化得到对象
# routers.DefaultRouter() -> 生成的路由更多
# routers.SimpleRouter() -> 生成两条路由
# router = routers.SimpleRouter()
router = routers.DefaultRouter()

# 3. 注册
# prefix: 路由前缀 不要加斜杠
# viewset: 继承自ModelViewSet的视图类
# basename: 别名 reverse反向解析使用
# router.register(prefix, viewset, basename=None)
router.register('books', views.BookViewSet)

# 4. 将router.urls自动生成的路由加到原路由中
     4.1 urlpatterns += router.urls
     4.2 urlpatterns = [path('', include(router.urls))]
```

### SimpleRouter和DefaultRouter

```python
"""SimpleRouter 自动生成2条路由"""
<URLPattern '^books/$' [name='book-list']>,
<URLPattern '^books/(?P<pk>[^/.]+)/$' [name='book-detail']>


"""DefaultRouter 自动生成6条路由"""
# 这两条跟SimpleRouter一样
<URLPattern '^books/$' [name='book-list']>, 
<URLPattern '^books/(?P<pk>[^/.]+)/$' [name='book-detail']>, 

# 下面两条有名分组：format关键字传参
<URLPattern '^books\.(?P<format>[a-z0-9]+)/?$' [name='book-list']>,  # http://127.0.0.1:8000/books.json
<URLPattern '^books/(?P<pk>[^/.]+)\.(?P<format>[a-z0-9]+)/?$' [name='book-detail']>,  # http://127.0.0.1:8000/books/5.json

<URLPattern '^$' [name='api-root']>,   # 根路径会显示出所有可以访问的地址
<URLPattern '^\.(?P<format>[a-z0-9]+)/?$' [name='api-root']>  # http://127.0.0.1:8000/books/books.api|.json
```

## action使用

```python
# 作用：为了给继承自ModelViewSet的视图类中自定义的方法自动生成路由

# 如何使用?
from rest_framework.response import Response
from rest_framework.viewsets import ModelViewSet
from rest_framework.decorators import action

from .models import Book
from .serializers import BookSerializer

class BookViewSet(ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

    """
    action是一个带参装饰器
    参数：methods -> 是一个列表 列表中放请求方式 methods=['GET', 'POST']
    参数：detail -> 布尔类型 True/False 是否带pk
      - True: ^books/(?P<pk>[^/.]+)/get_one/$ 生成带pk的地址
      - False: ^books/get_one/ 生成不带pk的地址
    当朝生成的地址发送methods里面支持的请求 会执行下面的函数
    """
    @action(methods=['GET', 'POST'], detail=True)
    def get_one(self, request, pk):
        print(pk)
        # 截取1条
        book = self.get_queryset()[:2]
        # 虽然只取1条 但是结果仍然是可迭代对象 不是单个对象 仍需设置many=True
        serializer = self.get_serializer(book, many=True)
        return Response(serializer.data)
```

# 认证组件

[Authentication](https://www.django-rest-framework.org/api-guide/authentication/)

## 认证类的写法

```python
"""
drf认证得实现：
  1. 写一个类 继承BaseAuthentication
     注：不继承也可以 只要实现authenticate方法即可 它只是规定了子类的行为 不用一定继承它
  2. 重写authenticate方法(实现认证得逻辑) 因为源码中 调用的时候 需要执行认证类的该方法
     2.1 认证通过 返回两个值(self.user self.auth) 
         - self.user最终给request.user 当前登录用户
         - self.auth
     2.2 认证失败 抛异常：AuthenticationFailed(推荐这个) or APIException
     
  3. 使用
     3.1 全局使用 settings.py配置
     3.2 局部使用 视图类写类属性
     
     看源码知道 全局或者局部配置的属性名字为 如：authentication_classes = [SessionAuthentication, BasicAuthentication]
     需要是一个可迭代对象 可以配置多个 按顺序执行 类似配置认证装饰器
"""
```

## 认证源码分析

```python
"""
-> APIView 
-> dispath方法 
-> self.initial(request, *args, **kwargs)
-> self.initial里面包含：
   - 认证 self.perform_authentication(request)  # 认证组件
   - 权限 self.check_permissions(request)
   - 频率 self.check_throttles(request)
"""
   
# 读：self.perform_authentication(request)
class APIView(View):
     authentication_classes = api_settings.DEFAULT_AUTHENTICATION_CLASSES
        
    def initialize_request(self, request, *args, **kwargs):
        # 实例化Request类 传入authenticators认证器  
        return Request(
            request,
            parsers=self.get_parsers(),
            # 是一个列表 [认证类的对象, 认证类的对象, ...]
            authenticators=self.get_authenticators(),
            negotiator=self.get_content_negotiator(),
            parser_context=parser_context
        )
    
    def get_authenticators(self):
        # 列表解析式 去自己的authentication_classes(可迭代对象)取auth 然后直接auth() 执行
        # 这里authentication_classes是一个APIView的类属性 取的默认配置
        # 列表中是一堆对象(认证类实例) 视图类中配置的authentication_classes = [类名]
        return [auth() for auth in self.authentication_classes]   
    
 def initial(self, request, *args, **kwargs):
        self.perform_authentication(request)
        self.check_permissions(request)
        self.check_throttles(request)
 
    def perform_authentication(self, request):
        # 就一句话 需要去drf的Request类中查找user属性(方法)
        # 进源码看 实际是一个方法 用@property包装成了一个属性
        request.user
        
class Request:
    @property
    def user(self):
        if not hasattr(self, '_user'):
            with wrap_attributeerrors():  # 上下文管理
                # 刚开始来 没有_user 走self._authenticate
                self._authenticate()  # 核心
        # 有用户 直接返回用户
        return self._user
    
    # 做认证 核心
    def _authenticate(self):
        # self.authenticators Request类实例化(APIView的dispath的时候实例化的)的时候传入的参数
        # 这个参数的类型：是配置的一堆认证类产生的认证类对象组成的list
        # 遍历拿到一个个认证器(认证类的实例化后得到的对象) 进行认证
        for authenticator in self.authenticators:
            try:
                # 认证器(对象)调用authenticate(认证类对象(认证类自动注入的self), request请求对象) 所以自定义类实现该方法的时候 需要一个形参接收该self(request请求对象)
                # 返回值：登录用户与认证的信息组成的tuple
                # try语句 认证失败抛出异常
                user_auth_tuple = authenticator.authenticate(self)  # 这里的self是request请求对象
            except exceptions.APIException:
                self._not_authenticated()
                raise
   
            # 返回值的处理
            if user_auth_tuple is not None:
                self._authenticator = authenticator
                # 如果有返回值 将 登录用户 与 登录认证 分别保存到 request.user request.auth
                self.user, self.auth = user_auth_tuple
                return
  # 如果返回值user_auth_tuple为空 代表认证通过
        # 但是没有登录用户与登录信息 代表匿名用户(游客)
        self._not_authenticated()
```

## 自定义认证类

### 简单登录实现

```python
测试简单存了一个token值到服务端数据库 仅为测试方便类似session
```

- **认证测试模型**

```python
from django.db import models

class User(models.Model):
    username = models.CharField(max_length=32)
    password = models.CharField(max_length=256)
    user_type = models.IntegerField(choices=((1, '超级用户'), (2, '普通用户'), (3, '二笔用户')))
    age = models.IntegerField()
    register_time = models.DateField()
    gender_choices = [(1, '男'), (2, '女'), (3, '其他')]
    gender = models.IntegerField(choices=gender_choices)

class UserToken(models.Model):
    """垂直分表(库)|水平分表(库)"""
    token = models.CharField(max_length=64)
    # 一对一关联到User表 一对一：就是垂直分表
    user = models.OneToOneField(to='User', on_delete=models.CASCADE)
```

- **认证视图函数**

```python
import uuid
from rest_framework.views import APIView
from rest_framework.response import Response
from . import models

class LoginView(APIView):
    def post(self, request):
        username = request.data.get('username')
        password = request.data.get('password')
        user = models.User.objects.filter(username=username, password=password).first()
        if user:
            # 登录成功 生成一个随机字符串
            token = uuid.uuid4()
            # 存到UserToken表中
            # models.UserToken.objects.create(token=token, user=user)  # 这种方式每次登录都会记录一条 不好 有记录 更新即可
            # update_or_create 没有就新增 有就更新
            models.UserToken.objects.update_or_create(defaults={'token': token}, user=user)
            return Response({'status': 100, 'msg': '登录成功', 'token': token})
        return Response({'status': 101, 'msg': '用户名或密码错误'})
```

### 实现自定义认证类

- **编写自定义认证类**

```python
from rest_framework.authentication import BaseAuthentication
from rest_framework.exceptions import AuthenticationFailed
from .models import UserToken
from django.db import models

"""
访问URL 需要写法token参数：
127.0.0.1:8000/books/?token=147f71b0-7665-4a06-a178-24fbe98c9332
"""
class MyAuthentication(BaseAuthentication):
    def authenticate(self, request):
        # 认证逻辑
        # 如果认证通过 返回元组(user, auth)
        token = request.query_params.get('token')
        if token:
            try:
                user_token = UserToken.objects.get(token=token)
                # 这里返回(user对象, token) 不一定返回user对象
                # 可以返回字符串(user_token.user.username)或其他 返回对象 后面方便request.user使用
                return user_token.user, token
            except models.ObjectDoesNotExist:
                # 如果认证失败 抛出AuthenticationFailed异常
                raise AuthenticationFailed('认证失败')
        raise AuthenticationFailed('请求地址中需要携带token')
```

- **使用自定义认证类**

```python
class BookViewSet(ModelViewSet):
    authentication_classes = [MyAuthentication]  # 局部配置
    queryset = Book.objects.all()
    serializer_class = BookSerializer

    @action(methods=['get'], detail=False)
    def get_one(self, request):
        print(request.user)  # user - minho
        print(request.auth)  # token - 147f71b0-7665-4a06-a178-24fbe98c9332 
        book = self.get_queryset()[:1]
        serializer = BookSerializer(book, many=True)
        return Response(serializer.data)
    
    
# 全局配置 settings.py
# 可以有多个 顺序执行配置的认证类
"""
需要注意：如果配置多个认证类 要把返回两个值的放到最后
        因为request.user request.auth会一直重复覆盖
"""
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": ['app01.authentication.MyAuthentication']
}

"""
全局配置这里有一个问题：
  全局配置后 所有视图都会进行认证 包括登录视图
  还没登录就进行认证... 
  
  这里需要对登录视图做局部配置 登录视图不需要任何认证
"""
class LoginView(APIView):
    authentication_classes = []  # 局部禁用认证 置为空列表实现
```

## 内置认证类

### BasicAuthentication

```python
# 针对用户名/密码的HTTP基本身份验证
class BasicAuthentication(BaseAuthentication): pass
    """
    HTTP Basic authentication against username/password.
    ""
```

### RemoteUserAuthentication

```python
class RemoteUserAuthentication(BaseAuthentication): pass
    """
    REMOTE_USER authentication.

    To use this, set up your web server to perform authentication, which will
    set the REMOTE_USER environment variable. You will need to have
    'django.contrib.auth.backends.RemoteUserBackend in your
    AUTHENTICATION_BACKENDS setting
    """
```

### SessionAuthentication

```python
class SessionAuthentication(BaseAuthentication): pass
 """
 Use Django's session framework for authentication. 
 """
```

### TokenAuthentication

```python
class TokenAuthentication(BaseAuthentication): pass
 """
 Simple token based authentication.

 Clients should authenticate by passing the token key in the "Authorization"
 HTTP header, prepended with the string "Token ".  For example:

 Authorization: Token 401f7ac837da42b97f613d789819ff93537bee6a
    """
```

# 权限组件

```python
"""
-> APIView 
-> dispath方法 
-> self.initial(request, *args, **kwargs)
-> self.initial里面包含：
   - 认证 self.perform_authentication(request)
   - 权限 self.check_permissions(request)   # 权限组件 在APIView里面 比认证简单一些 而且在认证之后执行 可以拿到request.user
   - 频率 self.check_throttles(request)
"""

权限需要和认证配合使用 给认证的用户 分配不同的权限
```

## 权限源码分析

```python
class APIView(View):
    def check_permissions(self, request):
        """
        Check if the request should be permitted.
        Raises an appropriate exception if the request is not permitted.
        """
        for permission in self.get_permissions():  # 权限类的对象 放到列表中
            # 执行权限类的has_permission方法
            # 从原理这里的判断可以看出 返回值应该是 True/False
            # 权限通过：True  不通过：False
            # 在认证之后执行 可以拿到request.user
            if not permission.has_permission(request, self):
                self.permission_denied(
                    request,
                    message=getattr(permission, 'message', None),
                    code=getattr(permission, 'code', None)
                )    
```

## 自定义权限类

- **编写**

```python
"""
1. 继承BasePermission
2. 重写has_permission
   2.1 返回True: 有权限
   2.2 返回Flase: 无权限
"""
from rest_framework.permissions import BasePermission

class UserPermission(BasePermission):
    # request drf的request对象
    # view 自己写的视图类对象 也有view.request 可以操作更多属性
    def has_permission(self, request, view):
  """
  实现：不是超级用户 不能访问
        由于已经认证过 request.user可以直接取到当前登录用户
        if request.user.user_type == 1:
            return True
        return False
        如果该字段使用了choice 通过get_字段名_display()就能取出后面的备注信息
        print(request.user.get_user_type_display())
  """
        return True if request.user.user_type == 1 else False
```

- **使用**

```python
from .serializers import BookSerializer
from .models import Book
from .authentication import MyAuthentication
from .authentication import UserPermission

class BookViewSet(ModelViewSet):
    authentication_classes = [MyAuthentication]
    permission_classes = [UserPermission]  # 视图类里面局部使用
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    
# 全局使用 settings.py
REST_FRAMEWORK = {
        "DEFAULT_AUTHENTICATION_CLASSES": ['app01.authentication.MyAuthentication'],
        'DEFAULT_PERMISSION_CLASSES': ['rest_framework.permissions.AllowAny'],
}

# 全局使用之后 部分视图需要局部禁用
permission_classes = []
```

**输出示例**

```json
{
    "detail": "You do not have permission to perform this action."
}
```

## 内置权限类

### IsAdminUser

```python
# 使用django的Auth模块
class IsAdminUser(BasePermission):
    """
    Allows access only to admin users.
    """
    
    def has_permission(self, request, view):
        return bool(request.user and request.user.is_staff)
```

### IsAuthenticated

```python
class IsAuthenticated(BasePermission):
    """
    Allows access only to authenticated users.
    """

    def has_permission(self, request, view):
        return bool(request.user and request.user.is_authenticated)
```

### IsAuthenticatedOrReadOnly

```python
class IsAuthenticatedOrReadOnly(BasePermission):
    """
    The request is authenticated as a user, or is a read-only request.
    """

    def has_permission(self, request, view):
        return bool(
            request.method in SAFE_METHODS or
            request.user and
            request.user.is_authenticated
        )

```

**其余自带权限类参考drf.permissions模块**

# 频率组件

```python
可以对接口访问的频次进行限制 以减轻服务器压力
```

## 自定义IP频率限制类

### 继承BaseThrottle

- **BaseThrottle源码分析**

```python
class BaseThrottle:
    """
    Rate throttling of requests.
    """

    def allow_request(self, request, view):
        """
        Return `True` if the request should be allowed, `False` otherwise.
        """
        # 该方法判断是否限次 没有限次可以请求返回True 限次了不可以请求返回False
        raise NotImplementedError('.allow_request() must be overridden')

    def get_ident(self, request):
        """
        Identify the machine making the request by parsing HTTP_X_FORWARDED_FOR
        if present and number of proxies is > 0. If not use all of
        HTTP_X_FORWARDED_FOR if it is available, if not use REMOTE_ADDR.
        """
        xff = request.META.get('HTTP_X_FORWARDED_FOR')
        remote_addr = request.META.get('REMOTE_ADDR')
        num_proxies = api_settings.NUM_PROXIES

        if num_proxies is not None:
            if num_proxies == 0 or xff is None:
                return remote_addr
            addrs = xff.split(',')
            client_addr = addrs[-min(num_proxies, len(addrs))]
            return client_addr.strip()

        return ''.join(xff.split()) if xff else remote_addr

    def wait(self):
        """
        Optionally, return a recommended number of seconds to wait before
        the next request.
        """
        # 限次后调用 限制还需要等多久才能再次访问 返回等待时间的seconds
        return None
```

- **自定制频率限制类**

```python
"""
自定义的逻辑
  1. 取出访问者IP
  2. 判断当前IP在不在访问字典里
     2.1 不在 添加进去 并且直接返回True 表示第一次访问
     2.2 在字典里 继续往下走
  3. 循环判断当前IP的列表 有值 并且当前时间减去列表的最后一个时间大于60s 把这种数据pop掉 这样列表中只有60s以内的访问记录
  4. 判断 
     4.1 当列表小于3 说明一分钟以内访问不足三次 把当前时间插入列表第一个位置 返回True 顺序通过
     4.2 当大于等于3 说明一分钟内访问超过三次 返回False验证失败
"""

# 频率限制 自定制
import time
from rest_framework.throttling import BaseThrottle

class IPThrottle(BaseThrottle):
    # 这里我们的访问字典放在了内存里 重启就会被清空
    # 源码提供的放在了django缓存
    VISIT_DIC = {}  # 定义成类属性 所有对象都用这一个

    def __init__(self):
        # 不能设置为类属性 每个IP的列表不一样
        self.hisroty_list = []  # type: list

    def allow_request(self, request, view):
        ip = request.META.get('REMOTE_ADDR')
        ctime = time.time()
        if ip not in self.VISIT_DIC:
            self.VISIT_DIC[ip] = [ctime]
            return True
        # 当前访问者的时间列表拿出来
        self.history_list = self.VISIT_DIC[ip]
        # while ctime - history_list[-1] > 60:
        #     history_list.pop()
        while True:
            if ctime - self.history_list[-1] > 60:
                # 循环把时间列表最后一个与当前时间差大于60s的移除
                self.history_list.pop()
            else:
                break  # 把全部大于60s的时间移除之后break
        if len(self.history_list) < 3:  # 3 就是配置文件里面配置一分钟可以访问多少次
            self.history_list.insert(0, ctime)
            return True
        else:
            return False

    def wait(self):
        # history_list = [第三次 第二次 第一次访问时间戳]
        # 返回该IP还需要等待多少秒可以访问
        ctime = time.time()
        return 60-(ctime - self.history_list[-1])

    
# 使用
1. 局部使用 在视图类里使用
   throttle_classes = [MyThrottles,]
2. 全局使用 settings.py
   REST_FRAMEWORK = {
       'DEFAULT_THROTTLE_CLASSES':['app01.utils.MyThrottles',]
   }
```

### 继承SimpleRateThrottle

- **SimpleRateThrottle源码分析**

```python
class SimpleRateThrottle(BaseThrottle):
    """
    A simple cache implementation, that only requires `.get_cache_key()`
    to be overridden.

    The rate (requests / seconds) is set by a `rate` attribute on the View
    class.  The attribute is a string of the form 'number_of_requests/period'.

    Period should be one of: ('s', 'sec', 'm', 'min', 'h', 'hour', 'd', 'day')

    Previous request information used for throttling is stored in the cache.
    """
    cache = default_cache
    timer = time.time
    cache_format = 'throttle_%(scope)s_%(ident)s'
    scope = None
    THROTTLE_RATES = api_settings.DEFAULT_THROTTLE_RATES

    def __init__(self):
        # 通过反射找rate
        if not getattr(self, 'rate', None):
            self.rate = self.get_rate()  # self.rate = '3/m'
        # self.num_requests = 3
        # self.duration = 60
        self.num_requests, self.duration = self.parse_rate(self.rate)

    def get_cache_key(self, request, view):
        """
        Should return a unique cache-key which can be used for throttling.
        Must be overridden.

        May return `None` if the request should not be throttled.
        """
        # 必须重写 返回谁 用什么做key
        raise NotImplementedError('.get_cache_key() must be overridden')

    def get_rate(self):
        """
        Determine the string representation of the allowed request rate.
        """
        if not getattr(self, 'scope', None):
            msg = ("You must set either `.scope` or `.rate` for '%s' throttle" %
                   self.__class__.__name__)
            raise ImproperlyConfigured(msg)

        try:
            # 去配置文件取配置的scope值 '3/m'
            return self.THROTTLE_RATES[self.scope]
        except KeyError:
            msg = "No default throttle rate set for '%s' scope" % self.scope
            raise ImproperlyConfigured(msg)

    def parse_rate(self, rate):
        """
        Given the request rate string, return a two tuple of:
        <allowed number of requests>, <period of time in seconds>
        """
        if rate is None:
            return (None, None)
        num, period = rate.split('/')  # rate: '3/m' -> '3', 'm'
        num_requests = int(num)
        duration = {'s': 1, 'm': 60, 'h': 3600, 'd': 86400}[period[0]]  # period取字符串索引0 min:m mmmmm:m
        return (num_requests, duration)  # (3, 60)

    def allow_request(self, request, view):
        """
        Implement the check to see if the request should be throttled.

        On success calls `throttle_success`.
        On failure calls `throttle_failure`.
        """
        if self.rate is None:
            return True

        # 获取key(IP userID ...)
        self.key = self.get_cache_key(request, view)
        if self.key is None:
            return True

        # django缓存
        # 1. 导包：from django.core.cache import cache
        # 2. 添加缓存：cache.set(key, value, exp)
        # 3. 获取缓存： cache.get(key, default_value)
        # 初次访问缓存为空 self.history = [] 是存放时间的列表
        self.history = self.cache.get(self.key, [])
        # 获取当前时间 存到self.now
        self.now = self.timer()

        # Drop any requests from the history which have now passed the
        # throttle duration
        while self.history and self.history[-1] <= self.now - self.duration:
            self.history.pop()
        if len(self.history) >= self.num_requests:
            return self.throttle_failure()
        return self.throttle_success()

    def throttle_success(self):
        """
        Inserts the current request's timestamp along with the key
        into the cache.
        """
        self.history.insert(0, self.now)
        self.cache.set(self.key, self.history, self.duration)
        return True

    def throttle_failure(self):
        """
        Called when a request to the API has failed due to throttling.
        """
        return False

    def wait(self):
        """
        Returns the recommended next request time in seconds.
        """
        # 计算下一次访问需要等待的时间
        if self.history:
            remaining_duration = self.duration - (self.now - self.history[-1])
        else:
            remaining_duration = self.duration

        available_requests = self.num_requests - len(self.history) + 1
        if available_requests <= 0:
            return None

        return remaining_duration / float(available_requests)
```

- **继承SimpleRateThrottle自定义限制key**

```python
from rest_framework.throttling import SimpleRateThrottle

class MyThrottle(SimpleRateThrottle):
    """
    SimpleRateThrottle 已经提供了其他方法 
    只需要重写get_cache_key即可
    """
    scope = 'sss'  # scope配置限制的关键字

    def get_cache_key(self, request, view):
        # 该方法返回什么 就以什么为key -> 以key来限制访问频率 key对应的value是一个时间列表
        # 这里返回客户端IP 就以IP限制访问次数
        # 如果返回一个固定值 所有人都共有一个key的限制 return xxx
        print("IP", request.META.get('REMOTE_ADDR'))
        return request.META.get('REMOTE_ADDR') 
    
"""
自定义频率限制(IP user_id)
  1. 继承SimpleRateThrottle
  2. 重写get_cache_key 返回什么 就以什么为key进行限制(最好唯一 key对应的值是一个时间列表)
  3. scope字段 需要与setting中对应 配置访问频率限制 'scope_name': '5/m'
"""
```

## 内置的频率限制

```python
from rest_framework.throttling import *

# drf默认配置
'DEFAULT_THROTTLE_CLASSES': []

# Throttling
'DEFAULT_THROTTLE_RATES': {
    'user': None,
    'anon': None,
},
```

### AnonRateThrottle

```python
"""
限制所有匿名未认证用户 使用IP区分用户
使用DEFAULT_THROTTLE_RATES['anon'] 来限制频次
"""
class AnonRateThrottle(SimpleRateThrottle):
    """
    Limits the rate of API calls that may be made by a anonymous users.
    The IP address of the request will be used as the unique cache key.
    """
    scope = 'anon'

    def get_cache_key(self, request, view):
        if request.user.is_authenticated:
            return None  # Only throttle unauthenticated requests.

        return self.cache_format % {
            'scope': self.scope,
            'ident': self.get_ident(request)
        }
```

### UserRateThrottle

```python
"""
限制认证用户 使用user id区分
使用DEFAULT_THROTTLE_RATES['user'] 来限制频次
"""
class UserRateThrottle(SimpleRateThrottle):
    """
    Limits the rate of API calls that may be made by a given user.
    The user id will be used as a unique cache key if the user is authenticated. For anonymous requests, the IP address of the request will be used.
    """
    scope = 'user'

    def get_cache_key(self, request, view):
        if request.user.is_authenticated:
            ident = request.user.pk
        else:
            ident = self.get_ident(request)

        return self.cache_format % {
            'scope': self.scope,
            'ident': ident
        }
```

### ScopedRateThrottle

```python
"""
限制用户对于每个视图的访问频次 使用ip或者user id
"""
class ScopedRateThrottle(SimpleRateThrottle):
    """
    Limits the rate of API calls by different amounts for various parts of the API.  Any view that has the `throttle_scope` property set will be throttled.  The unique cache key will be generated by concatenating the user id of the request, and the scope of the view being accessed.
    """
    scope_attr = 'throttle_scope'

    def __init__(self):
        # Override the usual SimpleRateThrottle, because we can't determine
        # the rate until called by the view.
        pass

    def allow_request(self, request, view):
        # We can only determine the scope once we're called by the view.
        self.scope = getattr(view, self.scope_attr, None)

        # If a view does not have a `throttle_scope` always allow the request
        if not self.scope:
            return True

        # Determine the allowed request rate as we normally would during
        # the `__init__` call.
        self.rate = self.get_rate()
        self.num_requests, self.duration = self.parse_rate(self.rate)

        # We can now proceed as normal.
        return super().allow_request(request, view)

    def get_cache_key(self, request, view):
        """
        If `view.throttle_scope` is not set, don't apply this throttle.
        Otherwise generate the unique cache key by concatenating the user id with the '.throttle_scope` property of the view.
        """
        if request.user.is_authenticated:
            ident = request.user.pk
        else:
            ident = self.get_ident(request)

        return self.cache_format % {
            'scope': self.scope,
            'ident': ident
        }
```

## 使用

- **限制匿名用户每分钟访问5次**

```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': ['rest_framework.throttling.AnonRateThrottle',],
 'DEFAULT_THROTTLE_RATES': {
     'anon': '5/m',
 },
}
# 使用 second minute hour day来置命周期
# 可以全局使用 局部使用

# 局部使用 设置视图使用的频率限制类 访问频次仍在全局配置
throttle_classes = [AnonRateThrottle]
```

- **限制登陆用户每分钟访问10次**

```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': ['rest_framework.throttling.UserRateThrottle',],
 'DEFAULT_THROTTLE_RATES': {
     'user': '10/m',
 },
}

# 可以全局使用 局部使用
# 限制登陆用户 局限在于需要使用Django Amdin的那一套认证

# 多个频率类同时使用
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.UserRateThrottle',                   'rest_framework.throttling.AnonRateThrottle'
    ],
 'DEFAULT_THROTTLE_RATES': {
     'user': '10/m',
        'anon': '5/m'
 },
}
```

**输出示例**

```python
{
    "detail": "Request was throttled. Expected available in 48 seconds."
}
```

# 解析组件

```python
from rest_framework.parsers import *

# 看下对应解析器源码即可

# 默认配置如下
'DEFAULT_PARSER_CLASSES': [
        'rest_framework.parsers.JSONParser',
        'rest_framework.parsers.FormParser',
        'rest_framework.parsers.MultiPartParser'
]
```

# 过滤

[Filtering](https://q1mi.github.io/Django-REST-framework-documentation/api-guide/filtering_zh/)

## 过滤django-filter

[DjangoFilterBackend](https://q1mi.github.io/Django-REST-framework-documentation/api-guide/filtering_zh/#djangofilterbackenddjango)

```python
"""
对于列表数据 可能需要根据字段进行过滤 
我们可以通过添加django-filter扩展来增强支持
"""
```

- **使用**

```python
# 不是drf内置 需要安装
pip install django-filter

# 注册app
INSTALLED_APPS = [
    ...
    'django_filters',
]

# 全局配置：在配置文件中增加过滤后端的配置
REST_FRAMEWORK = {
    ...
    'DEFAULT_FILTER_BACKENDS': ['django_filters.rest_framework.DjangoFilterBackend']
}

# 在视图中添加filter_fields属性 指定可以过滤的字段
class Bks(ListAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    filter_fields = ['title', 'price']  # 配置可以按照哪个字段来过滤
    
# http://127.0.0.1:8000/bks/?price=8
```

- **Django Admin管理其他Model**

```python
# admin.py
from django.contrib import admin

from app01.midels import Book

admin.site.register(Book)
```

## 排序OrderingFilter

[OrderingFilter](https://q1mi.github.io/Django-REST-framework-documentation/api-guide/filtering_zh/#orderingfilter)

```python
"""
在类视图中设置filter_backends 使用rest_frameworks.OrderingFilter过滤器 RESTframework会在请求的查询字符串参数中检查是否包含了ordering参数 如果包含了ordering参数 则按照ordering参数指明的排序字段对数据集进行排序

前端可以传递的ordering参数的可选字段值 需要在ordering_fields中声明
"""

from rest_framework.filters import OrderingFilter
from django_filters.rest_framework import DjangoFilterBackend

class Bks(ListAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    # 因为局部配置回覆盖全局配置 所以需要重新把过滤组件核心类再次声明
    # 否则过滤功能会失效
    filter_backends = [OrderingFilter, DjangoFilterBackend]  # 局部配置
    filter_fields = ['title', 'price']
    ordering_fields = ['id', 'price']
    
# http://127.0.0.1:8000/bks/?ordering=-id
# -id 表示针对id字段进行倒叙排序
# id  表示针对id字段进行升序排序
```

## 搜索SearchFilter

[SearchFilter](https://q1mi.github.io/Django-REST-framework-documentation/api-guide/filtering_zh/#searchfilter)

```python
# 使用
from rest_framework.filters import OrderingFilter, SearchFilter

class Bks(ListAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    filter_backends = [SearchFilter]
    search_fields = ['title', '=price']
    
# http://127.0.0.1:8000/bks/?search=8.00

"""
默认情况下 搜索将使用不区分大小写的部分匹配 搜索参数可以包含多个搜索项 其应该是空格和/或逗号分隔
如果使用多个搜索术语 则仅当所有提供的术语都匹配时才在列表中返回对象

可以通过在search_fields前面添加各种字符来限制搜索行为
'^' 以指定内容开始
'=' 完全匹配
'@' 全文搜索(目前只支持Django的MySQL后端)
'$' 正则搜索
"""
```

# 异常处理

```python
# 只要出异常 按照固定的信息返回
# 就算出错 我们也要返回json数据 然后再里面提示error信息
```

## 异常处理部分源码

```python
# dispath方法 做了异常捕获
class APIView(View):
    def dispatch(self, request, *args, **kwargs):
        .../
        try:
            self.initial(request, *args, **kwargs)

            # Get the appropriate handler method
            if request.method.lower() in self.http_method_names:
                handler = getattr(self, request.method.lower(),
                                  self.http_method_not_allowed)
            else:
                handler = self.http_method_not_allowed

            response = handler(request, *args, **kwargs)

        except Exception as exc:
            response = self.handle_exception(exc)

        self.response = self.finalize_response(request, response, *args, **kwargs)
        return self.response


# Exception handling
'EXCEPTION_HANDLER': 'rest_framework.views.exception_handler',
'NON_FIELD_ERRORS_KEY': 'non_field_errors',
    
# exception_handler异常处理源码
from rest_framework.views import exception_handler

def exception_handler(exc, context):
    """
    Returns the response that should be used for any given exception. By default we handle the REST framework `APIException`, and also Django's built-in `Http404` and `PermissionDenied` exceptions. Any unhandled exceptions may return `None`, which will cause a 500 error to be raised.
    """
    if isinstance(exc, Http404):
        exc = exceptions.NotFound()
    elif isinstance(exc, PermissionDenied):
        exc = exceptions.PermissionDenied()

    if isinstance(exc, exceptions.APIException):
        headers = {}
        if getattr(exc, 'auth_header', None):
            headers['WWW-Authenticate'] = exc.auth_header
        if getattr(exc, 'wait', None):
            headers['Retry-After'] = '%d' % exc.wait

        if isinstance(exc.detail, (list, dict)):
            data = exc.detail
        else:
            data = {'detail': exc.detail}

        set_rollback()
        return Response(data, status=exc.status_code, headers=headers)

    return None  # 没有处理的异常返回None 引发500 Error

# raise_uncaught_exception
if response is None:
    self.raise_uncaught_exception(exc)
    
def raise_uncaught_exception(self, exc):
    if settings.DEBUG:
        .../
    raise exc  # 未处理的异常 直接raise
```

## 自定义异常处理方法

```python
"""
自定义异常处理 
  1. 处理drf未处理的异常
  2. 记录错误日志
  
自定义：统一接口的返回
"""

# 自定义异常处理方法
from rest_framework.views import exception_handler
from rest_framework.response import Response
from rest_framework import status


def my_exception_handler(exc, context):
    response = exception_handler(exc, context)
    """
    两种情况
      1. None drf没有处理
      2. Response对象
    """
    if not response:
        return Response(data={'status': 0, 'msg': str(exc)}, status=status.HTTP_400_BAD_REQUEST)
    # return response
    return Response(data={'status':0, 'msg': response.data.get('detail')}, status=response.status_code)

"""
如果需要分得更细 再去捕获判断：
if isinstance(exc, ZeroDivisionError):
 pass
 
# 替换为自定义响应对象
return APIResponse(code=0, msg='error', result=str(exc))
return APIResponse(code=0, msg='error', result=ret.data)
"""
```

# 书籍管理接口详细

## 模型类

```python
from django.db import models


class BaseModel(models.Model):
    is_delete = models.BooleanField(default=False)
    # auto_now_add=True 只要记录创建 不需要手动插入时间 自动把当前时间插入
    create_time = models.DateTimeField(auto_now_add=True)
    # auto_now=True 只要更新 就会把当前时间插入
    last_update_time = models.DateTimeField(auto_now=True)
    # import datetime
    # datetime.datetime.now一定不要加()执行 否则时间都是项目创建的时间
    # create_time = models.DateTimeField(default=datetime.datetime.now)

    class Meta:
        # 单个字段有索引:db_index=True 有唯一:unique=True 直接配在字段参数中
        # 多个字段 有联合索引:index_together 联合唯一:unique_together
        abstract = True  # 抽象表 不在数据库中建立表

        
class Book(BaseModel):
    # help_text admin管理后台的网页提示
    name = models.CharField(max_length=32, verbose_name='书名', help_text='这里填书名')
    price = models.DecimalField(max_digits=8, decimal_places=2)
    # 一对多的关系一但确立 关联字段写在多的一方
    # to_field 默认不屑 关联到主键
    # db_constraint=False 控制是否应该在数据库中为这个外键创建一个约束 默认值是True
    publish = models.ForeignKey(to='Publish', on_delete=models.DO_NOTHING, db_constraint=False)

    # 对对多 跟作者 关联字段写在查询次数多的一方
    # 三种创建方式 根据实际选取选择
    # 第三张表 只有关联字段 用自动创建
    # 第三张表 有扩展字段 需要手动写
    authors = models.ManyToManyField(to='Author', db_constraint=False)
    class Meta:
        verbose_name_plural = '书籍'  # admin中显示

    def __str__(self):
        return self.name

    @property
    def publish_name(self):
        return self.publish.name

    def author_list(self):
        return [{'name': author.name, 'gender': author.get_gender_display()} for author in self.authors.all()]
    

class Publish(BaseModel):
    name = models.CharField(max_length=32)
    addr = models.CharField(max_length=32)

    def __str__(self):
        return self.name
    

class Author(BaseModel):
    name = models.CharField(max_length=32)
    gender = models.IntegerField(choices=((1, '男'), (2, '女'), (3, '其他')))
    # 一对一关系 写在查询频率高的以方
    # OneToOneField的本质就是ForeignKey+unique=True
    authordetail = models.OneToOneField(to='AuthorDetail', db_constraint=False, on_delete=models.CASCADE)

    
class AuthorDetail(BaseModel):
    phone = models.CharField(max_length=11)


"""
# on_delete
models.CASCADE
  删除关联数据 与之关联也删除
models.DO_NOTHING
  删除关联数据 引发错误IntegrityError
  不采取任何行动 如果你的数据库后端强制执行引用完整性 这将导致一个 IntegrityError
models.PROTECT
  删除关联数据 引发错误ProtectedError
models.SET_NULL
  删除关联数据 与之关联的值设置null(前提是FK字段需要设置可为空)
models.SET_DEFAULT
  删除关联数据 与之关联的值设置为默认值(前提FK字段需要设置默认值)
models.SET
  删除关联数据
  1. 与之关联的值设置为指定值 设置：models.SET(值)
  2. 与之关联的值设置为可执行对象的返回值 设置：models.SET(可执行对象)
  
  
# 表断关联
  1. 表之间没有外键关联 但是有外键逻辑关联(有外键字段 没有设置约束) db_constraint
  2. 断关联后不会影响数据库查询效率 但是会极大的提高数据库增删改效率(不影响增删改查操作)
  3. 断关联一定要通过逻辑保证表之间数据的安全 不要出现脏数据(代码控制)
  4. 断关联
  5. 级联关系
     作者没了 详情也没：on_delete=models.CASCADE
     出版社没了 书还是哪个出版社出版：on_delete=models.DO_NOTHING
     部门没了 员工没有部门(空部门)：null=True, on_delete=models.SET_NULL
     部门没了 员工进入默认部门(默认值)：default=0, on_delete=models.SET_DEFAULT
"""
```

## CBV视图

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from . import models
from . import serializers


class BookAPIView(APIView):
    def get(self, request, *args, **kwargs):
        # 查所有
        books = models.Book.objects.all().filter(is_delete=False)
        serializer = serializers.BookModelSerializer(books, many=True)
        return Response(serializer.data)
        # 查一个 取pk实现

    def post(self, request, *args, **kwargs):
        """具备增单条 和增多条的功能"""
        if isinstance(request.data, dict):
            # 增单条
            serializer = serializers.BookModelSerializer(data=request.data)
            serializer.is_valid(raise_exception=True)
            serializer.save()
            return Response(serializer.data)
        elif isinstance(request.data, list):
            # 增多条 现在serializer类型为：ListSerialzer
            serializer = serializers.BookModelSerializer(data=request.data, many=True)
            print(type(serializer))
            serializer.is_valid(raise_exception=True)
            serializer.save()
            """
            新增 -> ListSerializer.save() -> create()
            def create(self, validated_data):
                return [self.child.create(attrs) for attrs in validated_data]
            增多条：循环新增
            self.child: BookModelSerialzer
            """
            return Response(serializer.data)

    def put(self, request, *args, **kwargs):
        # 改一个
        if kwargs.get('pk', None):
            book = models.Book.objects.filter(pk=kwargs.get('pk')).first()
            # partial=True允许局部更新
            serializer = serializers.BookModelSerializer(book, data=request.data, partial=True)
            serializer.is_valid(raise_exception=True)
            serializer.save()
            return Response(serializer.data)
        else:
            # 修改多个 需要新建一个BookListSerializer 并重写 update()方法
            # 规定数据格式：request.data: [{id:1, name=xx, price:xxx}, {id:2, name=xx, price:xxx},]
            # 处理传入的数据：对象列表[book1, book2] 修改的数据列表：[{name:xx,price:xx}, {}]
            # 方案1：for循环一个个修改
            # 方案2：重写ListSerializer的update方法
            book_list = []
            modify_data = []
            for item in request.data:
                pk = item.pop('id')
                book = models.Book.objects.get(pk=pk)
                book_list.append(book)
                modify_data.append(item)
            # 第一种方案 for循环实现
            # for k, v in enumerate(modify_data):
            #     serializer = serializers.BookModelSerializer(book_list[k], data=v)
            #     serializer.is_valid(raise_exception=True)
            #     serializer.save()  # 调ListSerializer的update方法 改到自己的
            # return Response('succ')

            # 第二种方案 重写ListSerializer的update()
            serializer = serializers.BookModelSerializer(instance=book_list, data=modify_data, many=True)
            serializer.is_valid(raise_exception=True)
            serializer.save()  # 调ListSerializer的update方法 改到自己的
            return Response(serializer.data)

    def delete(self, request, *args, **kwargs):
        # 单个删除 和批量删除
        pk = kwargs.get('pk')
        pks = []
        if pk:
            # 单条删除
            pks.append(pk)
        else:
            # 不管单条删除还是多条删除 都用多条删除
            # 多条删除 {'pks': [1, 2, 3]}
            pks = request.data.get('pks')
        # 把id_delete设置为True
        # res返回受影响的行数
        res = models.Book.objects.filter(pk__in=pks, is_delete=False).update(is_delete=True)
        if res:
            return Response(data={'msg':'删除成功 删除%s条数据' % res})
        return Response(data={'msg':'没有要删除的数据'})
```

## 序列化器

```python
from rest_framework import serializers
from . import models


# 写一个类 继承ListSerializer 重写update
class BookListSerialzier(serializers.ListSerializer):
    def create(self, validated_data):
        return super().create(validated_data)

    def update(self, instance, validated_data):
        print(instance)
        print(validated_data)
        # 保存数据
        # self.child: 是BookModelSerializer对象
        # return [self.child.update(对象, 字典) for attrs in validated_data]
        return [self.child.update(instance[i], attrs) for i, attrs in enumerate(validated_data)]


# 如果序列化的是数据库的表 尽量用ModelSerializer
class BookModelSerializer(serializers.ModelSerializer):
    # 关联字段显示name
    # 第一种方案：source(只序列化可以 反序列化有问题)
    # publish = serializers.CharField(source='publish.name')
    # 第二种方案：models中写方法 也可以使用SerializerMethodField字段
    class Meta:
        # many=True的时候 用这个类实例化(源码)
        list_serializer_class = BookListSerialzier
        model = models.Book
        # fields = '__all__'
        # 联表嵌套深度 用得少
        # depth = 0
        # fields = ('name', 'price', 'authors', 'publish')
        fields = ('id', 'name', 'price', 'authors', 'author_list', 'publish', 'publish_name')
        read_only_fileds = ('publish_name', 'author_list')
        extra_kwargs = {
            'publish': {'write_only': True},
            'authors': {'write_only': True}
        }
```

### 序列化

```python
"""
序列化输出结果
由于设置 read_only 和 write_only字段
序列化只展示read_only字段 不展示write_only字段
反序列化需要给write_only字段 不需要给read_only字段
"""
[
    {
        "name": "Python",
        "price": "131.88",
        "author_list": [
            {
                "name": "minho",
                "gender": "男"
            },
            {
                "name": "kimi",
                "gender": "女"
            }
        ],
        "publish_name": "南京出版社"
    },
    {
        "name": "js",
        "price": "78.00",
        "author_list": [
            {
                "name": "kimi",
                "gender": "女"
            }
        ],
        "publish_name": "东京出版社"
    }
]

# source参数可以更改关系字段显示名字 
# 但是反序列化会报错
The `.create()` method does not support writable dotted-source fields by default.
Write an explicit `.create()` method for serializer `api.serializers.BookModelSerializer`, or set `read_only=True` on dotted-source serializer fields.
```

### 反序列化

```python
"""
反序列化传入json数据 提交post
注意：此时反序列化关系字段为 authors[id] 和 publish(id)
"""
{
    "name": "数据结构",
    "price": "99.00",
    "authors": [2],
    "publish": 2
}
```

## 总结

```python
# book其实是5个表(自动生成多对多关系表)
  - 一对一关系 其实是ForeignKey uniqe=True
  - on_delete: 级联删除 设置为空 什么都不干 设置成默认值
  - 联合索引 联合唯一
  - 日期类型参数：auto_now_add 和 auto_now
  - 基表：abstract

# book
  - 单条查询 批量查询(判断kwargs.get('pk')处理)
  - 单条新增 批量新增(生成序列化对象 many=True -> ListSerializer -> 循环掉self.child.save)
  - 单条修改 批量修改(自定义BookListSerializer 继承ListSerializer: 重写update方法 因为ListSerizlizer没有实现)
           class Meta:
               # many=True的时候 用这个类实例化(源码)
               list_serializer_class = BookListSerialzier  # 关联自定义BookListSerializer
  - 单条删除 批量删除(is_delete) 同一用批量删除 pk__in=[1, 2, 3]
```

# 分页器

**drf内置三个分页器**

```python
from rest_framework.pagination import PageNumberPagination
from rest_framework.pagination import LimitOffsetPagination
from rest_framework.pagination import CursorPagination

"""
def默认配置
  DEFAULT_PAGINATION_CLASS: None  # 默认不分页
  PAGE_SIZE: None  # 每页显示数目 配置分页需要配置此参数才能生效
"""

# settings.py
REST_FRAMEWORK = {
    ...
    'PAGE_SIZE': 2,
}

class BookView(ListAPIView):
    # 局部配置分页
    pagination_class = PageNumberPagination
    queryset = models.Book.objects.all()
    serializer_class = serializers.BookModelSerializer
        
# 如何使用APIView分页
class BookView(APIView):
    def get(self, request, *args, **kwargs):
        # 实例化得到一个分页器对象
        books = models.Book.objects.all()
        page_cursor = MyPageNumberPagination()
        # paginate_queryset进去筛选数据
        books = page_cursor.paginate_queryset(books, request, view=self)
        # 得到上一页下一页的url
        next_url = page_cursor.get_next_link()
        prev_url = page_cursor.get_previous_link()
        print(next_url)
        print(prev_url)
        serializer = serializers.BookModelSerializer(books, many=True)
        return Response(serializer.data)
```

## PageNumberPagination

```python
# 页码分页 基本分页
# 继承 并配置4个重要参数
class MyPageNumberPagination(PageNumberPagination):
    page_size = 2                   # 默认每页显示数量
    page_query_param = 'aaa'        # /api/books2/?aaa=2 查询key
    page_size_query_param = 'size'  # /api/books2/?aaa=2&size=5 每页显示条数
    max_page_size = 5               # 每页最大显示数量 配合上一个参数
```

## LimitOffsetPagination

```python
# 偏移分页
class MyLimitOffsetPagination(LimitOffsetPagination):
    default_limit = 3              # 默认每页显示数量
    limit_query_param = 'limit'    # 每页数量
    offset_query_param = 'offset'  # 偏移位置
    max_limit = 200                # limit最大可以设置数量
```

## CursorPagination

```python
# 效率高 只能选择往前走 或者往后走(只有上一页下一页)
class MyCursorPagination(CursorPagination):
    cursor_query_param = 'cursor'  # 查询参数
    page_size = 3                  # 每页显示的条数
    ordering = '-id'               # 排序字段 默认-created 没有该字段需要修改配置
```

# 自动生成接口文档

```python
# drf可以自动帮助我们生成接口文档
# 接口文档以网页的方式呈现
# 自动接口文档生成的是继承自 APIView 及其子类的视图 ** 

# 额外了解：swagger
```

## 安装coreapi

```python
# drf生成接口文档需要coreapi库的支持
pip install coreapi
```

## 路由配置

```python
# 配置路由
from django.urls import path
from rest_framework.documentation import include_docs_urls

urlpatterns = [
    ...
    path('docs/', include_docs_urls(title='API文档'))
]

# 报错: AttributeError: 'AutoSchema' object has no attribute 'get_link'
# 在settings.py添加配置
REST_FRAMEWORK = {
    'DEFAULT_SCHEMA_CLASS': 'rest_framework.schemas.coreapi.AutoSchema',
}
```

## 文档描述说明的定义位置

### 单一方法的视图

```python
# 单一方法的视图 可以直接使用类视图的文档字符串
class BookListView(ListAPIView):
    """返回所有图书信息"""
```

### 多个方法的视图

```python
# 包含多个方法的视图 在类视图的文档字符串中 分开方法定义
class BookListView(ListAPIView):
    """
    get:
    返回所有图书信息
    
    post:
    新增图书
    """
```

### 视图集ViewSet

```python
# 对于视图集 仍在类视图的文档字符串中分开定义 但是应该使用action名称区分
# 视图及ViewSet中的retrieve名称 在接口文档网站中叫做read
# 参数的Description需要在模型类或序列化类的字段中以help_text选项定义
# verbose_name help_text
class BookListView(ModelViewSet):
    """
    list:
    返回图书列表数据
    
    retrieve:
    返回图书详情数据
    
    latest:
    返回最新的图书数据
    
    read:
    修改图书的阅读量
    """
```

# JWT认证

[JsonWebToken入门教程](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)

```python
在用户注册或登录后 我们想记录用户的登录状态 或者为用户创建身份认证的凭证 我们不在使用Session认证机制 而是用Json Web Token(本质就是token)认证机制

"""
JsonWebToken 是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准(RFE 7519) 该token被设计为紧凑且用于分布式站点的单点登录(SSO)场景 
JWT的声明一般被用来在身份提供者和服务者间传递被认证的用户身份信息 以便于从资源服务器获取资源 也可以额外的增加一些其他业务逻辑所必须的声明信息 该token页可以直接被用于认证 也可被加密

不能被篡改 但是你可以拿出来用 只要没有你过期 -> 爬虫
"""
```

## JWT数据结构

```python
# 三段信息
Header.Payload.Signature

"""
Header    -  头部
Payload   -  负载
Signature - 签名
"""

# 原理
"""
1. JWT分三段式：Header.Payload.Signature
2. 头和体是可逆加密 让服务器可以反解出user对象 签名是不可逆加密 保证整个token的安全性
3. 头体签名有三部分 都是采用json格式的字符串进行加密 可逆加密一般采用base64算法 不可逆加密一般采用hash(md5)算法
4. Header中的内容是基本信息：公司信息 项目组信息 token采用的加密方式信息
5. Payload中的内容是关键信息：用户主键 用户名 签发时客户端信息(设备号 地址) 过期时间
6. Signature中的内容时安全信息：头的加密结果 + 体的加密结果 + 服务器不对外公开的安全码进行加密
"""
```

### Header

```python
# Header 部分是一个JSON 对象 描述JWT的元数据 通常是下面的样子:
{
  "alg": "HS256",
  "typ": "JWT"
}

"""
上面代码中 
  alg属性表示签名的算法(algorithm) 默认是 HMAC SHA256(写成 HS256)
  typ属性表示这个令牌(token) 的类型(type) JWT令牌统一写为JWT

  最后 将上面的 JSON对象使用 Base64URL算法 转成字符串
"""
```

### Payload

```python
# Payload部分也是一个JSON对象 用来存放实际需要传递的数据 JWT规定了7个官方字段 供选用
iss (issuer)：签发人
exp (expiration time)：过期时间
sub (subject)：主题
aud (audience)：受众
nbf (Not Before)：生效时间
iat (Issued At)：签发时间
jti (JWT ID)：编号

# 除了官方字段 你还可以在这个部分定义私有字段 比如：
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}

"""
注意 JWT默认是不加密的 任何人都可以读到 所以不要把秘密信息放在这个部分
这个JSON对象也要使用 Base64URL算法 转成字符串
"""
```

### Signature

```python
"""
Signature 部分是对前两部分的签名 防止数据篡改

  首先 需要指定一个密钥(secret) 这个密钥只有服务器才知道 不能泄露给用户
  然后 用Header里面指定的签名算法(默认是 HMAC SHA256) 按照下面的公式产生签名
"""
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)

# 算出签名以后 把Header Payload Signature三个部分拼成一个字符串 每个部分之间用"点"(.)分隔 就可以返回给用户
```

### Base64URL

```python
"""
前面提到 Header和Payload串型化的算法是 Base64URL 这个算法跟Base64算法基本类似 但有一些小的不同

JWT 作为一个令牌(token) 有些场合可能会放到URL
  比如 api.example.com/?token=xxx
  
Base64有三个字符 -> + / =  在URL里面有特殊含义 所以要被替换掉 
  =被省略
  +替换成-
  /替换成_ 
  
这就是 Base64URL算法
"""
```

### 检验

```python
"""
1. 将token按 . 拆分为三个字符串 第一段：头加密字符串 一般不需要做任何处理
2. 第二段 体加密字符串 要反解出用户主键 通过主键从User表中就能得到登录用户 过期时间和设备信息都是安全信息 确保token没有过期 且是同一设备来的
3. 再用第一段 + 第二段 + 服务器安全码 不可逆hash(md5等)加密 与第三段签名字符串进行碰撞校验 通过后才能代表第二段校验得到的user对象就是合法的登录用户
"""
```

## DRF项目的JWT认证开发流程

```python
"""
1. 用账号密码访问登录接口 登录接口逻辑中掉用签发token算法 得到token 返回给客户端 客户端自己存到cookies中

2. 校验token的算法应该写在认证类(在认证类中调用) 全局配置给认证组件 所有视图函数类请求 都会进行认证校验 所以请求带token过来 就会反解出user对象 在视图类中用request.user就能访问登录的用户

注：登录接口需要做 认证 + 权限 两个局部禁用
"""
```

### 第三方JWT的使用

```python
# 官网
https://jpadilla.github.io/django-rest-framework-jwt/
    
# 第三方写好的
pip install djangorestframework-jwt

# 使用
"""
1. 基于django的Auth模块
2. 使用自己的认证模块
"""
```

#### 继承DjangoUser表扩展

```python
"""
继承django 原生的User表 最好一开始就使用
"""
# 继承AbstractUser表 扩展字段
from django.db import models
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    phone = models.CharField(max_length=11)
    icon = models.ImageField(upload_to='icon', default='icon/default.png')  # ImageField依赖pillow模块
    
# settings.py配置
# 扩展django自带的user表 
AUTH_USER_MODEL = 'api.User'
    
# 配置头像相关
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
# 开发MEDIA文件
from django.views.static import serve  # django内置给你的一个视图函数 类似默认开了static静态文件的路由
from django,urls import re_path
from django.conf import settings  # 以后取配置文件都用这个 内置 比较全

urlpatterns = [
    re_path('media/?P<path>.*', serve, {'document_root': settings.MEDIA_ROOT})
]

# 执行数据库迁移
# 创建超级用户 进行测试

# 国际化
LANGUAGE_CODE = 'zh-Hans'
TIME_ZONE = 'Asia/Shanghai'
USE_I18N = True
USE_L10N = True
USE_TZ = False  # 时区
```

#### 登录认证简单使用

```python
from rest_framework_jwt.views import JSONWebTokenAPIView  # 基类 继承APIView

# 都继承JSONWebTokenAPIView
from rest_framework_jwt.views import ObtainJSONWebToken
from rest_framework_jwt.views import RefreshJSONWebToken
from rest_framework_jwt.views import VerifyJSONWebToken


# 1. urls.py配置路由即可简单快速使用登录接口获取jwt token
from django.contrib import admin
from django.urls import path
from rest_framework_jwt.views import ObtainJSONWebToken

urlpatterns = [
    path('admin/', admin.site.urls),
    path('login/', ObtainJSONWebToken.as_view()),  # 正确登录获取jwt token
]


# 2. views.py 局部使用jwt认证
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework_jwt.authentication import JSONWebTokenAuthentication


class BookBiew(APIView):
    authentication_classes = [JSONWebTokenAuthentication]  # 局部配置jwt认证

    def get(self, request):
        print(request.user, request.user.email)
        print(request.auth)
        return Response('ok')


# 3. 在请求headers带认证参数 否则为匿名用户AnonymousUser(return None)
Authorization JWT jwt_value

"""
注意：这个时候 如果请求headers中不带认证参数 源码直接return None 相当于匿名用户AnonymousUser访问
     这个时候 如果视图函数允许匿名用户访问 那么此时不带jwt token也可以访问成功
     
可以通过通过认证类：JSONWebTokenAuthentication 和权限类：IsAuthenticated 来控制用户登录以后才能访问某些接口
如果用户不等录就可以访问 只需要把权限类：IsAuthenticated去掉即可
"""

# 4. 禁止匿名用户访问
from rest_framework.permissions import IsAuthenticated

class BookBiew(APIView):
    authentication_classes = [JSONWebTokenAuthentication]
    """配置权限 IsAuthenticated 只允许认证通过的用户访问"""
    permission_classes = [IsAuthenticated]

    def get(self, request):
        return Response('ok')
```

#### 自定制JWT认证去除headers前缀

```python
# 自定义重写jwt认证类 重写get_jwt_value方法 去掉认证前缀 JWT token.xx.xx
from rest_framework import HTTP_HEADER_ENCODING
from rest_framework_jwt.authentication import JSONWebTokenAuthentication

class MyToken(JSONWebTokenAuthentication):
    def get_jwt_value(self, request):
        auth = request.META.get('HTTP_AUTHORIZATION', b'')
        if isinstance(auth, str):
            auth = auth.encode(HTTP_HEADER_ENCODING)
        return auth
    
# 视图函数局部配置
authentication_classes = [MyToken]

# 除此之外 还可以修改前缀 settings.py
JWT_AUTH = {
    # 'JWT_AUTH_HEADER_PREFIX': 'JWT',  JWT自带配置 认证请求headrs需要带前缀
    'JWT_AUTH_HEADER_PREFIX': 'aaa',  # aaa token.xx.xx
}
```

#### 手动签发JWT

```python
# 可以拥有原生登录基于Model类user对象签发JWT
from rest_framework_jwt.settings import api_settings

jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER

palyload = jwt_payload_handler(user)  # 把user传入 得到payload
token = jwt_encode_handler(palyload)  # 把payload传入 得到token

# 把三段信息解析出payload 判断认证是否篡改 是否过期
# 通过payload转成user对象
payload = jwt_decode_handler(jwt_value)
```

#### 登录接口返回数据格式

```python
"""
控制登录接口返回的数据格式
  第一种方案：自己写登录接口
  第二种方案：用内置
"""

# jwt的配置响应数据格式
'JWT_RESPONSE_PAYLOAD_HANDLER': 'rest_framework_jwt.utils.jwt_response_payload_handler'
 
# jwt_response_payload_handler源码
# 可以重写该方法 控制返回数据的格式(返回什么 前端登录接口就能看到什么)
def jwt_response_payload_handler(token, user=None, request=None):
    return {
        'token': token
    }

"""自定义控制登录jwt登录接口返回数据格式"""
# 1. 重写jwt_response_payload_handler方法
def jwt_response_payload_handler(token, user=None, request=None):
    return {
        'token': token,
        'msg': '登录成功',
        'status': 100,
        'username': user.username
    }

# 2. 配置 settings.py
JWT_AUTH = {
    # 'JWT_AUTH_HEADER_PREFIX': 'JWT',  JWT自带配置 认证请求headrs需要带前缀
    'JWT_AUTH_HEADER_PREFIX': 'aaa',
    'JWT_RESPONSE_PAYLOAD_HANDLER': 'api.utils.jwt_response_payload_handler',  # JWT控制返回数据格式
}
```

**返回结果示例**

```python
{
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6Im1pbmhvIiwiZXhwIjoxNjE0ODMwNzgzLCJlbWFpbCI6Ijk3NDMxMTEwQHFxLmNvbSJ9.90l3F4oTT0gWfcgdc0LH9euZYygyWinHF4Ec-kwra-U",
    "msg": "登录成功",
    "status": 100,
    "username": "minho"
}
```

### 基于JWT自定制认证类

#### 基于BaseAuthentication

```python
import jwt
from rest_framework.authentication import BaseAuthentication
from rest_framework.exceptions import AuthenticationFailed
from rest_framework_jwt.utils import jwt_decode_handler
from api import models

class MyJwtAuthentication(BaseAuthentication):
    def authenticate(self, request):
        jwt_value = request.META.get('HTTP_AUTHORIZATION')
        if jwt_value:
            try:
                # jwt提供了三段token 取出用payload的方法 并且有校验功能
                payload = jwt_decode_handler(jwt_value)
            except jwt.ExpiredSignature:
                raise AuthenticationFailed('签名过期')
            except jwt.InvalidTokenError:
                raise AuthenticationFailed('用户非法')
            except Exception as exc:
                raise AuthenticationFailed(str(exc))
            # 因为payload就是用户信息字典
            # print(payload)
            # return payload, jwt_value
            # 需要得到user对象
            # 1. 去数据库查
            # user = models.User.objects.get(pk=payload.get('user_id'))
            # 2. 不查库自己转 速度块一些
            user = models.User(id=payload['user_id'], username=payload['username'])
            return user, jwt_value

        # 没有带认证信息 直接抛异常
        raise AuthenticationFailed('你没有携带认证信息')
```

#### 基于BaseJSONWebTokenAuthentication

```python
import jwt
from rest_framework.exceptions import AuthenticationFailed
from rest_framework_jwt.authentication import BaseJSONWebTokenAuthentication
from rest_framework_jwt.utils import jwt_decode_handler

class MyJwtAuthentication(BaseJSONWebTokenAuthentication):
    def authenticate(self, request):
        jwt_value = request.META.get('HTTP_AUTHORIZATION')
        if jwt_value:
            try:
                # jwt提供了三段token 取出用payload的方法 并且有校验功能
                payload = jwt_decode_handler(jwt_value)
            except jwt.ExpiredSignature:
                raise AuthenticationFailed('签名过期')
            except jwt.InvalidTokenError:
                raise AuthenticationFailed('用户非法')
            except Exception as exc:
                raise AuthenticationFailed(str(exc))
            # 继承BasejsonWeb 直接调用下面方法获取user对象
            user = self.authenticate_credentials(payload)
            return user, jwt_value

        # 没有带认证信息 直接抛异常
        raise AuthenticationFailed('你没有携带认证信息')
```

### 多方式登录

```PYTHON
# 使用用户名 手机号 邮箱 都可以登录
# 前端需要传的数据格式
{
    "username": "lqz/13334449988/123@qq.com",
    "password": "123456"
}

# serializers.py
import re
from rest_framework import serializers
from rest_framework.exceptions import ValidationError
from rest_framework_jwt.settings import api_settings
from api import models

jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER

class LoginModelSerialzier(serializers.ModelSerializer):
    username = serializers.CharField()
    class Meta:
        model = models.User
        fields = ['username', 'password']

    def validate(self, attrs):
        # 在这写逻辑
        username = attrs.get('username')  # 用户名有三种方式
        password = attrs.get('password')
        # 通过判断username数据不同 查询字段不一样
        # 正则匹配 如果是手机号
        if re.match('^1[3-9][0-9]{9}$', username):
            user = models.User.objects.filter(phone=username).first()
        elif re.match('^.+@.+$', username):  # 邮箱
            user = models.User.objects.filter(email=username).first()
        else:
            user = models.User.objects.filter(username=username).first()
        if user:  # 存在用户
            # 校验密码 因为是密文 要用check_password
            if user.check_password(password):
                # 签发token
                payload = jwt_payload_handler(user)  # user传入 得到payload
                token = jwt_encode_handler(payload)  # payload传入 得到token
                # self.token = token  推荐使用context
                self.context['token'] = token
                self.context['username'] = user.username
                return attrs
            else:
                raise ValidationError('密码错误')
        raise ValidationError('用户不存在')

# views.py
from rest_framework.viewsets import ViewSet
from api import serializers

# class Login2View(ViewSetMixin, APIView):
class Login2View(ViewSet):
    """这是登录接口"""
    # def post(self, request):  # 不写post 直接写login?
    #     pass

    def login(self, request, *args, **kwargs):
        # 1. 需要有个序列化的类
        # 2. 生成序列化类对象
        serializer = serializers.LoginModelSerialzier(data=request.data)
        # 3. 调用序列化对象的is_valid
        serializer.is_valid(raise_exception=True)
        token = serializer.context.get('token')
        # 4. return
        return Response({'status':100, 'msg':'success', 'token': token, 'username': serializer.context.get('username')})

# urls.py
path('login2/', views.Login2View.as_view({'post':'login'}))
```

### 配置过期时间

```python
# 过期时间
'JWT_EXPIRATION_DELTA': datetime.timedelta(seconds=300)
    
# 配置 settings.py
import datetime

JWT_AUTH = {
    # 过期时间手动配置
    'JWT_EXPIRATION_DELTA': datetime.timedelta(days=7),
}
```

# RBAC

```python
# RBAC： 基于角色的访问控制 用在公司内部系统居多
# Django的Auth体系 就是内置了一套RBAC的权限系统

# 权限三表
User    Group    Permisson
    多对多     多对多
    
# 权限六表
User    Group    Permisson
    UG关系表    GP关系表
正常5个就够：Django多一个 UP关系表 单独直接给每一个人加权限
```

## DjangoAuth 权限6表

```python
"""
# 后台的权限控制(公司内部系统 crm/erp/协同平台)
  auth_user        # 用户表
  auth_group       # 用户组
  auth_permission  # 权限表

  auth_user_groups            # 用户 组 关系表 多对多
  auth_group_permissions      # 组 权限 关系表 多对多
  auth_user_user_permissions  # 用户 权限 关系表 多对多

# 前台(主站) 需要用三大认证
  - 认证
  - 权限
  - 频率
  
后台管理用户 和前台注册用户 可以是两个user表 也可以共用一个
"""
```
