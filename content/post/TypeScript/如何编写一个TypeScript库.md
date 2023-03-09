---
title: "如何编写一个TypeScript库"
date: 2023-03-09T20:53:42+08:00
tags: [TypeScript]
categories: [TypeScript]
---

编写模块化的代码是一个非常不错的想法，没有比编写一个库更加模块化了。那要如何编写一个TypeScript库呢？这就是本文的意图，本文的代码可以运行在**TypeScript 2.x 3.x和4.x**上。

<!-- more -->

## 第一步∶设置tsconfig.json

创建一个项目目录，本文目录名称为 `typescript-library` ，然后在这个目录下面添加配置文件 `tsconfig.json` 文件内容类似于下面这样：

typescript-library/tsconfig.json

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "es2015",
    "declaration": true,
    "outDir": "./dist"
  },
  "include": [
    "src/**/*"
  ]
}
```

这跟平常配置一个一般的项目差不多，唯一需要强调的一点就是，需要添加 `"declaration": true` 字段。有了这个配置，编译后的代码会生成以 `.d.ts` 结尾的文件，这些文件里面会包含代码里面的类型。如果别人在使用你写的代码，他们就会从TypeScript的类型系统中获得自动补全的好处，类型安全的好处。

关于其他选项，让我们快速浏览一下：如果你想让你的代码与大多数当前的node.js应用程序无缝运行，那么`"module": "commonjs"`模块编译器选项是必需的。如果你正在构建一个面向浏览器的库，那么请将其替换为`"module": "esnext"`。`"target": "es2015"`指定你的代码将被转译成哪个版本的JavaScript。这需要与你想要支持的最旧的node.js版本（或最旧的浏览器）保持一致。将编译目标选择为`es2015`可以使你的库与node.js版本8及以上版本兼容。`"outDir": "./dist"`将把你编译后的文件写入到dist文件夹中，`"include"`选项指定你的源代码存放位置。

## 第二步：实现你的库

就像你不写库代码，同样的方式。创建一个src文件夹，把你的所有库源代码（程序逻辑，数据，资产）都放在这个目录。

For this demo, we'll setup a silly `hello-world.ts` file, that looks like so:

为了这个demo，我们将写一个不太实用的`hello-world.ts`文件，看起来像这样：

typescript-library/src/hello-world.ts

```TypeScript
export function sayHello() {
  console.log('hi')
}
export function sayGoodbye() {
  console.log('goodbye')
}

```

## 第三步：创建一个 index.ts 文件

在您的`src`文件夹中添加一个`index.ts`文件。它的目的是导出您希望供消费者使用的库的所有部分。在我们的例子中，它只是：

typescript-library/src/index.ts

```TypeScript
export {sayHello, sayGoodbye} from './hello-world'
```

消费者以后可以像这样使用库：

someotherproject/src/somefile.ts

```TypeScript
import {sayHello} from 'hwrld'
sayHello();
```

You see that we have a new name here, "hwrld", we haven't seen anywhere yet. What is this name? It's the name of the library you're gonna publish to npm also known as the package name!

你看到我们有一个新的名字叫做“hwrld”，我们还没有在任何地方见过。这是什么名字呢？这是你要发布到npm的库的名字，也叫做包名！

## 第四步：配置 package.json

包名是消费者以后用来从您的库中导入功能的。 对于此demo，我选择了`hwrld`，因为它仍然可以在npm上使用。 包名通常位于`package.json`的顶部。 整个`package.json`看起来如下：

typescript-library/package.json

```json
{
  "name": "hwrld",
  "version": "1.0.0",
  "description": "Can log \"hello world\" and \"goodbye world\" to the console!",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "files": [
    "/dist"
  ]
}

```

如果你还没有一个包，你可以使用`npm init`来创建一个，它会指导你完成整个过程。记住，你选择的包名将被人们用于他们的imports，所以要明智地选择！

在 `package.json` 中还有一项非常重要的标志：您必须声明在哪里找到类型声明！ 这是通过 `"types": "dist/index.d.ts"` 来完成的，否则消费者将无法找到您的模块！

最后一个属性`files`可以帮助您包含想要发布到npm的文件。 这通常比使用 `.npmignore`文件更容易更安全。因为具体指定了这个库会发布的文件及目录列表

```shell
tsc
npm publish
```

现在您已经准备就绪！ 通过运行可以在任何地方消费您的库：

```shell
npm install --save hwrld
```

下面是如何调用

```TypeScript
import {sayHello} from 'hwrld'
sayHello();
```

对于后续发布，请使用semvar原则。当您对库进行补丁/错误修复时，可以运行`npm version patch`，对新功能运行`npm version minor`，对api的重大更改运行`npm version major`。
