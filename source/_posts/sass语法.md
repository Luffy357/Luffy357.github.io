---
title: sass语法
date: 2019-01-18 23:56:16
tags:
  sass
category: 前端
---

众所周知，css 不是一门编程语言，他没有像 js 和 python 那样拥有逻辑处理的能力，甚至导入其他的 css 文件中的样式都做不到，而 sass 就是为了解决 css 的这些问题。它允许你使用变量、嵌套规则、mixins、导入等功能。并且完全兼容 css 语法。sass 文件不能直接被网页所识别，写完 sass 后还有用估计转化为 css 才能使用。
<!-- more -->

## Sass 文件的后缀名
Sass 文件有两种后缀名，一个是 scss，一个是 sass。不同后缀名，相应的语法也不一样。这里使用 scss 的后缀名。sass 语法就是使用 scss 的后缀名语法。

## 使用gulp将sass准换为css
将 sass 文件转化为 css 文件的工具有很多，这里使用gulp来实现。还需要安装插件 gulp-sass 插件来完成。
```
>>> npm install gulp-sass --save-dev

# gulfile.js
gulp.task('css', function (){
    gulp.src(path.css+'*.scss')
        .pipe(sass(),on('error', sass.logError))
        .pipe(cssnano())
        .pipe(rename({'suffix': ".min"}))
        .pipe(gulp.dest(path.dist_css))
    })
```

## Sass基本语法
###注释
支持 /* comment */ 和 //注释 两种方式

### 嵌套
Sass 语法运行嵌套。比如 #main 下有一个类为 .header, 那么可以写成以下形式
```
#main{
    background: #fff;
    .header{
        width: 20px;
        height: 20px;
    }
}
```
这样写起来更加直观，一看就知道 .header 是在 #main 下的。

### 引用父选择器
有事在嵌套的子选择器中，需要使用父选择器。那么这时候可以通过 & 来表示。
```
a{
    font-weight: bold;
    text-decoration: none;
    &:hover{
        color:#888;
    }
}
```

### 定义变量
在 sass 中可以定义变量。对于比较常用的值，我们可以通过变量存储起来。以后要使用的时候直接用就可以了。 定义变量使用 $ 符合。
```
$mainWidth: 980px;
#main{
    width:$mainWidth;
}
```

### 运算
在 Sass 中支持运算。比如现在有一个容器总宽度是 900， 要在里面平均放三个盒子，那么我们可以通过变量来设置他们的宽度。
```
$mainWidth: 900px;
.box{
    width: $mainWidth/3;
}
```

### @import语法
在 css 中@import 只能导入 css 文件，而且对网站的性能有很大影响。而 Sass 中的 @import 则是完全实现了一套自己的机制。他可以直接将指定文件的代码拷贝到导入的地方。
```
@import "init.scss";
```

### @extend语法
有时候一个选择器中，可能会需要另一个选择器的样式，那么我们就可以通过 extend 来直接将指定选择器的样式加入进来。
```
.error{
    background-color: #fdd;
    border: 1px solid #foo;
}
.serious-error{
    @extend .error;
    border-width: 3px;
}
```

### @mixin语法
有时候一段样式代码。我们可能要用很多地方。那么我们可以把他定义成 mixin。 需要用的时候直接引用就可以了。
```
@mixin large-text{
    font:{
        family: Arial;
        size: 20px;
        weight: bold;
    }
    color: #ff0000;
}
```
如果其他地方想要使用这个 mixin 的时候，可以通过 @include 来包含进来。
```
.page-title{
    @include large-text;
    padding: 4px;
    margin-top: 100px;
}
```
@mixin也可以使用参数
```
@mixin sexy-border($color, $width){
    border: {
        color: $color;
        width: $width;
        style: dashed;
    }
}
```
那么以后再 include 的时候，就需要传递参数。
```
p{
    @include sexy-horder(blue, 1px);
}
```
更多教程："http://sass.bootcss.com/docs/sass-reference/"