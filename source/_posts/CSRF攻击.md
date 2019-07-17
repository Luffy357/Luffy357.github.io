---
title: CSRF攻击
date: 2018-12-28 20:17:13
tags:
  - Django2.0
category: 学习笔记
---

CSRF(Cross Site Request Forgery 跨站域请求伪造)是一种挟制用户在当前已登录的Web应用程序上执行非本意的操作的攻击方法，
<!-- more -->

## CSRF攻击原理
网站是通过 cookie 来实现登录功能的，而 cookie 只要存在浏览器中，那么浏览器在访问这个 cookie 的服务器的时候，就会自动的携带 cookie 信息到服务器上去。这样就产生了一个漏洞，如果访问了一个别有用心或者病毒网站，这个网站可以在页面源代码中插入 js 代码，给其他服务器发送请求，那么因为在发送请求的时候，浏览器会自动的把 cookie 发送给对应服务器，这个相应的服务器，就不知道这个请求是伪造的。从而达到在用户不知情的情况下，给某个服务器发送了一个请求。简单地说，是攻击者通过一些技术手段欺骗用户的浏览器去访问一个自己曾经认证过的网站并运行一些操作（如发邮件，发消息，甚至财产操作如转账和购买商品）。由于浏览器曾经认证过，所以被访问的网站会认为是真正的用户操作而去运行。这利用了web中用户身份验证的一个漏洞：简单的身份验证只能保证请求发自某个用户的浏览器，却不能保证请求本身是用户自愿发出的。

## 防御CSRF
CSRF 攻击的要点就是在向服务器发送请求的时候，相应的 cookie 会自动的发送给对应的服务器。造成服务器不知道这个请求是用户发起的还是伪造的。这时候，我们可以在用户每次访问有表单的页面的时候，在网页源代码中加一个随机的字符串叫做 csrf_token，在cookie 中也加入一个相同值的 csrf_token 字符串。以后给服务器发送请求的时候，必须在 body 中以及 cookie 中都携带 csrf_token，服务器只有检测到 cookie 中的 csrf_token 和 body 中的 csrf_token 都相同，才认为这个请求是正常的，否则就是伪造的。那么黑客就没办法伪造请求了。在 Django 中，如果想要防御 CSRF 攻击，应该做两步工作。第一个是在 settings.MIDDLEWARE 中添加 CsrfMiddleware 中间件。第二个是在模版代码中添加一个 input 标签，加载 csrf_token。示例代码如下
```
# settings.py
'django.middleware.csrf.CsrfViewMiddleware',
```
```
# xx.html
<input type="hidden" name="csrfmiddlewaretoken" value="{{ csrf_token }}"/>

# 或者使用标签csrf_token 可以直接生成上面内容
{% csrf_token %}
```

### 在ajax中处理csrf防御
在 ajax 中处理 csrf 防御，要么手动在 form 中添加 csrfmiddlewaretoken, 或者是在请求头中添加 X-CSRFToken。就可以从返回的 cookie 中提取 csrf-token，再设置进行。
```
# 返回 cookie 信息
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

# 设置csrftoken信息
  '_ajaxSetup': function () {
        $.ajaxSetup({
            beforeSend: function(xhr, settings) {
                if (!/^(GET|HEAD|OPTIONS|TRACE)$/.test(settings.type) && !this.crossDomain) {
                    xhr.setRequestHeader("X-CSRFToken", getCookie('csrftoken'));
                }
            }
        });
    }
```


## iframs 相关知识
1、iframe可以加载嵌入别的域名下的网页。也就是说可以发送跨域请求。比如我可以在我自己的网页中加载百度的网站，示例代码如下：
```
<iframe src="http://www.baidu.com/">
</ifrmae>
```
2、因为iframe加载的是别的域名下的网页。根据同源策略，js只能操作属于本域名下的代码，因此js不能操作通过iframe加载来的DOM元素。
3、如果ifrmae的src属性为空，那么就没有同源策略的限制，这时候我们就可以操作iframe下面的代码了。并且，如果src为空，那么我们可以在iframe中，给任何域名都可以发送请求。
4、直接在iframe中写html代码，浏览器是不会加载的。

攻击者就可以利用 iframs 标签向目标服务器发送请求，因为攻击者这个时候已经拿到了 cookie 信息。在目标服务器没做 crsf 验证的情况下，目标服务器无法区分是用户本人还是另外的人发出的请求。攻击者就可以利用 js 代码执行 iframe 里面的请求。 

## 中间件应用
因为有些页面需要判断用户是否登录再就决定给用户返回什么页面。所以可设置一个中间件，在 request 过来时看里面是否携带登录时写入的 session，如果有就在 request 里面添加一个 user 对象。没有的话就返回一个空的 user 对象。再利用装饰器来装饰类视图。判断用户是否登录没有登录则重定向到登录页面。
```
# middlewares.py
# 自定义中间件
def front_user_middleware(get_request):
    def middleware(request):
        user_id = request.session.get('user_id')
        if user_id:
            try:
                user = UserInfo.objects.get(pk=user_id)
                request.front_user = user
            except:
                request.front_user = None
        else:
            request.front_user = None
        response = get_request(request)
        return response
    return middleware


# views.py
# 在视图中就可以直接获取
user = request.front_user


# decorators.py
# 用户如果没有登录就重定向到登录页面
from django.shortcuts import redirect, reverse

def login_required(func):
    def wrapper(request, *args, **kwargs):
        if request.front_user:
            return func(request, *args, **kwargs)
        else:
            return redirect(reverse('login'))
    return wrapper
```



