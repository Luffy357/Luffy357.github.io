---
title: QuerySet API
date: 2018-12-08 17:09:47
tags:
  - Django2.0
category: 学习笔记
---

在我们做查找的时候，都是通过 模型名+objects
的方式进行操作，其实'模型名+objects' 所创建的是一个 "Djagno.db.models.manager.Manger" 对象，而 Manager 只是一个空壳，它本身没有任何的属性和方法，它的属性和方法都是通过继承 QuerySet 获得。QuerySet 上还有很多API 可以进行查找操作，这些API 都是在 QuerySet 对象上进行查找之后再创造一个新的 QuerySet 对象，也就是说在新创造的对象上面还是可以使用这些 API 实现链式调用。
<!-- more -->

## QuerySet API

1、filter:
   将满足条件的数据取出来返回一个新的 QuerySet 对象。

2、exclude:
    排除满足条件的数据返回一个新的 QuerySet 对象。

3、annotate:
   给 QuerySet 对象中的每一个字段，都添加一个使用查询表达式(聚合函数，F表达式，Q表达式)的新字段，可以使用关键字参数的形式给表达式字段重命名。

4、order_bay:
   根据查询结果依据某个字段进行排序，如果要倒序排序只需要在字段前加一个负号。排序可以对多个字段进行排序，也就是前一个字段排序有多个数据相同的情况，依据第二个提供的字段进行排序，需要注意的是多个 order_bay 在链式调用的时候只有最后一个会起作用。

5、values:
   默认情况下会把所有字段提取出来，也可以指定要提取的字段。需要注意的是虽然他提取出来的还是一个 QuertSet 对象但不在是模型的形式，而是字典的形式。

6、values_list:
   和values 类似只不过返回的是一个元组对象而不是字典。如果返回的只有一个字段，可以用 flat=True 扁平化，返回的就是一个列表对象。

7、all:
   返回 ORM 模型中所有的 QuerySet 对象。

8、select_related:
   提取某个模型数据的同时也将相关联的字段提取出来，这样可以减少查询次数，优化查询。但只能在一对多或者一对一中使用，不能再多对多或者 多对一中使用。

9、perfetch_related:
   这个方法和 select_related 非常的类似，就是在访问多个表的数据的时候，减少查询的次数，这个方法是为了解决 多对一 和 多对多 的关系的查询。使用这个方法要注意，如果在之后 再进行过滤查询那么优化就没有意义，所以如果有过滤操作，这个时候可以使用 django.db.models.Prefetch 来实现。Prefetch 这个可以提前定义好 queryset 。

10、defer:
   在提取数据时可以过滤掉不需要的字段。返回的是一个模型对象。但主键是不会被过滤的。

11、only:
   这个方法只能获取一条，而且有且只有一条。返回多条或者没有数据都会报错。

12、get:
   获取满足条件的数据，只能返回一条，给出的数据有多条或者没有都会报错。

13、create:
   创建一条数据，并且保存到数据库中。相当于用指定的模型创建一个对象，然后调用这个对象的 save() 方法

14、get_or_create:
   根据条件进行查找，如果找到了就返回这条数据，如果没有找到就创建一个。
```
obj,created = Category.objects.get_or_create(title="默认分类") 
```
如果有标题等于默认分类的分类，那么就会查找出来，如果没有，则会创建并且存储到数据库中。
这个方法的返回值是一个元组，元组的第一个参数obj是这个对象，第二个参数created代表是否创建的。

15、bulk_create：
一次性创建多个数据。示例代码如下：
```
Tag.objects.bulk_create([
    Tag(name='111'),
    Tag(name='222'),
])
```

16、count：
获取提取的数据的个数。如果想要知道总共有多少条数据，那么建议使用 count，而不是使用 len(articles) 这种。因为count在底层是使用 select count(*)来实现的，这种方式比使用 len 函数更加的高效。

17、first和last：
返回 QuerySet 中的第一条和最后一条数据。

18、aggregate：
使用聚合函数。

19、exists：
 判断某个条件的数据是否存在。如果要判断某个条件的元素是否存在，那么建议使用 exists，这比使用 count 或者直接判断 QuerySet 更有效得多。示例代码如下：
```
if Article.objects.filter(title__contains='hello').exists():
    print(True)

比使用count更高效：
if Article.objects.filter(title__contains='hello').count() > 0:
    print(True)
也比直接判断QuerySet更高效：
if Article.objects.filter(title__contains='hello'):
    print(True)
```

20、distinct：
去除掉那些重复的数据。这个方法如果底层数据库用的是 MySQL，那么不能传递任何的参数。比如想要提取所有销售的价格超过80元的图书，并且删掉那些重复的，那么可以使用 distinct 来帮我们实现，示例代码如下：
```
books = Book.objects.filter(bookorder__price__gte=80).distinct()
```
需要注意的是，如果在 distinct之 前使用了 order_by，那么因为 order_by会提取 order_by 中指定的字段，因此再使用 distinct就会根据多个字段来进行唯一化，所以就不会把那些重复的数据删掉。示例代码如下：
```
orders = BookOrder.objects.order_by("create_time").values("book_id").distinct()
```
那么以上代码因为使用了 order_by，即使使用了 distinct，也会把重复的 book_id提取出来。

21、update：
执行更新操作，在SQL底层走的也是 update 命令。比如要将所有 category 为空的 article 的 article 字段都更新为默认的分类。示例代码如下：
```
Article.objects.filter(category__isnull=True).update(category_id=3)
```
注意这个方法走的是更新的逻辑。所以更新完成后保存到数据库中不会执行 save 方法，因此不会更新 auto_now 设置的字段。

22、delete：
删除所有满足条件的数据。删除数据的时候，要注意 on_delete 指定的处理方式。

23、切片操作：
有时候我们查找数据，有可能只需要其中的一部分。那么这时候可以使用切片操作来帮我们完成。QuerySe t使用切片操作就跟列表使用切片操作是一样的。示例代码如下：
```
books = Book.objects.all()[1:3]
for book in books:
    print(book)
```
切片操作并不是把所有数据从数据库中提取出来再做切片操作。而是在数据库层面使用 LIMIE 和 OFFSET 来帮我们完成。所以如果只需要取其中一部分的数据的时候，建议大家使用切片操作。

## 什么时候Django会将QuerySet转换为SQL去执行：
生成一个 QuerySet 对象并不会马上转换为 SQL语句去执行。
比如我们获取 Book表下所有的图书：
```
books = Book.objects.all()
print(connection.queries)
```
我们可以看到在打印 connection.quries 的时候打印的是一个空的列表。说明上面的 QuerySet 并没有真正的执行。
在以下情况下 QuerySet 会被转换为 SQL语句执行：

1、迭代：在遍历 QuerySet 对象的时候，会首先先执行这个 SQL语句，然后再把这个结果返回进行迭代。比如以下代码就会转换为SQL语句：
```
 for book in Book.objects.all():
     print(book)
```
2、使用步长做切片操作：QuerySet 可以类似于列表一样做切片操作。做切片操作本身不会执行 SQL语句，但是如果如果在做切片操作的时候提供了步长，那么就会立马执行 SQL语句。需要注意的是，做切片后不能再执行 filter 方法，否则会报错。
3、调用 len函数：调用len函数用来获取 QuerySet中总共有多少条数据也会执行SQL语句。
4、调用 list函数：调用 list函数用来将一个 QuerySet对象转换为 list对象也会立马执行 SQL语句。
5、判断：如果对某个 QuerySet进行判断，也会立马执行 SQL语句。