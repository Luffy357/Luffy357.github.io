---
title: SQL注入攻击
date: 2019-01-03 21:46:44
tags:
  - Django2.0
category: 学习笔记
---

SQL 注入就是通过把 SQL 命令插入到表单或者页面请求的查询字符串中，最终达到欺骗服务器执行恶意的 SQL 命令，就是利用现有的应用程序，将恶意的 SQL 命令注入到后台数据库引擎执行的能力， 它可以通过在 Web 表单中输入恶意 SQL 语句得到一个存在安全漏洞的网站上的数据库，而不是按照设计者意图无执行 SQL 语句。
<!-- more -->

## 场景
如现在数据库中有一个 front_user 表
```
class User(models.Model):
    telephone = models.CharField(max_length=11)
    username = models.CharField(max_length=100)
    password = models.CharField(max_length=100)
```

当用原生 sql 语句实现需求时
1、实现一个根据用户 id 获取用户详情的视图
```
def index(request):
    user_id = request.GET.get('user_id')
    cursor = connection.cursor()
    cursor.execute("select id, usename from front_user where id=%s" % user_id)
    rows = cursor.fetchall()
    for row in rows:
        print(row)
    return HttpRponse('success')
```

这样表面上看没有问题，但是如果用户传的 user_id 是 1 or 1=1，那么获取这个字符拼接后就是这样
```
select id,username from front_user where id=1 or 1=1
```

这样就给这个查询语句添加了一个 1=1 永远成立的条件，执行这个语句会将 front_use
表中所有数据都提取出来。

## SQL注入攻击防御
SQL 注入攻击就是利用后端原生 sql 语句拼接 url 的漏洞，传递一些恶意的参数来破坏原有的 sql 语句老达到自己的目的。要防范这个攻击需要注意以下几点：
 + 永远不要信任用户的输入，对用户的输入进行校验，可以通过正则表达式，或者限制长度，对单引号等特殊符号进行转换。
 + 永远不要使用动态拼装 sql，可以使用参数化的 sql 或者直接使用存储过程进行数据查询存取。比如
```
 def index(request):
     user_id = "1 or 1=1"
     cursor = connection.cursor()
     cursor.execute("select id,username from front_user where id=%s",(user_id,))
     rows = cursor.fetchall()
     for row in rows:
         print(row)
     return HttpResponse('success')
```
 + 永远不要使用管理员权限的数据库连接，为每个应用使用单独的权限有限的数据库连接。
 + 不要把机密信息直接存放，加密或者hash掉密码和敏感的信息。
 + 应用的异常信息应该给出尽可能少的提示，最好使用自定义的错误信息对原始错误信息进行包装。

## 在Django中防御SQL注入攻击
1、使用ORM来做数据的增删改查。因为ORM使用的是参数化的形式执行sql语句的。
2、如果万一要执行原生sql语句，那么建议不要拼接sql，而是使用参数化的形式。
