---
title: Django的URL处理
date: 2018-12-01 19:08:21
tags: 
  - Django2.0
category: 学习笔记
---


在创建 django 项目时其中默认会生成一个 urls.py 的文件，django 将 url模块化都集中在一个 py 文件中处理。

<!-- more -->

下面是 urls.py 中的注释说明，简单的介绍了 URL 的配置。

```
url_params_demo URL Configuration

The `urlpatterns` list routes URLs to views. For more information please see:
https://docs.djangoproject.com/en/2.0/topics/http/urls/
Examples:
Function views
		1. Add an import:  from my_app import views
		2. Add a URL to urlpatterns:  path('', views.home, name='home')
	Class-based views
		1. Add an import:  from other_app.views import Home
		2. Add a URL to urlpatterns:  path('', Home.as_view(), name='home')
	Including another URLconf
		1. Import the include() function: from django.urls import include, path
		2. Add a URL to urlpatterns:  path('blog/', include('blog.urls'))
```

## Django如何处理请求

当浏览器发过来一个请求时，django 会根据 setting.py 文件中 ROOT_URLCONF 设置来寻找匹配。django 会加载设置中的模块，并找到 urlpatterns 变量，这个变量是一个 python 列表保存着 django.urls.path() 和 django.urls.re_path() 的实例。一旦其中一个URL 与请求匹配，就会调用给定的视图。

```
 # settings.py
ROOT_URLCONF = 'url_params_demo.urls'

 # urls.py
urlpatterns = [
    path('admin/', admin.site.urls),
]
```

## url传参

浏览器传过来的请求可能是多种多样的，如果 URL 携带着参数，django 的 url 如何接收， 

​1.在 url 中使用变量：
  用一个变量接收然后用尖括号包裹起来（变量名可以自己定义，当然要符合 python 变量名规则），这个变量保存的值就可以传递到视图中进行处理。视图函数中的参数必须和url中的参数名称保持一致，不然就找不到这个参数。另外，url中可以传递多个参数。

```
 # urls.py
urlpatterns = [
    path('index/<nums>/', view.index),
    ]
```

​2.采用查询字符串的方式：
   如果浏览器是以查询字符串的方式传递过来的参数，就不需要在 url 中用变量接收，只需要在视图函数中使用如方式获取
```
    request.GET.get('参数名称')
```

3.指定默认参数:
   在url 中指定一个默认参数，视图使用关键字参数的方式接收。还可以通过转换器对默认参数进行类型限制。如场景，有些内容显示默认读取第一页，如果用户请求其他页面可以再传递参数。
```
   # urls.py
   urlpatterns = [
       path('blog/', views.page),
       path('blog/page<int:num>', views.page)
   ]

   # views.py
   def page(request, num=1):
       ......
```


### url 参数转换器：

  可以通过参数转换器，对 url 中的参数类型进行限制，以下是内置转换器，也可以自己定制。
```
   path('blog/page<int:num>', views.page)
```

url参数的转换器：
1. str：除了斜杠`/`以外所有的字符都是可以的。
2. int：只有是一个或者多个的阿拉伯数字。
3. path：所有的字符都是满足的。
4. uuid：只有满足`uuid.uuid4()`这个函数返回的字符串的格式。
5. slug：英文中的横杆或者英文字符或者阿拉伯数字或者下划线才满足。

### 自定义url 转换器
 当内置的转换器不能满足需求的时候，可以自定义转义器，主要分五步:
  1、定义一个类。
  2、在类中定义一个属性regex，这个属性是用来保存url转换器规则的正则表达式。
  3、实现to_python(self,value)方法，这个方法是将url中的值转换一下，然后传给视图函数的。
  4、实现to_url(self,value)方法，这个方法是在做url反转的时候，将传进来的参数转换后拼接成一个正确的url。
  5、将定义好的转换器，注册到django中。
```
# converter.py
from django.urls import register_converter

# 定义类
class CategoryConverter(object):
    # 参数检验规则
    regex = r'\w+|(\w+\+\w+)+'

    def to_python(self,value):
        # python+django+flask
        # ['python','django','flask']
        result = value.split("+")
        return result

    def to_url(self,value):
        # value：['python','django','flask']
        # python+django+flask
        if isinstance(value,list):
            result = "+".join(value)
            return result
        else:
            raise RuntimeError("转换url的时候，分类参数必须为列表！")

# 注册转换器
register_converter(CategoryConverter,'cate')
```
定义好了转换器，还要注意在项目中要执行这段代码才行，在每个 app模块中都有一个 '__init__.py' 的文件，每个模块在被导入的时候，这个文件都会被最先执行。所有只需要在这里导入转换器这个文件就行了。
```
from . import converter
```


## urls 模块化

如果项目越来越大，那么 url 会越来越多因为都放在urls.py 中，这样就不太好管理，因此就可以将每个 app 自己的 url 放到自己的 app 中管理，也就是在app 中 新建一个 urls.py 文件来存储和这个 app 相关的url。
```
    # appname/urls.py
    from django.urls import path

    urlpatterns = [
        path('book', books.urls),
    ]
```

在这个文件的 url 是和主 urls.py 中的 url 进行拼接的，所以不必多加斜杠。在主 urs.py 中的 url 使用 include 函数进行跳转

```
    # urls.py
    path('', include('addname.urls')) 
```

### url 命名

在项目开发中 url 是经常变化的，如果把 url 写死了就经常需要改代码，以后要引用的时候就直接用名字。

```
    urlpatterns = [
        path('', views.index, name='index'),
        path('login/', views.login, name='login')
    ]
```

### 应用命名空间：

在多个 app 之间有可能产生同名的url，这时候为了避免反转 url 的时候产生混淆，可以是应用命名空间区分不同的 app 。应用命名空间只需要在 app 的 urls.py 中定义一个 app_name 的变量来指定这个应用的命名空间名。

```
    # app/urls.py

    app_name = 'book'

    urlpatterns = [
        path('', views.index, name='index'),
        path('login/', views.login, name='login')
    ]
```

在做 url 反转的时候就可以使用应用命名空间反转，
```
    login_url = reverse('book:login')
```

### 实例命名空间：

一个 app 可以创建多个实例，可以使用多个 url 映射同一个 app ，所以这样就会产生一个问题，以后再做反转的时候如果使用应用命名空间，那么就会发生混淆，为了避免这个问题，我们可以使用实例命名空间。实例命名空间只需要在include 函数中传递一个 namesapce 变量即可。

```
    # urls.py
    urlpatterns = [
        path('admin/', admin.site.urls),
        path('book1/', include('book.urls')),
        path('book2/', include('book.urls')),
    ]

	# app/urls.py
    app_name = 'book'

    urlpatterns = [
        path('', views.index, name='index'),
        path('login/', views.login, name='login')
    
    # app/viws.py
    def index(request):
        username = request.GET.get('username')
        if username:
            return HttpResponse('书籍首页')
        else:
            # 获取当前的实例命名空间
            current_namespace = request.resolver_match.namespace
            return redirect(reverse("%s:login" % current_namespace))


    def login(request):
        return HttpResponse('书籍登录页面')
```

实例命名空间需要注意，在反转的时候可以先获取到实例命名：
```
    current_namespace = request.resolver_match.namesapce
    return redirect(reverse('%s:login'% current_namespace))
```

## reverse函数url反转

1、如果在url 反转的时候需要添加参数，那么可以传递kwargs 参数到 reverse 函数中

```
    detail_url = reverse('detail', 'kwargs={'article_id':1, 'page':2}')
```

2、如果想要添加的参数是查询字符串的参数，则必须手动的进行拼接

```
    login_url = reverse('login') + '?next=/
```


## include 函数用法

1.include(module, namespace=None):

​       *module: 子 url 的模块名字。

​       *namespace: 实例命名空间，如果使用实例命名空间，前提是必须先指定应用命名空间，也就是在子 urls 中添加变量 app_name。

2.include((pattern_list, app_namespace), namespace=Nonw): 

include 函数的第一参数既可以是一个字符串，也可以是一个元组，元组的第一个参数是一个列表，在这个列表中可以是多个 path 或者 re_path 函数。元组的第二个参数就是应用命名空间。也就是所说可以不采用在 app 中创建子 urls.py 的方式来区分各个 app  的 url 了。用元组的方式将子 urls 的url 作为列表放到 include 的元组参数中，app 的应用命名作为元组的第二个参数。

```
    from book import views

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('book/', include(([
            path('detail/', views.detail),
            path('list/', views.book_list)
                        ], 'book'), namespace='book')),
    ]
```


## re_path 函数：

1. re_path和path的作用都是一样的。只不过`re_path`是在写url的时候可以用正则表达式，功能更加强大。
2. 写正则表达式都推荐使用原生字符串。也就是以`r`开头的字符串。
3. 在正则表达式中定义变量，需要使用圆括号括起来。这个参数是有名字的，那么需要使用`?P<参数的名字>`。然后在后面添加正则表达式的规则。示例代码如下：

```
    from django.urls import re_path
    from . import views

    urlpatterns = [
        # r""：代表的是原生字符串（raw）
        re_path(r'^$',views.article),
        # /article/list/<year>/
        re_path(r"^list/(?P<year>\d{4})/$",views.article_list),
        re_path(r"^list/(?P<month>\d{2})/$",views.article_list_month)
    ]
```
 
4. 如果不是特别要求。直接使用`path`就够了，省的把代码搞的很麻烦（因为正则表达式其实是非常晦涩的，特别是一些比较复杂的正则表达式，今天写的明天可能就不记得了）。除非是url中确实是需要使用正则表达式来解决才使用`re_path`。
