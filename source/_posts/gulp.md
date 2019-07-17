---
title: 前端工具 gulp
date: 2019-01-16 00:30:23
tags:
  - gulp
category: 工具
---

gulp 是一种自动化开发流程的工具，配置好 gulp 后，可以自动处理好一些工作，比如写完 css 后，要压缩成 .min.css,写完 js 后，要做混淆和压缩，图片压缩等。gulp 都可以完成。
<!-- more -->

## 安装gulp
### 1、创建本地包管理环境
使用 npm init 命令在本地生成一个 package.json 文件。这个文件是用来记录当前项目前端依赖了哪些包，以后别人拿这个项目后，不需要 node_modules 文件夹(依赖包都在这个文件夹)，只需要执行 npm install 命令，就会自动把 package.json 下 devDependencies 中指定的依赖包安装好。

### 2、安装gulp
```
# 安装在系统级目录中
npm install gulp -g

# 安装在本地
npm install gulp --save-dev
```
"--save-dev" 就是将安装的包添加到 package.json 下的 devDependencies 依赖中，以后通过 npm install 即可自动安装。devDependencies 这个是用来记录开发环境中使用的包，如果想要记录生产环境中使用的包，那么在安装包的时候使用 "npm install xx --save" 就会记录到 package.json 下的 dependencies 中， dependencies 是专门用来记录生产环境下的依赖包的。

### 3、创建gulp任务
要使用 gulp 来流程化开发工作，首先要在项目的根目录下创建一个 gulpfile.js 文件，然后在 gulpfile.js 中写如下代码。
```
<!-- 变量 gulp 调用 gulp包 -->
var gulp = require('gulp');

<!-- 监控 名为 greet 的任务, 执行函数中代码 -->
gulp.task('greet', function () {
    console.log('hello world');
})
```
 + 通过 require 语句引用已经安装的第三方依赖包，这个 require 只能是引用当前项目的，不能引用全局下的。require 语句是 node.js 独有的，只能在node.js 环境下使用。
 + gulp.task 是用来创建一个任务。gulp.task 的第一参数是命令的名字，第二个参数是一个函数，就是执行这个命令的时候会做什么事情，都写在这个函数里面。
 + 写完以下代码后，以后如果想要执行 greet 命令，那么只需要进入到项目所在的路径，然后终端使用 gulp greet 即可执行。

4、创建处理css文件的任务
gulp 只是提供一个框架，如果要实现一些复杂的功能，比如 css 压缩，那么还需要安装一下 gulp-cssnano 插件，gulp 相关的插件安装也是通过 npm 命令安装，安装方式跟其他包是一模一样的。对 css 文件的处理，需要做的事情就是压缩，然后在将压缩后的文件放到指定目录下(不要与原来的css文件重合)，
```
# 安装 gulp-cssnano
>>> npm install gulp-cssnano --save-dev

# gulpfile.js
var cssnano = require('gulp-cssnano')

gulp.task('css', function(){
    gulp.src('.css/*.css')
    .pipe(cssnano())
    .pipe(gulp.dest('.css/dist/'))
    })
```
 + gulp.task：创建一个 css 处理任务
 + gulp.src：找到当前 css 目录下所有以 .css 结尾的 css 文件。
 + pipe：管道方法，将上一个方法返回结果传给另一个处理器。如以上 cssnano
 + gulp.dest: 将处理完后的文件，放到指定目录下。

5、修改文件名
压缩完 css 文件后，如果还想给文件改个名字，比如增加一个 .min.css
的后缀，这样就可以轻易分辨出那是压缩处理后的文件了，要实现这个功能需要 “gulp-rename” 插件。
```
>>> npm install gulp-rename --save-dev

# gulpfile.js
var rename = require('gulp-rename')

gulp.task('css', function (){
    gulp.src('.css/*.css')
    .pipe(cssnano)
    .pipe(rename({'suffix': ".min"}))
    .pipe(gulp.dest('.css/dist/'))
    })
```
 + .pipe(rename({'suffix': '.min'})): 使用 rename 方法，并且传递一个对象参数，指定修改名字的规则为添加一个 .min 后缀名。gulp-rename 插件还有其他指定文件名的方式，参考文档 "https://www.npmjs.com/package/gulp-rename"。

6、创建处理js文件任务
处理 js 文件，需要 gulp-uglify 插件
```
>>> npm install gulp-uglify --save-dev

# var uglify = require('gulp-uglify')

gulp.task('js', function(){
    gulp.src('.js/*.js')
    .pipe(uglify())
    .pipe(rename({"suffix": ".min"}))
    .pipe(gulp.desk('.dist/js/'))
    })
```
 + pipe(uglify())： 处理 js 文件压缩和修改变量名等处理，更多参数查看"
 + https://github.com/mishoo/UglifyJS2#minify-options"

7、合并多个文件
在网页开发中，为了加快网页的渲染速度，有时候会将多个文件压缩成一个文件，从而减少请求的次数，要拼接文件，需要 "gulp-concat" 插件
```
>>> npm install gulp-concat --save-dev

# gulpfile.js
var concat = require('gulp-concat')

gulp.task('js', function(){
    gulp.src('.js/*.js')
    .pipe(concat('index.min.js'))
    .pipe(uglify())
    .pipe(rename({"suffix": ".min"}))
    .pipe(gulp.desk('.dist/js/'))
    })
```

8、压缩图片
压缩图片可以减少图片的大小，可以采用 "gulp-imagemin"插件来实现
未必免对图片重复压缩可以使用 "gulp-cache" 插件。
```
>>> npm install gulp-imagemin --save-dev
>>> npm install gulp-cache --save-dev

# gulpfile.js
var imagemin = require('gulp-imagemin');
var cache = require('gulp-cache');

gulp.task('imgae', function(){
    gulp.src('./images/*.*')
    .pipe(cache(imagemin()))
    .pipe(gulp.dest('dist/images/'))
    });
```

9、检测代码修改，自动刷新浏览器
以上所有的任务，我们都需要手动在终端去执行，这样太麻烦。gulp 可以监听文件，只要文件有修改，我们就可以让它去执行一些任务
```
gulp.task('watch', function(){
    // 监听所有的 css 文件，然后执行 css 这个任务
    gulp.watch(".css/*.css", ['css'])
    })
```
以后只要在终端执行了 gulp watch 命令就可以监听 css 文件，然后自动执行 css 任务。

10、更改文件后，自动刷新浏览器
以上实现更改一些 css 文件后，可以自动执行处理 css 的任务，但还需要手动刷新浏览器才能看到效果。要自动刷新浏览器，可以使用 browser-sync 插件。
```
>>> npm install browser-sync --save-dev

# gulpfile.js

var gulp = require("gulp")
var cssnano = require("gulp-cssnano")
var rename = require("gulp-rename")
var bs = require("browser-sync").create()

gulp.task("bs",function () {
    bs.init({
        'server': {
            'baseDir': './'
        }
    });
});

// 定义一个处理css文件改动的任务
gulp.task("css",function () {
    gulp.src("./css/*.css")
    .pipe(cssnano())
    .pipe(rename({"suffix":".min"}))
    .pipe(gulp.dest("./css/dist/"))
    .pipe(bs.stream())
});

// 定义一个监听的任务
gulp.task("watch",function () {
    gulp.watch("./css/*.css",['css'])
});

// 执行gulp server开启服务器
gulp.task("server",['bs','watch'])
```
 + bs任务：这个任务开启一个 3000 端口，以后我们在访问html页面的时候，就需要通过http://127.0.0.1:3000的方式来访问了。然后接下来我们还定义了一个server任务。这个任务会去执行bs和watch任务，只要修改了css文件，那么就会执行css的任务，然后就会自动刷新浏览器。
 browser-sync更多的教程请参考："http://www.browsersync.cn/docs/gulp/"。


 ### BUG
1、处理路径写错，导致看不到生成的压缩文件
```
# 拼接路径少了斜杠
var path = {
    'html': './templates/*.*',
    'css': './src/css',
    'js': './src/js',
    'images': './src/images',
    'dist_css': './dist/css',
    'dist_js': './dist/js',
    'dist_images': './dist/images',
};

gulp.task("css", function () {
    gulp.src(path.css+"*.css")
        .pipe(cssnano())
        .pipe(rename({"suffix": ".min"}))
        .pipe(gulp.dest(path.dist_css))
        .pipe(bs.stream())
});
```
2、执行任务报"dest.on is not a function" 错误, 函数掉了括号
```
# 错
 .pipe(bs.stream)

# 改
 .pipe(bs.stream())

```