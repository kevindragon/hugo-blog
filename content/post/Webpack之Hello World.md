---
title: Webpack之Hello World
date: 2016-08-06 14:13:55
tags: [Webpack, webpack-dev-server]
---

QQ: 380800878, 微信: kittenll

最近自己在做一些项目的时候使用Restful和react来做到前后端分离，感受到开发效率的成倍提升，故在此分享一下前端使用webpack作为前端的构建工具。

[Webpack](http://webpack.github.io)是一个前端资源加载，编译，打包工具，可以把多个分散的功能模块javascript, css打包压缩到一个模块当中；合并图片。Webpack使用使用各种的loader来加载编译对应的资源，在网上也有非常多的loader。

### 0、安装

1. 全局安装：

```bash
npm install webpack -g
```

还有一个开发者工具，此工具非常实用，后面介绍：

```bash
npm install webpack-dev-server -g
```

2. 也可以使用本地模式随源码一起发行：

```bash
npm init
npm install webpack --save-dev
npm install webpack-dev-server --save-dev
```

随源码一起发行的好处就是不需要别人也强制全局安装，别人拿到源码后直接`npm update`后即可使用

### 1、简单使用

在介绍使用配置文件之前，先来看一下官方给出的最简单的编译例子：

新建一个目录`hello`，进行到`hello`目录，新建一个javascript文件`entry.js`，内容如下：

```javascript
document.write("It works.");
```

然后新建一个`index.html`文件，内容如下：

```html
<html>
    <head>
        <meta charset="utf-8">
    </head>
    <body>
        <script type="text/javascript" src="bundle.js" charset="utf-8"></script>
    </body>
</html>
```

编译`entry.js`到`bundle.js`：

```bash
webpack ./entry.js bundle.js
```

如果正常编译通过，你会看到如下的输出（这是我自己的输出，你的输出可能会不太一样）：

```
Hash: a41c6217554e666594cb
Version: webpack 1.13.1
Time: 67ms
    Asset     Size  Chunks             Chunk Names
bundle.js  1.42 kB       0  [emitted]  main
   [0] ./entry.js 29 bytes {0} [built]
```

使用浏览器打开`index.html`，你将会看到网页上呈现`It works.`

### 2、多文件

现在假设咱们的代码写了非常多，一个文件看起来已经非常长而且混乱不便于维护，这时候就需要把不同的功能代码分开到不同的文件，最终使用webpack编译打包压缩到一个文件。

在`hello`目录新建一个`content.js`文件，内容如下：

```javascript
module.exports = "It works from content.js.";
```

把`entry.js`修改成下面的内容，这一行是包含`content.js`的功能，然后写到网页上面：

```javascript
document.write(require("./content.js"));
```

编译：

```bash
webpack ./entry.js bundle.js
```

刷新浏览器打开的`index.html`，你将会看到如下内容：

```
It works from content.js.
```

### 3、CSS

是的你没有看错，webpack也可以加载编译打包压缩css文件。

再来新建一个css文件`style.css`，内容如下，把内页的背景变成黄色：

```css
body {
    background: yellow;
}
```

修改`entry.js`，在原来的内容上面添加一行包含css文件：

```javascript
require("!style!css!./style.css");
document.write(require("./content.js"));
```

继续编译：

```bash
webpack ./entry.js bundle.js
```

这时候你会看到一个错误信息：

```
ERROR in ./entry.js
Module not found: Error: Cannot resolve module 'style' in ......
```

是时候loader出场了，因为在`entry.js`里面require了css代码，但是前面安装的webpack里面不带有css loader，所以编译css的时候会出错，解决办法就是：

```bash
npm install css-loader style-loader
```

再编译后，再次刷新浏览器打开的index.html，你将会看到网页的背景颜色全部变成了黄色。

`css-loader`加载、编译css资源文件，而`style-loader`会把编译之后的css内容放入到`<style>`标签内插入到dom的head里面。

在`entry.js`里面使用了`!style!css!./style.css`来告诉webpack需要使用css-loader和style-loader来处理`style.css`，也可以不写`!style!css!`，而在编译的时候指定，简化`entry.js`文件如下：

```javascript
require("./style.css");
document.write(require("./content.js"));
```

这时候编译需要指定多一个参数了`--module-bind 'css=style!css'`：

```bash
webpack ./entry.js bundle.js --module-bind 'css=style!css'
```

### 4、配置文件
webpack使用的配置文件名是`webpack.config.js`，在`hello`新建此文件，内容如下：

```javascript
module.exports = {
    entry: "./entry.js",
    output: {
        path: __dirname,
        filename: "bundle.js"
    },
    module: {
        loaders: [
            { test: /\.css$/, loader: "style!css" }
        ]
    }
};
```

`entry`：是入口程序，从哪个文件开始编译
`output`：指定编译后的输出目录和文件名
`module`里面的`loaders`数组指定各种`loader`，处理各种css, image, url, ttf等待文件。

这里指定的入口是`entry.js`文件，输出到当前目录的`bundle.js`文件，如果碰到代码require的css代码（在`test`处配置，这后面可以写任何js的正则表达式），使用`style-loader`和`css-loader`加载编译。

编译：

```bash
webpack
```

有了配置文件这里只需要使用一个webpack的命令就可以进行编译，省了不少打字的时间。

### 5、watch
现在在命令行里面输入：

```bash
webpack --watch
```

然后修改一下代码，webpack会实时的帮你编译，是不是又省了不少打字的时间。

### 6、webpack-dev-server
这是一个神器，特别是结合React的时候可以做到源代码修改的时候，不需要刷新浏览器就可以看到修改后的结果，这又省了不少时间，还可以做到可视化编程。费话不多说，首先需要安装：

    npm install webpack-dev-server -g

命令行会输出类似如下的内容：

```
http://localhost:8080/webpack-dev-server/
webpack result is served from /
content is served from /Users/Dragon/awesome/Javascript/hello
Hash: ad76e97b228926df8b02
Version: webpack 1.13.1
Time: 812ms
    Asset     Size  Chunks             Chunk Names
bundle.js  11.8 kB       0  [emitted]  main
chunk    {0} bundle.js (main) 9.9 kB [rendered]
    [0] ./entry.js 66 bytes {0} [built]
    [1] ./style.css 927 bytes {0} [built]
    [2] ./~/.0.23.1@css-loader!./style.css 197 bytes {0} [built]
    [3] ./~/.0.23.1@css-loader/lib/css-base.js 1.51 kB {0} [built]
    [4] ./~/.0.13.1@style-loader/addStyles.js 7.15 kB {0} [built]
    [5] ./content.js 46 bytes {0} [built]
webpack: bundle is now VALID.
```

在浏览器输入[http://localhost:8080/webpack-dev-server/](http://localhost:8080/webpack-dev-server/)，在最上方会有一个`App ready.`的黑条条，下面的iframe加载的就是`index.html`的内容。

把`entry.js`修改成如下的内容：

```javascript
require("./style.css");
document.write(require("./content.js"));
document.write('<br>Hello World!');
```

回到浏览器你会发现，网页的内容已经改变了，下面多了一行Hello World!。

这个Hello World!有点长，呵呵！^_^



