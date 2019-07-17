---
title: HttpResponse对象和JsonResponsee类
date: 2018-12-12 23:07:39
tags:
  - Django2.0
category: 学习笔记
---

Django 服务器在接收到客户端请求之后，会将提交上来的数据封装成一个 HttpRequest 对象传给函数视图，在视图进行相关的逻辑处理后，也需要将处理后的结果作为一个响应给浏览器，而这个响应必须是一个 HttpResponseBase 或者他的子类的对象，而HttpResponse 就是 HttpResponseBse 中用得比较多的子类
。
<!-- more -->

### 常用属性：
  1、content：返回内容。
  2、status_code: 返回 Http 响应状态码。
  3、content_type: 返回的数据的 MIME 类型，默认为 text/html，浏览器会根据这个类型来显示数据。常用的 Content-Type 如下：
   + text/html (默认的，html文件)
   + text/plain (纯文本)
   + text/css (css 文件)
   + text/javascript (js 文件)
   + multipart/form-data (文件提交)
   + application/json (json 传输)
   + aplication/xml (xml文件)
  4、设置请求头: response['X-Access-Token'] = 'xxxxx'。

### 常用方法:
   1、set_cookie: 用来设置 cookie 信息。
   2、delete_cookie: 用来删除 cookie 信息。
   3、write： HttpResponse 是一个类似文件的对象，可以写入数据到 content 中。

   ```
   def index1(request):
    response = HttpResponse('<h1>ThreeFire</h1>', content_type='text/json')
    response['X-Access-Token'] = 'threefire'
    # response.status_code = 400
    return response
   ```


## JsonResponse对象
 将对象 dump 为 json 字符串，然后返回将 json 字符串封装成 Response 对象给浏览器
 。它的 Content-Type 是 application/json。
```
from django.http import JsonResponse

def index(request):
   return JsonResponse({'username': "threefire", "age":18})
```

JsonResponse 默认只能对字典进行 dump，如果要对非字典的数据进行 dump，那么就需要给 JsonResponse 传递一个 safe=False 参数
```
return JsonResponse(["threefire",18]，safe=False)
``` 
