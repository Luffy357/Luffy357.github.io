---
title: Memcached
date: 2018-12-19 22:20:55
tags:
  - Django2.0
category: 学习笔记
---

memcached 是一个高性能的分布式的内存对象缓存系统。将一些经常需要用到又不太重要的数据缓存到内存中，来分担数据库的压力。memcache 是通过在内存里维护一个统一的巨大哈希表。memcached 能存储各种各样的数据，包括图像、视频、文件、数据的检索结果。
<!-- more -->

## 安装 memcached
1.windows:
   + 安装: 官网下载 memcached.exe 文件，在命令行运行 memcached.exe -d install
   + 启动: memcached.exe -d start
2.linux(ubuntu):
   + 安装: sudo apt install memcached
   + 启动: 
      cd /usr/local/memcached/bin
      ./memcached -d start

  其他命令：
   + -d: 在后台运行
   + -m: 指定能使用的内存
   + -p: 指定端口号
   + -l: 别的机器通过那个 IP 连接到服务器的 memcached。
   要使用以上参数必须 使用 /usr/bin/memcached 的方式来运行。比如通过 service memcached start 启动的 memcached 那么只能本机连接，其他机器无法访问。

## telnet 操作 memcached
telnet 连接 memcache: telnet 127.0.0.1[IP地址] 11211[端口号]
memcached 保存数据是以键值对的形式存在的。
### 常用命令：
+ set: 添加数据，有这个 key 就覆盖，没有就添加
```
>set username[key] 0\1[是否压缩] 120[过期时间] 10[字符长度]
>threefire[value]
>STORED
```
+ add: 向 memcached 中添加数据，有相同的 key 就报错。
```
>add username[key] 0\1[是否压缩] 120[保存时间] 10[字符长度]
>threefire[value]
>STORED
```
+ get：通过 key 获取数据
```
>get username
>threefire
```
+ delete: 删除数据
```
>delete username
>NOT_FOUND
```
+ flush_all: 删除 memcached 中的所有数据
+ incr: 对数字类型的值进行相加操作，都必须是数字类型，否则会报错
```
>set age 0 120 1
>5
>STORED
>incr age 5
10
```
+ decr: 给数字类型的值进行相减操作。都必须是数字类型，否则会报错
+ stats: 查看 memcached 当前的状态。
```
STAT get_hits： get命令命中了多少次
STAT get_misses： get命令get空了多少次
STAT curr_items： 当前 memcached 中的键值对的个数
STAT total_connections: 从 memcached 开启到现在总共的连接数
STAT curr_connections: 当前的连接数
```

## python 操作 memcached
1、安装 python-memcached: pip install python-memcached
2、建立连接:
```
import memecache
mc = memcache.Client(['127.0.0.1:11211', '192.168.0.1:11211'], debug=True)
# memcached 可以是分布式的可以部署到多台机器上，所有可以填入多个 IP
```
3、设置数据:
```
mc.set('username', 'threefire', time=60)
# 一次性添加多条数据
mc.set_multi({'email': 'xxx@qq.com', 'telphone': '11112222'}, time=60)
```
4、获取数据：
```
mc.get('telphone')
```
5、删除数据：
```
mc.delete('email')
```
6、自增长：delta 默认为 1
```
mc.incr('age'，delta=10)
```
7、自减少：
```
mc.decr('age', delta=5)
```

## memcached的安全性
因为 memcached 的操作不需要任何用户名和密码。知道 ip 地址和端口号就可以链接。所有存在安全问题。为了不让别人轻易访问到，可以限制访问 IP 和 更改端口的形式。
  + 使用 -l 参数设置为只有本机可以链接。
  +  使用防火墙，关闭 11211 端口
```
  ufw enable # 开启防火墙
  ufw disable # 关闭防火墙
  ufw default deny # 防火墙以禁止的方式打开，默认是关闭那些没有开启的端口
  ufw deny 端口号 # 关闭某个端口
  ufw allow 端口号 # 开启某个端口
```

## 在 Django 中使用 memcached
首先在 settings.py 总配置好缓存
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': [
        '127.0.0.1:11211',
        '192.168.0.1:11211'
    }
}
```

配置好 memcached 的缓存后，再 Django 项目中就可以操作 memcached 了
```
from django.core.cache import cache

def index(request):
    cache.set('abc','zhiliao',60)
    print(cache.get('abc'))
    response = HttpResponse('index')
    return response
```

需要注意的是，django在存储数据到memcached中的时候，不会将指定的key存储进去，而是会对key进行一些处理。比如会加一个前缀，会加一个版本号。如果想要自己加前缀，那么可以在settings.CACHES中添加KEY_FUNCTION参数
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': '127.0.0.1:11211',
        'KEY_FUNCTION': lambda key,prefix_key,version:"django:%s"%key
    }
}
```