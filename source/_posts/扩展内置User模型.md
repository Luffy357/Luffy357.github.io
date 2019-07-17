---
title: 扩展内置User模型
date: 2019-01-07 20:41:32
tags:
  - Django2.0
category: 学习笔记
---

Django 有一个内置的授权系统，用来处理用户、分组、权限以及基于 cookie 的会话系统。 Django 的授权系统包括验证和授权两个部分。验证时验证这个用户是否是他声称的人(比如用户名和密码验证，角色验证), 授权是给与他相应的权限。
Django 内置的权限系统包括用户、权限、分组、一个可以配置的密码哈希系统和可拔插的后台管理系统。
<!-- more -->

##使用授权系统
在创建 Django 项目的时候，就已经集成了授权系统。在 settings.py 中有关于授权系统的配置如下。
 + INSTALLED_APPS:
 1、django.contrib.auth: 包含了一个核心授权框架，以及大部分的模型定义。
 2、django.contrib.contenttypes: Content Type 系统，可以用来关联模型和权限。
 + 中间件:
 1、SessionMiddleware: 用来管理 session。
 2、AuthenticationMiddleware: 用来处理和当前 session 相关联的用户。

##User模型
User 模型是这个框架的核心部分。它的路径是 django.contrib.auth.models.User。 它有以下字段：
+ username: 用户名。150个字符以内，可以包含数字和英文字符，以及_、@、+、.和-字符，不能为空，且必须唯一。
+ first_name：姓氏，在30个字符以内，可以为空。
+ last_name: 名字，在150个字符以内，可以为空。
+ email: 邮箱，可以为空。
+ password: 密码，经过哈希过后的密码。
+ groups: 分组, 一个用户可以属于多个分组，一个分组可以拥有多个用户。 groups 这个字段是跟 Group 的一个多对多的关系。
+ user_permissions: 权限。一个用户可以拥有多个权限，一个权限可以被多个用户所有和 Permission 属于一种多对多的关系。
+ is_staff: 是否可以进入到 admin 的后台站点。意为是否是员工。
+ is_ active：是否是可用的，对于一些想要删除账号的数据,我们设置这个值为 False 就可以了，而不是真正的从数据库中删除。
+ is_superuser: 是否是超级管理员，那么拥有整个网站的所有权限。
+ last_login: 上次登录的时间。
+ date_joined: 账号创建的时间。

## User模型的基本用法。
### 创建用户
通过 create_user 方法可以快速的创建用户，这个方法必须要传递 username、email、password。
```
# views.py
from django.contrib.auth.models import User

def create_user(request):
    # 这个方法创建的用户保存在 auth_user 表中
    new_user = User.objects.create_user('threefire', 'xxx@qq.com', '11111')
    new_user.save()
    return HttpRponse('succed')
```

### 创建超级用户
创建超级用户的方式有两种，一种是在命令行使用命令。后面会提示输入必须字段。
```
python manage.py createsuperuser
```
一种是通过 create_superuser 方法，和创建普通用户一样。

## 修改密码
因为密码是经过加密处理后才能存储进去的，所以如果要修改密码，不能直接修改 pssword 字段，而需要通过调用 set_password 来达到修改密码的目的。
```
user = User.objects.get(username='threefire')
user.set_password('新密码')
user.save()
```

## 登录验证
Django 的验证系统可以直接实现登录的验证功能，通过 django.contrib.auth.authenticate 就可以实现。这个方法只能通过 username 和 password 来进行验证。
```
from django.contrib.auth import authenticate

user = authenticate(username='threefire', password='111111')
# 如果验证通过会返回一个user对象
if user is not None：
    # 验证通过要执行的代码
else：
    # 验证没有通过执行的代码
```

## 扩展用户模型
如果 Django 内置的 user 模型不能满足需求，比如在验证用户登录的时候，需要用手机号或者邮箱验证。或者需要增加新的字段。那么就需要扩展用户模型。扩展用户模型有多种方式。
### 1、设置Proxy模型
如果对 Django 提供的字段，已经验证的方法都满意，无需其他改动，只需要在原有的基础上增加一些操作的方法。那么可以使用代理的模式。
```
from django.contrib.auth.models import User

class Person(User):
    class Meta:
        proxy = True

    @classmethod
    def get_blacklist(cls):
        return cls.objects.filter(is_active=False)
```
定义一个代理类，继承 User，并且在 Meta 中设置 proxy=True。 说明这个只是 User的一个代理模型，并不会影响原来 User 模型在数据库中表的结构，也不需要做映射。如果想要获取所有黑名单的人，就可以通过 Person.get_blacklist() 来取到。代理类就是对父类的方法扩展，而不用去修改源码。User.objects.all() 和 person.objects.all() 是等价的。因为都是从 User 这个模型中获取所有的数据。

### 2、一对一设置外键
如果想在内置的 User 模型上添加其他字段，那么可以使用一对一外键的方式。
```
from django.contrib.auth.models import User
from django.dispatch import receiver
from django.db.models.signals import post_save

class UserExtension(models.Model):
    extension = models.OneToOneField(User, on_delete=models.CASCADE, related_name='extension')
    telephone = models.CharField(max_length=100)
    school = models.CharField(max_length=100)


@receiver(post_save, sender=User)
def create_user_extension(sender, instance, created, **kwargs):
    if created:
        UserExtension.objects.create(user=instance)
    else:
        instance.extension.save()

# views.py
# 通过手机号码验证
def my_authenticate(telephone, password):
    user = User.objects.filter(extension__telephone=telephone).first()
    if user:
        is_correct = User.check_password(password)
        if is_correct:
            return user
        else:
            return None
    else:
        return None

def one_view(request):
    # user = User.objects.create_user(username='threefire', email='xxx@qq.com', password='111111')
    # 通过一对一外键创建字段
    # user.extension.telephone = '18800003333'
    # 自定义验证登录
    user = my_authenticate(telephone='18800003333', password='111111')
    if user：
       # 验证成功代码
    else：
       # 验证失败代码

```
定义一个 UserExtension 模型，与 User 模型是一对一关系。要增加新的字段就添加到 UserExtension 上。 因为扩展的模型和 User 模型是一对一模型，那么在创建了一个 User对象时，也必须创建一个 UserExtension 对象。要知道 User 对象什么时候创建了对象。就见监控 User 的 save 方法。只要 User 调用了 save 方法。就创建一个 UserExtension 和 User 进行绑定。create_user_extension 就是实现这个功能。

### 3、继承 AbstractUser
这个类是 User 的父类，所以如果不想修改原来的 User ， 就可以继承 AbstractUser 对 User 类重写。可以重定义登录验证的字段，增加一些字段。
```
class User(AbstractUser):
    telephone = models.CharField(max_length=11, unique=True)
    school = models.CharField(max_length=100)
    
    # 登录时使用 authenticate 验证的字段
    USERNAME_FIELD = 'telephone'
    
    # 指定 objects 调用的方法
    objects = UserManager()
```

重写 objects 对象。因为 create_user 这些创建用户的方法还是原始 User 对象上面的。 对象上的。如果不重写这个方法，那么还是要传递原始 User 之前定义的字段，就达不到想要的效果。
```
class UserManager(BaseUserManager):
    def _create_user(self, telephone, username, password, **kwargs):
        if not telephone:
            raise ValueError('必须要传递手机号')
        if not password:
            raise ValueError('必须要传递密码')
        user = self.model(telephone=telephone, username=username, **kwargs)
        user.set_password(password)
        user.save()
        return user
    
    def create_user(self, telephone, username, password, **kwargs):
        kwargs['is_superuser'] = False
        return self._create_user(telephone=telephone, password=password, username=username, **kwargs)
    
    def create_superuser(self, telephone, username, password, **kwargs):
        kwargs['is_superuser'] = True
        return self._create_user(telephone=telephone, password=password, username=username, **kwargs)
```

还需要在 settings 中配置好 AUTH_USER_MODEL=youapp.user ,因为这种方式破坏了原来 User 模型的表结构，所以必须要在第一次 migrate 前定义好，或者删除所以表进行重新映射。

### 4、继承AbstractBaseUser 模型
如果想修改默认的验证方式，并且不想要原来 User 模型上的一些字段，那么就可以自定义一个模型，然后继承自 AbstractBaseUser , 再添加你想要的字段。在对 Django 比较了解情况下才用这样方式，因为改动较多。
如果你想修改默认的验证方式，并且对于原来User模型上的一些字段不想要，那么可以自定义一个模型，然后继承自AbstractBaseUser，再添加你想要的字段。这种方式会比较麻烦，最好是确定自己对Django比较了解才推荐使用。步骤如下：

```
class User(AbstractBaseUser,PermissionsMixin):
    email = models.EmailField(unique=True)
    username = models.CharField(max_length=150)
    telephone = models.CharField(max_length=11,unique=True)
    is_active = models.BooleanField(default=True)

    USERNAME_FIELD = 'telephone'
    REQUIRED_FIELDS = []

    objects = UserManager()

    def get_full_name(self):
        return self.username

    def get_short_name(self):
        return self.username
```
其中password和last_login是在AbstractBaseUser中已经添加好了的，我们直接继承就可以了。然后我们再添加我们想要的字段。比如email、username、telephone等。这样就可以实现自己想要的字段了。但是因为我们重写了User，所以应该尽可能的模拟User模型：

 + USERNAME_FIELD：用来描述User模型名字字段的字符串，作为唯一的标识。如果没有修改，那么会使用USERNAME来作为唯一字段。
 + REQUIRED_FIELDS：一个字段名列表，用于当通过createsuperuser管理命令创建一个用户时的提示。
 + is_active：一个布尔值，用于标识用户当前是否可用。
 + get_full_name()：获取完整的名字。
 + get_short_name()：一个比较简短的用户名。
 + 重新定义UserManager：我们还需要定义自己的UserManager，因为默认的UserManager在创建用户的时候使用的是username和password，那么我们要替换成telephone。示例代码如下：
```
class UserManager(BaseUserManager):
    use_in_migrations = True

    def _create_user(self,telephone,password,**extra_fields):
        if not telephone:
            raise ValueError("请填入手机号码！")
        user = self.model(telephone=telephone,*extra_fields)
        user.set_password(password)
        user.save()
        return user

    def create_user(self,telephone,password,**extra_fields):
        extra_fields.setdefault('is_superuser',False)
        return self._create_user(telephone,password)

    def create_superuser(self,telephone,password,**extra_fields):
        extra_fields['is_superuser'] = True
        return self._create_user(telephone,password)
```
在创建了新的User模型后，还需要在settings中配置好。配置AUTH_USER_MODEL='appname.User'。

如何使用这个自定义的模型：比如以后我们有一个Article模型，需要通过外键引用这个User模型，那么可以通过以下两种方式引用。
第一种就是直接将User导入到当前文件中。示例代码如下：
```
 from django.db import models
 from myauth.models import User
 class Article(models.Model):
     title = models.CharField(max_length=100)
     content = models.TextField()
     author = models.ForeignKey(User, on_delete=models.CASCADE)
```

这种方式是可以行得通的。但是为了更好的使用性，建议还是将User抽象出来，使用 from django.contrib.auth import get_user_model来表示。示例代码如下：
```
 from django.db import models
 from django.conf import settings
 from django.contrib.auth import get_user_model

 class Article(models.Model):
     title = models.CharField(max_length=100)
     content = models.TextField()
     author = models.ForeignKey(get_user_model(), on_delete=models.CASCADE)
```

