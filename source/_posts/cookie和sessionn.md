---
title: cookie和sessionn
date: 2018-12-20 23:23:23
tags:
  - Django2.0
category: 学习笔记
---

因为浏览器与服务器通讯使用的是 HTTP 协议，而 http 请求是无状态的为了能够使服务器分清那个浏览器发过来的请求，引入 cookie 技术。
<!-- more --> 

## cookie和session
1、cookie：在网站中，http请求是无状态的。也就是说即使第一次和服务器连接后并且登录成功后，第二次请求服务器依然不能知道当前请求是哪个用户。cookie的出现就是为了解决这个问题，第一次登录后服务器返回一些数据（cookie）给浏览器，然后浏览器保存在本地，当该用户发送第二次请求的时候，就会自动的把上次请求存储的cookie数据自动的携带给服务器，服务器通过浏览器携带的数据就能判断当前用户是哪个了。cookie存储的数据量有限，不同的浏览器有不同的存储大小，但一般不超过4KB。因此使用cookie只能存储一些小量的数据。

2、session: session和cookie的作用有点类似，都是为了存储用户相关的信息。不同的是，cookie是存储在本地浏览器，session是一个思路、一个概念、一个服务器存储授权信息的解决方案，不同的服务器，不同的框架，不同的语言有不同的实现。虽然实现不一样，但是他们的目的都是服务器为了方便存储数据的。session的出现，是为了解决cookie存储数据不安全的问题的。

3、cookie和session使用：web开发发展至今，cookie和session的使用已经出现了一些非常成熟的方案。在如今的市场或者企业里，一般有两种存储方式：

 + 存储在服务端：通过cookie存储一个sessionid，然后具体的数据则是保存在session中。如果用户已经登录，则服务器会在cookie中保存一个sessionid，下次再次请求的时候，会把该sessionid携带上来，服务器根据sessionid在session库中获取用户的session数据。就能知道该用户到底是谁，以及之前保存的一些状态信息。这种专业术语叫做server side session。Django把session信息默认存储到数据库中，当然也可以存储到其他地方，比如缓存中，文件系统中等。存储在服务器的数据会更加的安全，不容易被窃取。但存储在服务器也有一定的弊端，就是会占用服务器的资源，但现在服务器已经发展至今，一些session信息还是绰绰有余的。
 + 将session数据加密，然后存储在cookie中。这种专业术语叫做client side session。flask框架默认采用的就是这种方式，但是也可以替换成其他形式。

## 在 Django 中操作 cookie
### 设置 cookie
设置 cookie 是要将 cookie 的值给浏览器，所有需要通过 response 的对象来设置，设置 cookie 通过 response.set_cookie 来实现。这个方法还有其他参数
 + key: 这个 cookie 的 key
 + value：这个 cookie 的 value
 + max_age: 最长的生命周期，单位是秒。也就是 cookie 的有效时间
 + expires：过期时间，跟 max_age 类似同时设置会覆盖 max_age。要传递一个 datetime 的日期格式的字符串。
 + path: 对域名下那个路径有效。默认对域名下所有路径都有效。如果设置了那么其他路径不可以使用这个路径的 cookie。
 + domain：针对哪个域名有效。默认是针对主域名下都有效，如果只要针对某个子域名才有效，那么可以设置这个属性.
 + secure：是否是安全的，如果设置为True，那么只能在https协议下才可用。
 + httponly：默认是False。如果为True，那么在客户端不能通过JavaScript进行操作。
```
def index(request):
    response = HttpResponse("index")
    expires = datetime(year=2018, month=12, day=20, hour=23, minute=30, second=0)
    expires = make_aware(expires)
    response.set_cookie('username', 'abc', max_age=180, expires=expires)
    return response
```

### 获取 cookie
获取浏览器发过来的 cookie 信息。通过 request.COOKIES 来实现。这个对象是一个字典类型。
```
def my_list(request):
    cookies = request.COOKIES
    username = cookies.get('username')
    return HttpResponse(username)
```

### 删除 cookie
通过delete_cookie即可删除cookie。实际上删除cookie就是将指定的cookie的值设置为空的字符串，然后使用将他的过期时间设置为0，也就是浏览器关闭后就过期。
```
def delete_cookie(request):
    response = HttpResponse('delete')
    response.delete_cookie('username')
    return response
```

## Django 中操作 session
django中的session默认情况下是存储在服务器的数据库中的，在表中会根据sessionid来提取指定的session数据，然后再把这个sessionid放到cookie中发送给浏览器存储，浏览器下次在向服务器发送请求的时候会自动的把所有cookie信息都发送给服务器，服务器再从cookie中获取sessionid，然后再从数据库中获取session数据。但是我们在操作session的时候，这些细节压根就不用管。我们只需要通过request.session即可操作。
```
def session_view(request):
    request.session['username']='threefire'
    return HttpResponse('session')
```
session 常用方法如下
  + get: 用来从 session 中获取指定值。
```
username = request.session.get('username')
```
  + pop: 从 session 中删除一个值。
```
username = request.session.pop('username')
```
  + keys：从 session 中获取所有键。
  + items：从 session 中获取所有值。
  + clear: 清除当前这个用户的 session 数据。
```
request.session.clear()
```
  + flush：删除 session 并且删除在浏览器中存储的 session_id，一般注销的时候用得比较多。
  + set_expiry(value): 设置过期时间。
     + 整形: 表示秒数，表示多少秒后过期。
     + 0：代表只要关闭浏览器， session 就会过期。
     + None：会使用全局的session配置。在settings.py中可以设置SESSION_COOKIE_AGE来配置全局的过期时间。默认是1209600秒，也就是2周的时间。
  + clear_expired: 清除过期的session。Django并不会清除过期的session，需要定期手动的清理，或者是在终端，使用命令行python manage.py clearsessions来清除过期的session。
  
### 修改session的存储机制
默认情况下，session数据是存储到数据库中的。当然也可以将session数据存储到其他地方。可以通过设置SESSION_ENGINE来更改session的存储位置，这个可以配置为以下几种方案：
1、django.contrib.sessions.backends.db：使用数据库。默认就是这种方案。
2、django.contrib.sessions.backends.file：使用文件来存储session。
3、django.contrib.sessions.backends.cache：使用缓存来存储session。想要将数据存储到缓存中，前提是你必须要在settings.py中配置好CACHES，并且是需要使用Memcached，而不能使用纯内存作为缓存。
4、django.contrib.sessions.backends.cached_db：在存储数据的时候，会将数据先存到缓存中，再存到数据库中。这样就可以保证万一缓存系统出现问题，session数据也不会丢失。在获取数据的时候，会先从缓存中获取，如果缓存中没有，那么就会从数据库中获取。
```
setting.py

CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': ['127.0.0.1:11211']
    }
}

SESSION_ENGINE = django.contrib.sessions.backends.cached_db

```
5、django.contrib.sessions.backends.signed_cookies：将session信息加密后存储到浏览器的cookie中。这种方式要注意安全，建议设置SESSION_COOKIE_HTTPONLY=True，那么在浏览器中不能通过js来操作session数据，并且还需要对settings.py中的SECRET_KEY进行保密，因为一旦别人知道这个SECRET_KEY，那么就可以进行解密。另外还有就是在cookie中，存储的数据不能超过4k。