---
title: Django表单
date: 2018-12-17 23:25:55
tags:
  - Django2.0
category: 学习笔记
---

在 html 中表单是用来将数据提交到服务器的，不管后台使用的什么语言框架，html 中的 form 表单数据通过 input 标签就可以提交，Django 中的表单可以渲染表单模板，也可以验证表单数据是否合法。
<!-- more -->

## 在 Django 中使用表单
1、要在 Django 中使用表单，首先要定义一个表单类。一般的在 app 中创建一个 forms.py
文件来专门写表单类。表单类继承 django.forms.Form。
```
# forms.py
class MessgeBoardForm(forms.Form)：
    title = forms.CharField(max_length=3, label='标题', min_length=2, error_messages={'min_length':'标题字段不符合要求!'})
    content = forms.CharField(widget=forms.Textarea, label='内容')
    email = forms.EmailField(label='邮箱')
    reply = forms.BooleanField(required=False, label='回复')
```
2、然后在视图中，根据是 GET 还是 POST 请求来做相应的操作，如果是 GET 请求，那么返回一个空表单。 如果是 POST 请求，那么将数据进行校验。
```
from forms import MessageBoarForm

# views.py
class IndexView(View):
    def get(self, request):
        form = MessageBoarForm()
        return render(request, 'index.html', {'form': form})

    def post(self, request):
        form = MessageBoardForm(request.POST):
        if form.is_valid():
            title = form.cleaned_data.get('title')
            content = form.cleaned_data.get('content')
            email = form.cleaned_data.get('email')
            reply = form.cleaned_data.get('reply')
            return HttpRessponse('success')
        else:
            print(form.errors)
            return HttpResponse('fail')
```

如果浏览器是 GET 请求的话，就返回一个 form 给模板，模板就可以根据 form 来生成一个表单的 html 代码。在使用 POST 请求的时候，就根据提交上来的数据，构建一个新的表单，这个表单是用来验证数据是否合法的。如果数据验证通过了，那么就可以通过 cleaned_data 来获取相应的数据。

```
# index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>表单页面</title>
</head>
<body>
<form action="index" method="post">
    <table>
        {{ form.as_table }}
        <tr>
            <td></td>
            <td><input type="submit" value="提交"></td>
        </tr>
    </table>
</form>
</body>
</html>
```

用 form.as_table 可以在 html 中直接生成表单，将 forms.py 中的 Filed 字段渲染到前端页面中。

## 表单验证数据
在 form 类中，可以用 Field 对象数据验证，希望前端提交的数据是什么类型，那么就使用什么类型的 Field。

### CharField
用来接收文本，参数：
+ max_length: 这个字段值的最大小长度。
+ min_length: 这个字段值的最小长度。
+ required: 这个字段是否是必须的。默认是必须的。
+ error_meddages: 在某个条件验证失败的时候，给出错误信息。

### EmailField
用来接收邮件，会自动验证邮件是否合法。错误信息的 key：required、invalid。

### FloatField
用来接收浮点数类型，并且如果验证通过后，会将这个字段的值转换为浮点类型。
参数:
   + max_value: 最大的值。
   + min_value：最小的值
错误信息的 key: required、invalid、max_value、min_value。

### IntegerField
用来接收整形，并且验证通过后，会将这个字段的值转换为整形。
参数：
  + max_value: 最大的值。
  + min_value：最小的值。
错误信息的 key：required、invalid、max_value、min_value。

### URLField
用来接收 url 格式的字符串。
错误信息的 key: required、invalid。

## 验证器
上面的 Field 字段之所以可以对数据类型进行校验，其实本质上也是使用了验证器，验证器都在 validators 类中，可以在 Field 中提供validators 参数来指定验证器。常用验证器
 + MaxValueValidator：验证最大值。
 + MinValueValidator：验证最小值。
 + MinLengthValidator：验证最小长度。
 + MaxLengthValidator：验证最大长度。
 + EmailValidator：验证是否是邮箱格式。
 + URLValidator：验证是否是URL格式。
 + RegexValidator：如果还需要更加复杂的验证，那么我们可以通过正则表达式的验证 RegexValidator。
比如现在要验证手机号码是否合格，那么我们可以通过以下代码实现：
```
class MyForm(forms.Form):
     telephone = forms.CharField(validators=[validators.RegexValidator("1[345678]\d{9}",message='请输入正确格式的手机号码！')])
```

## 自定义验证器
当内置的验证器不能满足需要，就可以自己定义验证器。针对某个一个字段进行验证，只需要在 form 类中，定义一个 "clean_字段名" 要验证的字段名的方法。在对数据验证的时候就会自动调用这个方法。对这个字段进行验证，如果验证通过就返回这个数据。
```
class MyForm(forms.Form):
    telephone = forms.CharField(validators=[validators.RegexValidator("1[345678]\d{9}",message='请输入正确格式的手机号码！')])

    def clean_telephone(self):
        telephone = telephone.cleaned_data.get('telephone')
        exists = User.objects.filter(telephone=telephone).exists()
        if exists:
            raise forms.ValidationError("手机号码已经存在！")
        return telephone
```

如果同时要对多个字段进行验证，那么就需要重写 clean 方法。比如对两次输入的密码进行验证。
```
class MyForm(forms.Form):
    telephone = forms.CharField(validators=[validators.RegexValidator("1[345678]\d{9}",message='请输入正确格式的手机号码！')])
    pwd1 = forms.CharField(max_length=12)
    pwd2 = forms.CharField(max_length=12)

    def clean(self):
        cleaned_data = super().clean()
        pwd1 = cleaned_data.get('pwd1')
        pwd2 = cleaned_data.get('pwd2')
        if pwd1 != pwd2:
            raise forms.ValidationError('两个密码不一致！')
        else:
            return cleaned_data

```

## 提取错误信息
如果验证失败了，那么有一些错误信息是我们需要传给前端的。这时候我们可以通过以下属性来获取：
+ form.errors：这个属性获取的错误信息是一个包含了html标签的错误信息。
+ form.errors.get_json_data()：这个方法获取到的是一个字典类型的错误信息。将某个字段的名字作为key，错误信息作为值的一个字典。
+ form.as_json()：这个方法是将form.get_json_data()返回的字典dump成json格式的字符串，方便进行传输。
返回的错误消息
```
{'username': [{'message': 'Enter a valid URL.', 'code': 'invalid'}, {'message': 'Ensure this value has at most 4 characters (it has 22).', 'code': 'max_length'}]}
```
上述方法获取的字段的错误值，都是一个比较复杂的数据，为了简单化只需要将需要去的错误信息返回给前端。可以定义一个方法
```
def get_errors_message(self):
    errors = self.errors.get_json_data()
    new_message = {}
    for key,messages_dict in errors.items():
        messages = []
        for message in messages_dict:
            messages.append(message['message'])
        new_message[key] = messages
    return new_message
```

form 表单字段都有一个 errors_message 参数是用来保存提示的错误信息，当字段验证不通过时，这个参数的值将作为错误消息抛出，所以可以利用这个参数的值，自定义错误消息。

## ModelForm
在写表单的时候，如果发现表单中的 Field 和模型中的 Field 基本一样。而且表单中需要验证的数据也是模型中需要保存的。那么就可以将模型中的字段和表单中的字段进行绑定。
```
# models.py
from django.db import models
from django.core import validators

class Article(models.Model):
    title = models.CharField(max_lengt=10, validators=[validators.MinlengthValidator(limit_valu=30)])
    content = models.TextField()
    author = models.CharField(max_length=100)
```
在写表单的时候
```
# forms.py
from django import forms

class MyForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = "__all__"
```
MyForm 继承 forms.ModelForm , 然后在表单中定义一个 Meta 类，在 Meta 类中指定了 model=Article , 这个属性就是指定要绑定的 models 类。fields="__all__" 这属性指定将 Article 模型中所有的字段都复制过来进行验证。如果只是要对模型中的几个字段进行验证。那么就给 fields 指定一个列表。
```
# 需要验证的字段列表
      fields = ['title', 'content']
```
如果要验证的字段比较多，可以用 exclude 来代替 fields。exclude 属性将不需要的字段排除掉
```
# 不需要验证的字段
     exclude = ['author']
```

## 自定错误消息
使用 ModelForm ，因为字段都不是在表单中定义的，而是在模型中，因此错误消息无法在字段中定义，那么这个时候可以在 Meta 类中，定义 error_messages, 然后把相应错误消息写到里面去。
```
class MyForm(forms.ModelForm):
    # 提取错误消息
    def get_error(self):
        new_error = []
        error = self.errors.get_json_data()
        for messages in error.values():
            for message_dict in messages:
                for key, message in message_dict.items():
                    if key == 'message':
                        new_error.append(message)
        return new_error

    class Meta:
        model = Article
        exlcude = ['author']
        # 定义错误消息
        error_messages = {
            'title': {
                'max_length': '最多不能超过30个字符',
                'min_length': '最少不能少于3个字符'
            }，
            'content': {
                'required': '必须输入content'
            }
        }
```

## save方法
ModelForm 还有 save 方法，可以在验证完成后直接调用 save 方法，就可以将这个数据保存到数据库中。
```
# views.py
form = MyForm(requst.POST)
if form.is_valid():
    form.save()
    return HttpResponse('succes')
else:
    print(form.errors.get_json_data())
    return HttpResponse('fail')
```

直接进行保存要保证 models 需要的字段，都被表单验证通过了。如果表单验证的只是个别字段，那么可以在 form.save(commit=False) 中传递 commit=False 参数，就只会生成这个模型对象，而不把对象真正插入到数据库中就不会报错。再获得其他要保存的字段后再进行 save() 操作。
```
form = MyForm(requst.POST)
if form.is_valid():
    article = form.save(commit=False)
    article.author = 'threefire'
    article.save()
    return HttpResponse('succes')
else:
    print(form.error.get_json_data())
    return HttpResponse('fail')
```

