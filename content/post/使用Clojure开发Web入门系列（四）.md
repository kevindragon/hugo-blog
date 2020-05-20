---
title: 使用Clojure开发Web入门系列（四）
date: 2018-07-01 21:32:08
tags: [Clojure, Web]
---

在[（三）](/2016/03/14/使用Clojure开发Web入门系列（三）)介绍并使用了Hiccup这个生成html的库，甚是方便，但是对于前后端分离的一些web站点也不是太方便。接下来介绍另外一种方法来渲染html页面，[selmer](https://github.com/yogthos/Selmer)。如果你是一个PHP程序员，那么你会发现他跟smarty有几分相似。

# 零、添加依赖

在`project.clj`的`dependencies`添加`[selmer "1.11.8"]`

# 一、修改代码

修改系列三里面的代码到selmer实现。

## home

首先在`resources`目录下添加`home.html`，内容如下：

```html
<h1>Home</h1>
```

在`resources`目录添加是因为selmer默认会去classpath指定的路径里面找对应的模板文件。

修改`core.clj`，在`:require`后面添加selmer：`[selmer.parser :refer [render-file]]`

修改`defroutes app`，在添加一条路由规则:

```clojure
(GET "/home" [] (render-file "home.html" {}))
```

访问`http://localhost:3000/home`，可以看到html模板里面的内容已经显示出来了。如果没有运行程序请执行：`lein run`

<!-- more -->

## form
现在来修改表单页面，使用`resources/index.html`作为模板文件，内容如下:

```html
<!DOCTYPE html>
<html>
  <body>
    <h1>Hello form</h1>
    <form action="/message" method="post">
      <div>
        <label for="name">名字:</label>
        <input name="name" type="text" />
      </div>
      <div>
        <label for="message">消息:</label>
        <textarea cols="80" name="message" rows="5"></textarea>
      </div>
      <div><input type="submit" value="提交" /></div>
    </form>
  </body>
</html>
```

修改`defroutes app`里面`"/"`的定义为：

```clojure
(GET "/" [] (render-file "index.html" {}))
```

访问`http://localhost:3000`，表单出来了。

## message
修改表单提交后的页面。

先在建立消息模板文件`resources/message.html`，内容如下：

```html
<!DOCTYPE html>
<html>
  <body>
    <h2>{{ name }}</h2>
    <p>{{ message }}</p>
  </body>
</html>
```

`{{ name }}`和`{{ message }}`是占位符，selmer把render-file的第二个map参数解析到模板文件里面，`{{ name }}`对应的是`:name`的值。

接着修改`core.clj`的`show-message`函数：

```clojure
(defn show-message [params]
  (let [{name "name" message "message"} params]
    (render-file "message.html" {:name name :message message})))
```

访问[`http://localhost:3000`](http://localhost:3000)`，填写表单，提交消息后就可以看到跟原来一样的效果了。现在clj文件的代码更加简短，模板文件也被分离出来了，这样更便于前端和后端的协作。

这其实是一个相对比较传统前后端分离的方案，后面介绍了ClojureScript后，还有比较时尚的玩法。

# 二、抽取layout

index.html和message.html两个文件还是有不少的相同代码，如果html模板文件内容比较多，引用了很多的css和js的话，就少产生不少冗余代码。

layout就应运而生。layout是一个大的框架，把相同的代码抽离出来，然后把不相同的部分在layout里面做面变量或者替换的方式，不同的页面做不同的替换。在selmer里面，有两个办法：一是使用extends；二是使用include。这里先使用extends方法。

那么先来定义`resources/layout.html`，使用`block`和`endblock`这个[tag](https://github.com/yogthos/Selmer#tags)定义一个body块，这个块及其内容可以被子模板的同名块替换。再来看子模板`resources/index.html`，第一行使用tag`extends`引用父模板`layout.html`（路径也必须要在classpath当中）。

同样的原理修改`message.html`。

修改完后再访问[`http://localhost:3000`](http://localhost:3000)，结果是一样的。

# filter

filter跟clojure里面的filter不是一个概念，selmer的filter是对模板里面的变量做转换，其实是一个转换函数，比如把输出的内容全部变成大写或者小写。这里实现消息名字大写，把`{{ name }}`改成`{{ name|upper }}`。需要注意一个竖线两边不能有空格，否则就不对了。

selmer已经内置了非常多的filter以供使用，像：截取`abbreviate`、日期格式`date`等等。

如果selmer提供的filter不能满足你的需求，你还可以实现自定义的filter：

首先在`:require`引入`[selmer.filters :refer [add-filter!]]`，然后调用`add-filter!`添加自定义filter。例如要实现自己的大写filter：

```clojure
(add-filter! :embiginate clojure.string/upper-case)
```

使用方法与内置的filter一样，在模板的变量后面加竖线和filter钟宇：

```
{{shout|embiginate}}
```

# tag

假设你想在模板输出变量内容的时候，如果变量保存的是一段html内容，你并不想把标签显示在页面上，而是解析html的内容，显示相应html的样式，这个可以使用selmer的tag来实现。selmer默认会把变量的输出进行html转义，以保证表单或者其他地方来的数据的安全性，防止跨站脚本攻击（XSS）。例如要实现消息内容html格式输出，在`{{ message }}`的外面加上`safe`和`endsafe`这个tag：

```
{% safe %}{{ message }}{% endsafe %}
```

selmer的tag分两种，一个是单一出现的，像上面layout介绍的`extends`；另外一种是成对出现的，像`safe`和`endsafe`。

selmer内置了很多的tag来解决模板当中常见的一些问题。像条件判断`if else`、`for`、`now`、`script`、`style`等等很实用的tag。

与filter一样，如果selmer内置的tag不能满足你的需求，可以实现自定义的tag。

需要在`:require`引入：`[selmer.parser :refer [add-tag!]]`。调用`add-tag!`定义tag，定义tag稍微复杂一点，因为tag有两种形式，一个是单一出现的tag，`add-tag!`接受两个或者三个参数，两个参数的是定义单一的tag，三个参数的是定义成对出现的tag。

先来看单一出来的tag的定义：

```clojure
(add-tag! :foo
  (fn [args context-map]
    (str "foo " (first args))))
```

先来看一下在模板里面如何调用：

```clojure
(render "{% foo quux %}" {})
```

```
=>"foo quux"
```

foo是tag的名字，第二个参数是一个函数，这个函数接受两个参数，第一个参数`args`是模板里面写的tag后面的数据，上面的例子里面的quux；还有一个`context-map`是一个上下文map，里面包含有这个tag执行时的相关信息。

再看一下成对出来的tag的定义：

```clojure
(add-tag! :uppercase
          (fn [args context-map content]
            (.toUpperCase (get-in content [:uppercase :content])))
          :enduppercase)
```

调用方式如下：

```clojure
(render "{% uppercase %}foo {{bar}} baz{% enduppercase %}" {:bar "injected"})
```

输出：

```
=>"FOO INJECTED BAZ"
```

这个自定义的tag把包裹起来的内容全部转换成大写。定义的时候需要写上成对出现的tag名字，中间也是一个处理函数，这个函数接受三个参数了，`args`、`context-map`和上面是一样的，多了一个`content`参数，`[:uppercase :content]`存的是tag包裹的内容。

完整[代码](https://github.com/kevindragon/learn-clojure-web)请：`git checkout 1.4`。
