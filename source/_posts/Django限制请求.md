---
title: Django限制请求
date: 2018-12-12 22:00:50
tags: 
  - Django2.0
category: 学习笔记
---

浏览器在向服务器发起请求中，有不同的请求方法，相应的服务器会根据不同的请求做不同的处理。例如：GET 请求一般用来向服务器索取数据，不会改变服务器的状态。而 POST 请求一般用来向服务器提交数据，会对服务器的状态进行更改。为了安全服务器应该对不同的请求进行不同的处理。所以可以给视图进行设置只允许特定的求情访问。
<!-- more -->

## 限制请求的装饰器
  Django 内置的视图装饰器可以给视图提供一些限制，用来限制请求的方法。
### require_http_methods: 
 可以传递一个允许访问方法的列表。
```
from django.http.decorators.http import require_http_methods

@require_http_methods(['GET', 'POST'])
def index(request):
   pass
```

### require_GET: 
 只允许 GET 方法访问
```
from django.http.decorators.http import require_GET

@require_GET
def index(request):
   pass
```

### require_POST: 
只允许 POST 方法访问
```
from django.views.decorators.http import require_POST

@require_POST
def index(request):
   pass
```

## 页面重定向
页面重定向就是访问一个 URL 的时候让它跳转到另一个 URL，重定向也分永久重定向和暂时重定向，当然用户访问一个需要登录的页面时，如果该用户没有登录就重定向到登录页面。永久重定向的 http 状态码是301，暂时重定向的 http 状态码是 302。

在 Django 中，重定向使用 redirect 来实现：
```
redirect(to, *args, permanent=False, **kwargs)
# to: 一个url，
# permanent: 是否是一个永久重定向，默认为 False

# views.py
from django.shortcuts import reverse,redirect
def profile(request):
    if request.GET.get("username"):
        return HttpResponse("%s，欢迎来到个人中心页面！")
    else:
        return redirect(reverse("user:login"))
```
