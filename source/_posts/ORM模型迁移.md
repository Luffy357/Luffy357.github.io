---
title: ORM模型迁移
date: 2018-12-10 21:29:36
tags: 
  - Django2.0
category: 学习笔记
---

模型迁移就是将 ORM 中的模型映射到数据库中，这个过程一共分两步，makemigrations 命令生成迁移脚本，migrate 命令执行迁移脚本。
<!-- more -->


## 迁移命令：
 1、makemigrations: 将模型生成迁移脚本，模型所在的 app 必须放在 settings.py 中的 INSTALLED_APPS 中。该命令还有如下几个常用选项：
  + app_label: 后面可以跟一个或者多个app，就只会针对这几个 app 生成迁移脚本，如果没有任何的 app_label，那么会检查 INSTALLED_APPS 中的所有 app 下的模型，针对每个 app 的模型生成迁移脚本。
  + --name: 生成的迁移脚本都会在各自 app 下的 migrations 包中，这个选项可以给对迁移文件指定名字。
```
>>> python manages.py makemigrations --name "add_content_author"
# 生成迁移文件为
0001_add_content_author.py
```
  + empty：生成一个空的迁移脚本，如果你想自己写迁移脚本，可以使用这个命令生成一个空的文件，然后自己再在文件中写迁移脚本。

 2、migrate: 将新生成的迁移脚本映射到数据库中，创建新的表或者修改表的结构，以下是常用的选项：
  + app_label: 将某个 app 下的迁移脚本映射到数据库中，没有指定的话会将 INSTALLED_APPS 中的 app 下的模型都映射到数据库中。
  + app_label migrationname: 将某个 app 下指定名字的 migration 文件映射到数据库中。
  + --fake: 可以将指定的迁移脚本名字添加到数据库中，但是并不会把迁移脚本转换为 SQL 语句，修改数据库中的表。
  + --fake-initial: 将第一次生成的迁移文件版本号记录在数据库中，但并不会真正的执行迁移脚本。

## 执行 migrate 报错
  要知道 migrations 命令是生成的迁移脚本的，并且每个迁移脚本都记录在 app 文件在的 migrations 包中。而 migrate 是执行这些迁移脚本的。当 migrate 命令出错的时候，可能就是是数据库中的迁移版本与 migratins 中的版本一致导致的。
  每执行一次 migrate 操作在数据库 django_migrations 表中也会有记录。
  
  总结：
   + migrate 通过比对数据库 django_migrations 迁移脚本与项目中的迁移脚本，如果发现数据库中没有这个迁移脚本，就会执行这个脚本
   + migrate 将相关的脚本翻译成 SQL 语句，在数据库中执行生效。
     如果 SQL 语句没有问题，那么就会将这个迁移脚本名字记录到 django_migrations

   解决思路:
       1、使用 --fake 参数：
        首先对比数据库中的迁移脚本和代码中的迁移脚本，然后找到不同，之后使用 --fake ，将代码中的迁移脚本添加到 django_migrations 中，但并不会执行 SQL 语句，这样避免执行 migrate 命令重复执行一些迁移脚本，然后报错。
       2、使用 --fake inital 参数：
        如果代码中的迁移脚本和数据库中迁移脚本太多，而理不清时。想相关的 app 迁移文件和数据库中的记录都删除。然后重新映射。但要确保 app 中的模型和数据库的表保持一致。
        + 删除 app 中 migrations 包中的迁移文件，注意不要删除 init.py 文件
        + 删除 数据库中 django_migrations 表中的与 app 相关的记录。
        + 使用 migrations 重新将模型生成一个迁移脚本。
        + 使用 migrate --fake-initial 参数，将刚刚生成的迁移脚本，标记为完成。
        因为这些模型相对应的表，已经在数据库中存在了，不需要重复执行了。这样就不会报错。


## 根据已有的表自动生成模型:
  如果已经有了现成的表，或者要用以前非 Django 开发项目的数据库的表。就需要根据现有的表生成模型。
  1、Django 提供一个 inspectdb 的命令，可以将已经存在的表，自动的生成模型。
  要使用这个命令首先要在 settings.py 中配置好数据库相关信息。也就是要转换的那个数据库。
  ```
  # 通过就会将表转换为模型后的代码。
  python manage.py inspectdb 

  # 要将生成的代码保存到指定的文件夹
  python manage.py inspectdb > filename.py

  # 转换一个表为模型，那么可以指定表的名字
  python manage.py inspectdb table_name > filename.py
  ```

  2、修正模型:
   新生成的 ORM 模型有些地方可能不太合适，那么可以进行重新配置
  + 模型名：模型可以更改合适的
  + 模型所属 app: 只需要将模型放在对应的 app 中。
  + 模型外键引用: 使用 ForeignKey 的地方，模型引用改成字符串的形式。
  + 让 Django 管理模型: 将 Meta 下的 managed=False 删除，这个属性的意思是模型的修改，使用 migrate 都不会映射到数据库中。
  + 当有多对多的时候，应该也要修正模型。将中间表注视了，然后使用ManyToManyField来实现多对多。并且，使用ManyToManyField生成的中间表的名字可能和数据库中那个中间表的名字不一致，这时候肯定就不能正常连接了。那么可以通过db_table来指定中间表的名字。示例代码如下:
```
class Article(models.Model):
   title = models.CharField(max_length=100, blank=True, null=True)
   content = models.TextField(blank=True, null=True)
   author = models.ForeignKey('front.User', models.SET_NULL, blank=True, null=True)
   # 使用ManyToManyField模型到表，生成的中间表的规则是：article_tags
   # 但现在已经存在的表的名字叫做：article_tag
   # 可以使用db_table，指定中间表的名字
   tags = models.ManyToManyField("Tag",db_table='article_tag')

 class Meta:
     db_table = 'article'
```

  + 表名：切记不要修改表的名字。不然映射到数据库中，会发生找不到对应表的错误。

 3、执行命令python manage.py makemigrations生成初始化的迁移脚本。方便后面通过ORM来管理表。这时候还需要执行命令python manage.py migrate --fake-initial，因为如果不使用--fake-initial，那么会将迁移脚本会映射到数据库中。这时候迁移脚本会新创建表，而这个表之前是已经存在了的，所以肯定会报错。此时我们只要将这个0001-initial的状态修改为已经映射，而不真正执行映射，下次再migrate的时候，就会忽略他。

 4、将Django的核心表映射到数据库中：Django中还有一些核心的表也是需要创建的。不然有些功能是用不了的。比如auth相关表。如果这个数据库之前就是使用Django开发的，那么这些表就已经存在了。可以不用管了。如果之前这个数据库不是使用Django开发的，那么应该使用migrate命令将Django中的核心模型映射到数据库中。