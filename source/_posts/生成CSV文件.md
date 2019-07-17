---
title: 生成CSV文件
date: 2018-12-12 23:46:05
tags:
  - Django2.0
category: 学习笔记
---

如果在网站中需要下载一些 数据，就可以通过 HttpResponse 构造相应的文件对象。并且作为附件的形式响应给浏览器。提供给用户下载。比如生成一个 csv 文件。
<!-- more -->

```
import csv
from django.http import HttpResponse

def csv_view(request):
   response = HttpResponse(content_type='text/csv')
   response['Content-Disposition'] = 'attachment; filename="somefilename.csv"'
   csv_file = csv.writer(response)
   csv_file.writerow(['username', 'age'])
   csv_file.writerow(['threefire', 18])
   return response
```

1、首先初始化 HttpResponse 的时候，指定 Content-Type 为 text/csv，这将告诉浏览器这是一个 csv 格式文件，如果不指定类型，浏览器就会默认为 html 格式处理。
2、在 response 中添加一个 Conten-Dispositon 头，指定值为 attachment；那么就会将这文件作为附件下载，而不是显示。filename='somefilename.csv' 可以指定文件的名字。
3、使用 csv 模块的 wirter 方法，将数据写入到 response。
```
def index2(request):
    response = HttpResponse(content_type='text/csv')
    response['Content-Disposition'] = 'attachment; filename="附件.csv"'
    # with open('xxx.csv') as fb:
    #     csv.writer(fb)
    files = csv.writer(response)
    files.writerow(['username', 'age'])
    files.writerow(['threefire', 18])
    return response
```


## 使用模板的形式传递数据
 将一个模板文件作为 response 的 content 传入进去就行了。
  模板文件
 ```
 {% for row in rows %}{{ row.0}} {{row.1}}
{% endfor %}
 ```

 ```
from django.template import loader
 
 def index3(request):
    response = HttpResponse(content_type='text/csv')
    response['Content-Disposition'] = 'attachment; filename="附件.csv"'
    content = {
        'rows': [
            ['usernaem', 'age'],
            ['threefire', 18]
    ],
    }
    template = loader.get_template('abc.txt')
    csv_template = template.render(content)
    response.content = csv_template
    return response
 ```


## 较大的 CSV 文件
 因为较大的文件需要保持长时间连接，HttpResponse 无法满足这个要求，只能使用 StreamingHttpTesponse 对象，这个类继承 HttpResponseBase。将数据作为一个流返回给客户端。
 ```
 def large_csv_view(request):
    response = StreamingHttpResponse(content_type='text/csv')
    response['Content-Disposition'] = 'attachment; filename="large.csv"'
    rows = ('Row {}, {}'.format(row, row) for row in range(100000))
    response.streaming_content = rows
    return response
 ```

 + 使用 StreamingHttpResponse 必须传递一个可迭代对象。
 + 这个类没有属性content，相反是streaming_content。
 + 这个类的streaming_content必须是一个可以迭代的对象。
 + 这个类没有write方法，如果给这个类的对象写入数据将会报错。
注意：StreamingHttpResponse会启动一个进程来和客户端保持长连接，所以会很消耗资源。所以如果不是特殊要求，尽量少用这种方法。