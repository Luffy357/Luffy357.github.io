---
title: Django的模板
date: 2018-12-05 17:42:07
tags: 
  - Django2.0
category: 学习笔记
---


DTL模板是一种带有特殊语法的HTML文件，这个HTML文件可以被Django编译，可以传递参数进去，实现数据动态化。在编译完成后，生成一个普通的HTML文件，然后发送给客户端

<!-- more -->

## 渲染模板
有以下两种方式:
  1.render_to_string: 找到模板将模板渲染成 Python 的字符串格式，然后通过 HTTPResponse 类包装成一个 HTTPResponse 对象返回回去。

```
from django.template.loader import render_to_string
from django.http import HttpRseponse

def book_detail(request, book_id):
    html = render_to_string("detail.html")
    return HttpResponse(html)
```

  2.render：

````
from django.shortcuts import render

def book_list(request):
    return render(request, 'list.html')
````

## 模板的查找路径

在项目中的 settings.py 文件中，有一个 TEMPLATES 配置，这个配置包含了模板引擎的配置，模板查找路径的配置，模板上下文的配置等，

1.DIRS:  这是一个列表，在这个列表中可以存放所有的模板路径，以后在视图中使用 render 或 render_to_string 渲染模板的时候，会在这个列表的路径中查找模板。
2.APP_DIRS:  默认为True ，这个设置为 True 后，会在INSTALLED_APPS 的安装了的 APP 下的 templates 文件夹中查找模板
3.查找顺序：render('list.html'), 先会在DIRS 这个列表中依次查找路径下有没有这个模板， 如果有就返回，如果 DIRS 列表中所有路径都没有找到，那么会先检查当前这个视图所处的 app 是否已经安装， 如果已经安装了， 那么就先在当前这个 app 下的 templates 文件夹中查找模板，如果没有找到，那么会在其他已经安装了的 app 中的 templates 中查找，如果所有路径都没有找到， 那么会抛出一个 TemplateDoesNotExist 的异常。

## 模板变量

Django 渲染模板的时候，可以通过 context 变量传递到模板中去，在模板中变量的命令和 python 变量命名规则一样。

```
# index.html
<p>{{ username }}</p>

# views.py
def index(request):
    return render(request, 'index.html', context={'username': 'ThreeFire'})
```


  1.如果 context 传递是一个字典的时候，因为传递过去的变量本质是一个字典，所以字典的 keys 和 items 属性都可以用。在模板中同样也支持点( . )的方式读取值：

```
# views.py
def index(request):
    context = {
       'person': {
      'username': 'ThreeFire',
      'age': 28,
           }
    }
    return render(request, 'index.html, context=context')

# index.html
<p>{{ person.username }}</p>
```

传递字典的时候需要注意，如果要访问字典的 key 对应的 value ，那么就是通过'字典.key' 的方式进行访问，不能用'[]'的方式，如果字典中有个key同名属性 keys 那么返回的不是这个字典的所有key，而是对应的值。同样同名 items 也一样。

   2.如果 context 传递的是一个对象：

```
# views.py
class Person():
    def __init__(self, name):
          self.name = name
 
def index(request):
    context = {
      'p': Person
    }

# index.html
<p>{{ p.name }}</p>
```

   3.如果 context 传递的是一个元组或列表也通过点的方式，从 0 开始。但它不可以是一个负数。
```
   # views.py
   def index(request):
       context = {
           'person': (1, 2, 3, 4)
       }
       return render(request, 'index.html', context=context)
       
   # index.html
   <p>{{ person.0 }}
```

## 模板标签

### if 标签
  模板中的if标签与 python 中的if基本相同，if 标签中可以使用 ==、!=、<、<=、>、>=、in、not in、is、is not 等判断运算符。在模板中需要 '&#123;&#37; &#37;&#125;'进行包裹。

```
# index.html
    {% if '小明' in name_list %}
        <p>小明</p>
    {% else %}
        <p>他不在</p>
    {% endif %}

```

### for...in....标签： 
for...in...类似于 Python 中的 for...in...。可以遍历列表、元组、字符串、字典等一切可以遍历的对象。

```
 {% for person in persons %}
     <p>{{ person.name }}</p>
 {% endfor %}

```

如果想要反向遍历，那么在遍历的时候就加上一个 reversed 。示例代码如下：

```
 {% for person in persons reversed %}
     <p>{{ person.name }}</p>
 {% endfor %}

```

遍历字典的时候，需要使用 items、keys 和 values等方法。在 DTL 中，执行一个方法不能使用圆括号的形式。遍历字典示例代码如下：

```
 {% for key,value in person.items %}
     <p>key：{{ key }}</p>
     <p>value：{{ value }}</p>
 {% endfor %}

```

在'for'循环中，'DTL'提供了一些变量可供使用。这些变量如下：

- forloop.counter：当前循环的下标。以1作为起始值。

- forloop.counter0：当前循环的下标。以0作为起始值。

- forloop.revcounter：当前循环的反向下标值。比如列表有5个元素，那么第一次遍历这个属性是等于5，第二次是4，以此类推。并且是以1作为最后一个元素的下标。

- forloop.revcounter0：类似于forloop.revcounter。不同的是最后一个元素的下标是从0开始。

- forloop.first：是否是第一次遍历。

- forloop.last：是否是最后一次遍历。

- forloop.parentloop：如果有多个循环嵌套，那么这个属性代表的是上一级的for循环。

### for...in...empty标签：
这个标签使用跟'for...in...'是一样的，只不过是在遍历的对象如果没有元素的情况下，会执行'empty'中的内容。示例代码如下：

```
 {% for person in persons %}
     <li>{{ person }}</li>
 {% empty %}
     暂时还没有任何人
 {% endfor %}
```

### with标签：
在模版中定义变量。有时候一个变量访问的时候比较复杂，那么可以先把这个复杂的变量缓存到一个变量上，以后就可以直接使用这个变量就可以了。示例代码如下：

```
 context = {
     "persons": ["张三","李四"]
 }

 {% with lisi=persons.1 %}
     <p>{{ lisi }}</p>
 {% endwith %}

```

有几点需要强烈的注意：
在 with 语句中定义的变量，只能在'&#123;&#37;with&#37;&#125;&#123;&#37;endwith&#37;&#125;'中使用，不能在这个标签外面使用。
定义变量的时候，不能在等号左右两边留有空格。比如'&#123;&#37; with lisi = persons.1&#37;&#125;'是错误的。
还有另外一种写法同样也是支持的：
```
    {% with persons.1 as lisi %}
        <p>{{ lisi }}</p>
    {% endwith %}
```

### url标签：
在模版中，我们经常要写一些'url'，比如某个'a'标签中需要定义'href'属性。当然如果通过硬编码的方式直接将这个'url'写死在里面也是可以的。但是这样对于以后项目维护可能不是一件好事。因此建议使用这种反转的方式来实现，类似于'django'中的'reverse'一样。示例代码如下：

```
 <a href="{% url 'book:list' %}">图书列表页面</a>
```

如果'url'反转的时候需要传递参数，那么可以在后面传递。但是参数分位置参数和关键字参数。位置参数和关键字参数不能同时使用。示例代码如下：

```
# path部分
    path('detail/<book_id>/',views.book_detail,name='detail')

    # url反转，使用位置参数
    <a href="{% url 'book:detail' 1 %}">图书详情页面</a>

    # url反转，使用关键字参数
    <a href="{% url 'book:detail' book_id=1 %}">图书详情页面</a>
```

如果想要在使用'url'标签反转的时候要传递查询字符串的参数，那么必须要手动在在后面添加。示例代码如下：

```
  <a href="{% url 'book:detail' book_id=1 %}?page=1">图书详情页面</a>
```

如果需要传递多个参数，那么通过空格的方式进行分隔。示例代码如下：

```
  <a href="{% url 'book:detail' book_id=1 page=2 %}">图书详情页面</a>
```

### spaceless标签：
移除html标签中的空白字符。包括空格、tab键、换行等。示例代码如下：

```
 {% spaceless %}
     <p>
         <a href="foo/">Foo</a>
     </p>
 {% endspaceless %}
```

那么在渲染完成后，会变成以下的代码：

```
 <p><a href="foo/">Foo</a></p>
```

spaceless 只会移除html标签之间的空白字符。而不会移除标签与文本之间的空白字符。看以下代码：

```
 {% spaceless %}
     <strong>
         Hello
     </strong>
 {% endspaceless %}
```

这个将不会移除 strong 中的空白字符。

### autoescape 标签：
开启和关闭这个标签内元素的自动转义功能。自动转义是可以将一些特殊的字符。比如&#60;转义成 html 语法能识别的字符，比如&#60;会被转义成&#60;，而&#62;会被自动转义成&#62;。模板中默认是已经开启了自动转义的。autoescape，如果要将HTML的代码显示成一个普通的字符串，只需要把on 成 off。示例代码如下：

```
 # 传递的上下文信息
 context = {
     "info":"<a href='www.baidu.com'>百度</a>"
 }

 # 模板中关闭自动转义
 {% autoescape on %}
     {{ info }}
 {% endautoescape %}
```

### verbatim标签：
   默认在 DTL 模板中是会去解析那些特殊字符的。比如'&#123;&#37;' 和 '&#37;&#125;'以及'&#123;&#123;'等。如果你在某个代码片段中不想使用 DTL 的解析引擎。那么你可以把这个代码片段放在 verbatim 标签中。示例代码下：

```
 {% verbatim %}
     {{if dying}}Still alive.{{/if}}
 {% endverbatim %}

```

## 模板过滤器

在模板中，有些时候需要对数据进行一些处理，在模板中通过过滤器来实现这个功能。

### add:
将传过来的值添加到原来的值上面，如果值与参数类型相同，那么进行相加，如果不一样那么进行拼接。

```
# index.html
{{ value|add:"2" }}
# value = 18 输入出结果为 20
# value = ’abc' 输入结果为 abc2
```

### cut:
移除值中指定的字符串

```
# index.html
{{ value|cut:'' }}
# 将 value 中要剔除的字符放入 ‘’ 中
```

### date:
将一个日期按照指定的格式显示。

```
# index.html
{{ value|date: 'Y-m-d' }}
# 显示2018-11-24
```

那么将会输出 2018/02/01 。其中 'Y' 代表的是四位数字的年份，'m' 代表的是两位数字的月份，'d'代表的是两位数字的日。

&#124; 格式字符 &#124; 描述                    &#124; 示例    &#124;
&#124; &#45;&#45;&#45;&#45; &#124; &#45;&#45;&#45;&#45;&#45;&#45;&#45;&#45;&#45;&#45;&#45;&#45;&#45;&#45;&#45;&#45;&#45;&#45;&#45;&#45;&#45; &#124; &#45;&#45;&#45;&#45;&#45; &#124;
&#124; Y    &#124; 四位数字的年份               &#124; 2018  &#124;
&#124; m    &#124; 两位数字的月份               &#124; 01&#45;12 &#124;
&#124; n    &#124; 月份，1&#45;9前面没有0前缀         &#124; 1&#45;12  &#124;
&#124; d    &#124; 两位数字的天                &#124; 01&#45;31 &#124;
&#124; j    &#124; 天，但是1&#45;9前面没有0前缀        &#124; 1&#45;31  &#124;
&#124; g    &#124; 小时，12小时格式的，1&#45;9前面没有0前缀 &#124; 1&#45;12  &#124;
&#124; h    &#124; 小时，12小时格式的，1&#45;9前面有0前缀  &#124; 01&#45;12 &#124;
&#124; G    &#124; 小时，24小时格式的，1&#45;9前面没有0前缀 &#124; 1&#45;23  &#124;
&#124; H    &#124; 小时，24小时格式的，1&#45;9前面有0前缀  &#124; 01&#45;23 &#124;
&#124; i    &#124; 分钟，1&#45;9前面有0前缀          &#124; 00&#45;59 &#124;
&#124; s    &#124; 秒，1&#45;9前面有0前缀           &#124; 00&#45;59 &#124;

### default:
如果值为False，比如&#91; &#93; ,  none, &#123;&#125;,  那么会使用 defaul 提供的默认值。

```
{{ value|default:"nothing" }}
```

### first:
返回列表/元组/字符串中的第一个元素

```
{{ value|first }}
```

### last:
返回列表/元组/字符串中的最后一个元素

```
{{ value|last }}
```

### floatformat:
使用四舍五入的方式格式化一个浮点类型，默认保留一位小数，如果传递参数，则根据参数保留相应位数的小数。

```
# 四舍五入保留1位小数
{{ hight|floatformat }}
# 四舍五入保留3位小数
{{ hight|floatformat:"3"}}
```

### join:
将列表/字典/字符串/用指定的字符进行拼接。

```
# ['a', 'b', 'c'] 将输入a/b/c
{{ value|join:'/' }}
```

### length:
获取一个列表/元组/字符串/字典的长度。

```
# 如果value 为none，返回 0
{{ value|length }}
```

### lower:
将值中的字符串全部转为小写

```
{{ value|lower }}
```

### upper:
将值中的字符串全部转为大写

```
{{ value|upper }}
```

### random:
在value 中随机的选择一个值

```
{{ value|random }}
```

### safe
因为django 模板默认开启了字符串转义，如果要传递html 等特殊效果的字符，会被转义直接显示，而达不到效果。而 safe 就是关掉默认的自动转义。

```
# 如果 value 是一个不包含特殊字符的字符串，那么会正常显示，如果 value 包含有 html 代码， 那么会进行渲染在显示。
{{ value|safe }}
```

### slice
与pyhton中的切片类似

```
# 提取前value 中前两位
{{ value|slice:'2'}}
```

### stringtags
删除 value 中的html 标签

```
{{ value|stringtags }}
```

### trucatechars
如果给定的字符串长度超过了过滤器指定的长度，那么会进行切割，并且会拼接三个点来作为省略。而且三个点也占位三个字符, 也就是说如果你想显示10个字符，你应该限定13个。

```
{{ value|truncatechars:13 }}
```

### truncatechars_html
与 truncatechars 功能一样，只是不会切割 html 字符。也就是切割的时候排除掉 html 字符。

```
{{ value|truncatechars_html:5 }}
```


## 自定义模板标签

​    1、自定义模板过滤器必须放在 app 中，并且这个app 必须要在 INSTALLED_APPS 中进行安装。

​    2、在这个 app 下创建一个 python 包叫做 templatetags 。再在这个包下面创建一个 python 文件, 在里面就可以写过滤器函数了。

​    3、在创建了存储过滤器的文件后，接下来就是在这个文件中写过滤器了。过滤器实际上就是python中的一个函数，只不过是把这个函数注册到模板库中，以后在模板中就可以使用这个函数了。但是这个函数的参数有限制，第一个参数必须是这个过滤器需要处理的值，第二个参数可有可无，如果有，那么就意味着在模板中可以传递参数，并且过滤器的函数最多只能有两个参数。在写完过滤器后，再使用'django.template.Library'将对象注册进去。

```
my_filter.py
    
from django import tempate
    
创建模板对象
    
register = template.Library()
    
过滤器函数
@register.filter() # 可以用装饰器注册
def mycut(value, myster):
    return value.replace(mystr)
    
将函数注册到模板库中
register.filter('mycut', mycut) 
```

4、在模板中，只要先 load 这个文件，就可以使用里面的过滤器了

```
# index.html 
{% load my_filter %}

{{ value|mycut }}
```

## 模板优化

1、有些模板的代码是重复的，因此可以抽取出来，以后那里需要就使用 include 标签进去就行了。

```
# header.html
<p>我是header</p>

# footer.html
<p>我是footer</p>

# main.html
{% include 'header.html' %}
<p>我是main内容</p>
{% include 'footer.html' %}
```

2、如果想在 include 子模板的时候，传递一些参数，那么可以使用 whit "参数名=值" 的形式

```
{% include "header.html" with username='huangyong' %}
```

## 模板继承

模板继承可以在父模板中先定义好一些子模板需要用到的代码，然后子模板直接继承就可以了，为了对应不同的子模板有不同内容，可以在父模板中定义 block 接口。子模板可以根据不同需求替换 block 中的内容。

```
{% load static %}
<!DOCTYPE html>
<html lang="en">
<head>
    <link rel="stylesheet" href="{% static 'style.css' %}" />
    <title>{% block title %}我的站点{% endblock %}</title>
</head>

<body>
    <div id="sidebar">
        {% block sidebar %}
        <ul>
            <li><a href="/">首页</a></li>
            <li><a href="/blog/">博客</a></li>
        </ul>
        {% endblock %}
    </div>
    <div id="content">
        {% block content %}{% endblock %}
    </div>
</body>
</html>
```

子模拟通过 extends 标签继承父模板，注意extends 标签必须在第一行
```
{% extends "base.html" %}

{% block title %}博客列表{% endblock %}

{% block content %}
    {% for entry in blog_entries %}
        <h2>{{ entry.title }}</h2>
        <p>{{ entry.body }}</p>
    {% endfor %}
{% endblock %}
```

如果在某个 block 中需要使用父模版的内容，那么可以使用'&#123;&#123;block.super&#125;&#125;'来继承。为了对应 block 开始与结束，在 endblokc 标签中也可以写上名字

```
{% block content %}{% endblock content %}
```

## 加载静态文件

在一个网页中，不仅仅只有一个 html 骨架，还需要 css 样式文件，js 执行文件以及一些图片等。因此在 DTL 中加载静态文件是一个必须要解决的问题。在 DTL 中，使用 static 标签来加载静态文件。要使用 static 标签，首先需要'&#123;&#37; load static &#37;&#125;'。加载静态文件的步骤如下：

1. 首先确保 django.contrib.staticfiles 已经添加到 settings.INSTALLED_APPS 中。

2. 确保在 settings.py 中设置了 STATIC_URL。

3. 在已经安装了的 app 下创建一个文件夹叫做 static，然后再在这个 static文件夹下创建一个当前 app 的名字的文件夹，再把静态文件放到这个文件夹下。例如你的 app 叫做 book，有一个静态文件叫做 zhiliao.jpg，那么路径为 book/static/book/zhiliao.jpg。&#40;为什么在 app 下创建一个 static 文件夹，还需要在这个 static下创建一个同 app 名字的文件夹呢？原因是如果直接把静态文件放在 static 文件夹下，那么在模版加载静态文件的时候就是使用 zhiliao.jpg ，如果在多个 app 之间有同名的静态文件，这时候可能就会产生混淆。而在 static 文件夹下加了一个同名 app 文件夹，在模版中加载的时候就是使用 app/zhiliao.jpg，这样就可以避免产生混淆。&#41;

4. 如果有一些静态文件是不和任何 app 挂钩的。那么可以在 settings.py 中添加 STATICFILES_DIRS，以后 DTL 就会在这个列表的路径中查找静态文件。比如可以设置为:

```
    STATICFILES_DIRS = [
        os.path.join(BASE_DIR,"static")
    ]
```

5. 在模版中使用 load 标签加载 static 标签。比如要加载在项目的 static 文件夹下的 style.css 的文件。那么示例代码如下：

```
    {% load static %}
    <link rel="stylesheet" href="{% static 'style.css' %}">
```

6. 如果不想每次在模版中加载静态文件都使用 load 加载 static 标签，那么可以在 settings.py 中的 TEMPLATES/OPTIONS 添加 'builtins'&#91;'django.templatetags.static'&#93;'，这样以后在模版中就可以直接使用 static 标签，而不用手动的 load了。

7. 如果没有在 settings.INSTALLED_APPS 中添加 django.contrib.staticfiles。那么我们就需要手动的将请求静态文件的 url与静态文件的路径进行映射了。示例代码如下：

```
   from django.conf import settings
   from django.conf.urls.static import static

   urlpatterns = [
       # 其他的url映射
   ] + static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```