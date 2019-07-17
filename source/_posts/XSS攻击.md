---
title: XSS攻击
date: 2019-01-02 23:59:37
tags:
  - Django2.0
category: 学习笔记
---

XSS(Cross Site Script) 攻击又叫做跨脚本攻击，原理是用户在使用具有 XSS 漏洞的网站的时候向这个网站提交一些恶意的代码，当用户在访问这个网站的某个页面的时候，这个恶意代码就会被执行，从而来破坏网页的结构，获取用户的隐私信息等。
<!-- more -->

## XSS攻击场景
比如某个网站有一个发布评价的入口，如果用户在提交数据的时候，提交了一段 js 代码比如"&;lt;script&;gt;alert("hello world");&;lt;/script&;gt;"
那么这个网站在渲染这条消息的时候就会弹出一个对话框来显示。这样运行成功攻击就可以运行很多 js 代码。做其他的一些事情了。

## XSS攻击防御
1、在 Django 的模板中是默认转义的，也就是对提交的字符只做为普通字符处理，而不会做为前端代码渲染。也可以在数据存储到数据库之前就对它进行转义，这样渲染的时候就已经是转义处理后的代码，不会有安全问题。
```
# 使用 escape
from django.template.defaultfilters import escape

# 获取用户提交的数据进行转义再存储到数据库 
content = request.POST.get('content')
escaped_content = escape(content)
Comment.objects.create(content=escaped_content)
```

2、如果用户提交上来的数据包含了一些富文本，那么在渲染的时候也要进行富文本的形式进行渲染，也就一些标签样式要显示出来。这个时候可以在模板中使用标签 safe 就相当于关闭的 django 模板的默认转义功能，这样富文本提交的样式就都可以实现了。
```
# index.html
<ul>
    {% for comment in comments %}
        <li>{{ comment.content|safe }}</li>
    {% endfor %}
</ul>
```
但这样还是避免不了攻击者利用这个入口，提交攻击代码。这个时候可以引入第三方库，对于那种安全的标签我们可以不转义，哪些可能含有攻击代码的标签进行转义获取移除掉，就可以满足需要了。

##bleach库
bleach 库是用来清理包含 html 格式字符串的库，它可以指定哪些标签需要保留，哪些标签是需要过滤掉的。还可以指定标签上哪些属性是可以保留，哪些属性是不需要的。要使用这个库需要先安装
```
pip install beach
```

使用这个库的 clean 方法对数据进行过滤处理。
```
import bleach 
from beach.sanitizer import ALLOWED_TAGS, ALLOWED_ATTRIBUTES

@require_http_methods(['POST'])
def message(request):
    # 获取前端提交的数据
    content = reqeust.POST.get('content')
    # 在默认的允许标签中添加img标签
    tags = ALLOWED_TAGS + ['img']
    # 在默认的允许属性中添加src属性 
    attributes = {**ALLOWED_ATTRIBUTES, 'img':['src']}

    # 对提交的数据进行过滤
    cleaned_content = bleach.clean(content, tags=tags, attributes=attributes)
    
    # 保存到数据库中
    Messages.objects.create(content=cleaned_content)
    return redirect(reverse('index'))
```

+ ALLOWED_TAGS： bleach 中默认定义的一些标签，可以对其进行增删的操作，它是一个列表。
+ ALLOWED_ATTRIBUTES: 这个变量是 bleach 中默认定义的一些属性，也可以进行增删操作，这是一个字典。
自定义修改的这连个变量，作为参数传递到 bleach.clean 方法中。
需要注意的是在对第三方变量进行修改时，不要直接修改原始变量里的内容，而应以该重新定义，不要在其他地方要使用这个变量时会不清楚哪些内容被修改了。

更多 bleach 资料
github地址： "https://github.com/mozilla/bleach"
文档地址： "https://bleach.readthedocs.io/"