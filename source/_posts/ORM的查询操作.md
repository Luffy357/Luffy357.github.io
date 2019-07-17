---
title: ORM的查询操作
date: 2018-12-08 13:10:55
tags:
  - Django2.0
category: 学习笔记
---

查找数据库是一个很频繁的操作，查询一般使用 filter、exclude、get三个方法来实现。调用这些方法的时候还可以传递不同的参数来实现需求。在ORM 层面这些查询都是使用 field+'__'+ condition 的方式来使用的。
<!-- more -->
注意：要查询的字段与查询参数要用两个下划线连接。下面是一些常用的查询条件。

## 查询参数
### excat:
   精确的相等查找，如果提供一个None，那么就是查找数据库里面的 null
```
article = Article.objects.get(id__excat=5)
article = Article.objects.get(id__excat=None)
```
被翻译为 SQL 语句
```
   select ... from article where id=5;
   select ... from article where id=None;
```

### iexcat:
   使用 LIKE 进行查找
```
   article = Article.objects.filter(title__iexcat='hello')
```
   在 SQL 表现为
```
   select ... from article where title like 'hello';
```
   excat 与 iexcat 实际区别就是 '=' 和 LIKE 的区别。


### contains:
判断某个字段是否包含了某个数据：大写敏感
```
article = Article.objects.filter(title__contains='hello')
```
翻译成 SQL 语句就是
```
select ... where title like '%hello%';
```

### in:
提取给定的值在给定的容器中，容器可以是 list、tuple、或者任何可以迭代的对象，包括QuerySet 对象。
```
articles = Article.objects.filter(id__in=[1,2,3]) 
```
翻译成 SQL 语句为
```
select ... from article where id in (1,2,3)
```
如果传递 QuerySet 对象
需求查找标题中有hello的文章，有哪些分类。
在这里 Article 和 Cattegory 一对多外键关系。
```
queryset = Article.objects.filter(title__contains='hello')
categories = Category.objects.filter(article__in=queryset)
```
如果要判断相关联的表的字段，那么也是通过来连接。并且在做关联查询的时候，不需要写 models_set ，直接使用模型的名字的小写化就可以了。比如通过分类去查找相应的文章，那么通过 "article__id__in" 就可以了，而不是写成 "article_set__id__in" 的形式。而且在查找主键时还可以省略 "__id" , 直接写 article，如上
当然如果你不想使用默认的形式，可以在外键定义的时候传递 "related_query_name" 来指定反向查询的名字。
反向引用是将模型名字小写化，然后再加上 "_set"，比如 "article_set"，可以通过 "related_name"来指定自己的方式，而不是用默认的方式。

### gt、gte、lt、lte:
gt、gte、lt、lte分别表示大于、大于等于、小于、小于等于的意识
```
article = Article.objects.filter(id__gt=4)
```

### startwith、istartwith、endswith、iendswith:

判断某个字段以什么开始、大小写敏感、以什么字段结束、大小写敏感
查找文章标题中以'world'的开始的
```
article = Article.objects.filter(title__startwith='world')
```

###  range:
给定的字段值是否在某个区间内

在使用时间段查询时要注意将时间转换为 aware时间
```
from django.utils.timezone import make_aware
import datetime

start_time = make_aware(datetime(year=2018,month=11, day=29, hour=20, minute=0, second=0))
end_time = make_aware(datetime(year=2018,month=11, day=30, hour=20, minute=0, second=0))
articles = Article.objects.filter(create_time__range=(start_time, end_time)
```

### date、year、month、day:
指定某天、某年、某月、某日进行过滤
注意使用时区时要将时区文件导入到 mysql 中，因为默认情况下 MySQL 的表中是没有存储时区相关的信息的。因此我们需要下载一些时区表的文件，然后添加到 Mysql 的配置路径中。windows操作系统。那么在 "http://dev.mysql.com/downloads/timezones.html" 下载t imezone_2018d_posix.zip - POSIX standard。然后将下载下来的所有文件拷贝到 C:\ProgramData\MySQL\MySQL Server 5.7\Data\mysql 中覆盖即可。linux或者mac系统在命令行中执行以下命令："mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -D mysql -u root -p" ，然后输入密码，从系统中加载时区文件更新到mysql中。
```
article = Article.objects.filter(create_time__date=date(2018,11,29))
```

### week_day:
同year，根据星期几进行查找。1表示星期天，7表示星期六，2-6代表的是星期一到星期五。
```
article = Article.objects.filter(create_time__week_day=1)
```

### time:
根据时间进行查找
```
article = Artile.objects.filter(create_time__time=date.time(12,12,12)
```
以上的代码是获取每一天中12点12分12秒发表的所有文章。


### isnull:
根据值是否为空进行查找
```
article = Article.objects.filter(context__isnull=False)
```

### regex、iregex
大小写敏感和大小写不敏感的正则表达式
```
article = Article.objects.filter(title__regex=r'^hello')
```

更多的关于时间的过滤，请参考Django官方文档："https://docs.djangoproject.com/en/2.0/ref/models/querysets/#range"。


## 聚合函数:
在 SQL 中可以使用聚合函数来提取数据，在 ORM 查询中使用 aggregate 和 annotate 来实现。需要注意的是聚合函数返回的是字典而不是 QuerySet 对象，而且聚合函数可以作用在任何QuerySet 对象上。

```
#使用聚合函数要导入
from django.db.models import Avg
```

1、Avg:
    求平均值，比如想要所有图书的平均价格。
```
    result = Book.objects.aggregate(Avg('price'))
    print(result)

    # 返回结果
    {'price_avg': 23}
```
其中 price_avg 的结构是 field_avg 规则构成的，如果想要修改默认的名字，那么就可以将 Avg 赋值给一个关键字参数，
```
result = Book.objects.aggregate('my_avg'=Avg('price'))

# 返回
{'my_price': 23}
```

2、Count：
    获取指定的对象的个数
```
result = Book.objects.aggregate(book_nums=Count('id'))
```
Count 类中还有一个参数叫做 distinct， 默认是等于 False， 如果为True，那么可以去掉重复值。

3、Max和 Min： 
      获取指定对象的最大值和最小值
```
result = Author.objects.aggregate(Max('age'), Min('age'))
```

4、Sum
      求指定对象的总和
```
result = Book.objects.annotate(total=Sum('bookstore__price')).values("name", "tatle")
```
annotate 可以实现分组，total的值为一种图书的销售总和，values 方法意思只提取 name 和 total 两个字段的值。

aggregate：
    返回使用聚合函数后的字段和值。

annotate：
    在原来模型字段的基础之上添加一个使用了聚合函数的字段，并且在使用聚合函数的时候，会使用当前这个模型的主键进行分组（group by）。
比如以上Sum的例子，如果使用的是annotate，那么将在每条图书的数据上都添加一个字段叫做total，计算这本书的销售总额。
而如果使用的是aggregate，那么将求所有图书的销售总额

## F表达式：
F表达式是用来优化ORM操作数据库的。比如我们要将公司所有员工的薪水都增加1000元，如果按照正常的流程，应该是先从数据库中提取所有的员工工资到 Python 内存中，然后使用 Python 代码在员工工资的基础之上增加1000元，最后再保存到数据库中。这里面涉及的流程就是，首先从数据库中提取数据到 Python 内存中，然后在 Python 内存中做完运算，之后再保存到数据库中。示例代码如下：
```
employees = Employee.objects.all()
for employee in employees:
    employee.salary += 1000
    employee.save()
```
而我们的F表达式就可以优化这个流程，他可以不需要先把数据从数据库中提取出来，计算完成后再保存回去，他可以直接执行SQL语句，就将员工的工资增加1000元。示例代码如下：
```
from djang.db.models import F
Employee.object.update(salary=F("salary")+1000)
```
F表达式并不会马上从数据库中获取数据，而是在生成SQL语句的时候，动态的获取传给F表达式的值。

比如如果想要获取作者中，name和email相同的作者数据。如果不使用F表达式，那么需要使用以下代码来完成：
```
authors = Author.objects.all()
for author in authors:
    if author.name == author.email:
        print(author)
```
如果使用F表达式，那么一行代码就可以搞定。示例代码如下：
```
from django.db.models import F
authors = Author.objects.filter(name=F("email"))
```

## Q表达式：
如果想要实现所有价格高于100元，并且评分达到9.0以上评分的图书。那么可以通过以下代码来实现：
```
books = Book.objects.filter(price__gte=100,rating__gte=9)
```
以上这个案例是一个并集查询，可以简单的通过传递多个条件进去来实现。
但是如果想要实现一些复杂的查询语句，比如要查询所有价格低于10元，或者是评分低于9分的图书。那就没有办法通过传递多个条件进去实现了。这时候就需要使用Q表达式来实现了。示例代码如下：
```
from django.db.models import Q
books = Book.objects.filter(Q(price__lte=10) | Q(rating__lte=9))
```
以上是进行或运算，当然还可以进行其他的运算，比如有&和~（非）等。一些用Q表达式的例子如下：
```
from django.db.models import Q
# 获取id等于3的图书
books = Book.objects.filter(Q(id=3))
# 获取id等于3，或者名字中包含文字"记"的图书
books = Book.objects.filter(Q(id=3)|Q(name__contains("记")))
# 获取价格大于100，并且书名中包含"记"的图书
books = Book.objects.filter(Q(price__gte=100)&Q(name__contains("记")))
# 获取书名包含“记”，但是id不等于3的图书
books = Book.objects.filter(Q(name__contains='记') & ~Q(id=3))
```
