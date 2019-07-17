---
title: Django连接数据库
date: 2018-12-06 21:27:55
tags:
  - Django2.0
category: 学习笔记
---

django 操作数据库其实底层还是 Python 连接的，python 连接数据库就要通过驱动程序，驱动程序根据不同的 python 版本有很多，在 "https://www.lfd.uci.edu/~gohlke/pythonlibs/#" 这里有很多 python 扩展包。选择对应的版本进行安装。这里是用 mysqlclient 来连接数据库。
<!-- more -->

## djano配置数据库连接
django 连接数据库首先要在 settings.py 文件中的 DATABASES 变量进行配置。
```
DATABASES = {
    default: {
        # 选择数据引擎
        ENGINE: 'django.db.backends.mysql',
        # 数据库名字
        NAME：'threefile',
        # 连接数据库的用户名
        USER: 'root',
        # 连接数据库的密码
        PASSWORD: 'root',
        # 数据库主机地址
        HOST: '127.0.0.1',
        # 数据库使用的端口
        PORT: '3306'
    }
}
```

## Django 操作数据库
在 settings.py 中配置好了数据库连接信息后直接使用 Django 封装好的接口就可以操作了。
在Django中操作数据库有两种方式。
  第一种方式就是使用原生 sql 语句操作，
  第二种就是使用 ORM 模型来操作。
在 Django 中使用原生sql语句操作其实就是使用 python db API 的接口来操作。只不过 Django 将数据库连接的这一部分封装好了, 使用的什么数据库驱动就用相应的接口就可以了。

### 原生的 SQL 语句
使用 django 封装好的 connection 对象可以读取 settings 文件的数据库配置信息
```
from djagno.db import connection

# 首先获取游标对象
cursor = connection.cursor()
# 执行原生SQL语句
cursor.execute("select * from book")
# 获取所有数据
rows = cursor.fetchall()

# 遍历取出每条数据
for row in rows：
    print(row)

```

以上的 execute 以及 fetchall 方法都是 Python DB API 规范中定义好的。任何使用 Python 来操作 MySQL 的驱动程序都应该遵循这个规范。

#### Python DB API下规范下cursor对象常用接口：
description：如果cursor执行了查询的sql代码。那么读取 cursor.description 属性的时候，将返回一个列表，这个列表中装的是元组，元组中装的分别是 (name,type_code,display_size,internal_size,precision,scale,null_ok)，其中name代表的是查找出来的数据的字段名称，其他参数暂时用处不大。
rowcount：代表的是在执行了sql语句后受影响的行数。
close：关闭游标。关闭游标以后就再也不能使用了，否则会抛出异常。
execute(sql[,parameters])：执行某个sql语句。如果在执行sql语句的时候还需要传递参数，那么可以传给 parameters 参数。
示例代码如下：
```
 cursor.execute("select * from article where id=%s",(1,))
```
fetchone：在执行了查询操作以后，获取第一条数据。
fetchall：获取所有满足sql语句的数据。
fetchmany(size)：在执行查询操作以后，获取多条数据。具体是多少条要看传的size参数。如果不传size参数，那么默认是获取第一条数据。

## ORM 模型
ORM 全称 Object Relational Mapping，中文叫做对象关系映射，通过 ORM 我们可以通过类的方式去操作数据库，而不用再写原生的 SQL 语句。通过把表映射成类，把行作实例，把字段作为属性，ORM 在执行对象操作的时候最终还是会把对应的操作转换为数据库原生语句。

### 创建ORM模型：
ORM 模型一般都是放在 app 的 models.py 文件中。每个 app 都可以拥有自己的模型。并且如果这个模型想要映射到数据库中，那么这个 app 必须要放在 settings.py的 INSTALLED_APP 中进行安装。以下是写一个简单的书籍 ORM 模型
```
from django.db import models

class Book(models.Model):
    name = models.CharField(max_length=20,null=False)
    author = models.CharField(max_length=20,null=False)
    price = models.FloatField(default=0)
```

在数据库中，app 名加类名就是表的名字，Book 类的属性名就是表中字段的名字。所有在 ORM  模型中类和属性就是对应着数据库中的一张表。

### 映射模型到数据库中：
将 ORM 模型映射到数据库中
1、在 settings.py 中，配置好 DATABASES，做好数据库相关的配置。
2、在 app 中的 models.py 中定义好模型，这个模型必须继承自 django.db.models。
3、将这个 app 添加到 settings.py 的 INSTALLED_APP 中。
4、执行命令 makemigrations 来生成迁移脚本文件。
5、执行命令 migrate 来将迁移脚本文件映射到数据库中。

### 数据模型的基本操作：
1、取出所有数据
   all 方法取出所有数据
```
# views.py
# 导入模型
from .models import Book

book = Book.objects.all()
```

2、取出单个对象,
    get 方法获得单个符合条件的对象。
```
# pk就是主键的缩写，不管在数据库表中主键叫什么，pk都对应着主键
book = Book.objects.get(pk=1)
```

3、过滤数据
    filter 方法用来进行条件过滤，只要符合条件的都取出来
```
book = Book.objects.filter(price=100)
```

4、增加数据
   建立一个对象，然后保存
```
book = Book(name='西游记', author='吴承恩', price=100)
book.save()
```

5、删除数据
   获取数据对象，执行delete 方法
```
book = Book.objects.get(pk=1)
book.delete()
```

6、修改数据
   对数据对象重新赋值，保存
```                  
book = Book.objects.get(name='西游记')
book.price = 200
book.save()
```


## 模型常用属性
在 Django 中，定义了一些 Field 属性与数据库表的字段对应，不同的 Field 对应不同的字段类型。

### AutoField：
映射到数据库中是 int 类型，可以有自动增长的特性。一般不需要使用这个类型，如果不指定主键，那么模型会自动的生成一个叫做 id 的自动增长的主键。如果你想指定一个其他名字的并且具有自动增长的主键，使用 AutoField 也是可以的。

### BigAutoField：
64位的整形，类似于 AutoField，只不过是产生的数据的范围是从1-9223372036854775807。

### BooleanField：
在模型层面接收的是True/False。在数据库层面是 tinyint类型。如果没有指定默认值，默认值是 None。

### CharField：
在数据库层面是 varchar 类型。在 Python 层面就是普通的字符串。这个类型在使用的时候必须要指定最大的长度，也即必须要传递 max_length 这个关键字参数进去。

### DateField：
日期类型。在 Python 中是 datetime.date 类型，可以记录年月日。在映射到数据库中也是 date 类型。使用这个 Field 可以传递以下几个参数：

auto_now：在每次这个数据保存的时候，都使用当前的时间。比如作为一个记录修改日期的字段，可以将这个属性设置为 True。
auto_now_add：在每次数据第一次被添加进去的时候，都使用当前的时间。比如作为一个记录第一次入库的字段，可以将这个属性设置为 True。

### DateTimeField：
日期时间类型，类似于 DateField。不仅仅可以存储日期，还可以存储时间。映射到数据库中是datetime类型。这个 Field 也可以使用 auto_now 和 auto_now_add 两个属性。

### TimeField：
时间类型。在数据库中是time类型。在 Python 中是 datetime.time 类型。

### EmailField：
类似于 CharField。在数据库底层也是一个 varchar类型。最大长度是254个字符。

### FileField：
用来存储文件的。这个请参考后面的文件上传章节部分。

### ImageField：
用来存储图片文件的。这个请参考后面的图片上传章节部分。

### FloatField：
浮点类型。映射到数据库中是 float 类型。

### IntegerField：
整形。值的区间是-2147483648——2147483647。

### BigIntegerField：
大整形。值的区间是-9223372036854775808——9223372036854775807。

### PositiveIntegerField：
正整形。值的区间是0——2147483647。

### SmallIntegerField：
小整形。值的区间是-32768——32767。

### PositiveSmallIntegerField：
正小整形。值的区间是0——32767。

### TextField：
大量的文本类型。映射到数据库中是 longtext 类型。

### UUIDField：
只能存储uuid格式的字符串。uuid 是一个32位的全球唯一的字符串，一般用来作为主键。

### URLField：
类似于 CharField，只不过只能用来存储url格式的字符串。并且默认的 max_length 是200。


## Field的常用参数：
### null：
如果设置为 True，Django 将会在映射表的时候指定是否为空。默认是为 False。在使用字符串相关的 Field（CharField/TextField）的时候，官方推荐尽量不要使用这个参数，也就是保持默认值 False。因为 Django在处理字符串相关的 Field的时候，即使这个 Field的 null=False，如果你没有给这个Field传递任何值，那么 Django也会使用一个空的字符串""来作为默认值存储进去。因此如果再使用 null=True，Django会产生两种空值的情形（NULL或者空字符串）。如果想要在表单验证的时候允许这个字符串为空，那么建议使用 blank=True。如果你的 Field是 BooleanField，那么对应的可空的字段则为 NullBooleanField。

### blank：
标识这个字段在表单验证的时候是否可以为空。默认是 False。
这个和 null 是有区别的，nul l是一个纯数据库级别的。而 blank 是表单验证级别的。

### db_column：
这个字段在数据库中的名字。如果没有设置这个参数，那么将会使用模型中属性的名字。

### default：
默认值。可以为一个值，或者是一个函数，但是不支持 lambda 表达式。并且不支持列表/字典/集合等可变的数据结构。

### primary_key：
是否为主键。默认是False。

### unique：
在表中这个字段的值是否唯一。一般是设置手机号码/邮箱等。

官方文档中更多的Field参数：https://docs.djangoproject.com/zh-hans/2.0/ref/models/fields/
