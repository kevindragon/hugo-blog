---
title: "如何构建类似Vue的极简前端框架"
date: "2023-09-11T23:01:32+08:00"
tags: ["前端框架", "Vue"]
categories: [前端]
---

在开始之前，我们做一些前提设置，这个极简框架的目标是让我们避免去写一些常规的html和JavaScript代码，比如：

<!-- more -->

```html
<p id="cool-para"></p>
<script>
  const coolPara = 'Lorem ipsum.';
  const el = document.getElementById('cool-para');
  el.innerText = coolPara;
</script>
```

这个极简框架是让我们可以写一些神奇的HTML和JavaScript，就像Vue一样：

```html
<script setup>
  const coolPara = 'Lorem ipsum.';
</script>
<template>
  <p>{{ coolPara }}</p>
</template>
```

或者像React一样：

```jsx
export default function Para() {
  const coolPara = 'Lorem ipsum';
  return <p>{ coolPara }</p>;
}
```

这样的一个框架的好处是显而易见的，写代码的时候问题要记住多章节的内置变量和方法是困难的，如document、innerText、getElementById，而且也不利于软件的扩展。

我们的极简框架有两个特点：响应式和组合式。

> 响应式
>

在Vue和React代码中，数据和UI元素的显示内容有一个绑定，只要我们修改了数据，html元素的innerText值就会自动进行更新。这被称作响应式（Reactivity）

> 组合式
>

可以定义一个组件并重复使用它，而无需每次使用到它的时候还要再次定义它，这被称为可组合性。普通的HTML + JavaScript并不具备这个特性。因此，下面的代码并没有达到预期效果（仅仅是做了定义而，这有点类似Web Components的做法）：

```html
<!-- Defining the component -->
<component name="cool-para">
  <p>
    <content />
  </p>
</component>

<!-- Using the component -->
<cool-para>Lorem ipsum.</cool-para>
```

响应式和组合式是现代前端框架的两个主要特性，例如Vue, React等待。

要利用这些抽象概念（例如框架和库），我们需要先学习很多特定的概念。当这些东西以某种神奇的方式运作时，我们还要处理它们可能产生的问题。更不用说，我们还要处理一大堆容易出错的依赖关系。

但事实上，使用现代 Web API 来实现这两件事并不困难。而且，在大多数情况下，我们可能并不需要大而全的框架及其纷繁复杂的特性…

## 响应式（Reactivity）

一个句话解释响应式就是：当数据更新时，自动更新 UI。

首先，我们需要知道数据何时更新。不幸的是，这不是一个普通对象可以做到的。我们不能简单地附加一个名为 ondataupdate 的监听器来监听数据更新事件。

幸运的是，JavaScript 有一个可以实现这个功能的东西，它叫做代理（Proxy）。

## 代理对象（Proxy Objects）

`Proxy` 允许我们在常规对象上创建一个代理对象：

```jsx
const user = { name: 'Lin' };
const proxy = new Proxy(user, {});
```

这个代理对象可以被用来监听数据的变化。

在上面的例子当中，我们有了一个代理对象，但是当 `name` 属性发生变化的时候，不会发生任何额外的事情。

因此，我们需要一个 `handler` ，用来告诉代理对象，当数据发生变化的时候需要做些什么。

```jsx
// Handler that listens to data assignment operations
const handler = {
  set(user, value, property) {
    console.log(`${property} is being updated`);
    return Reflect.set(user, value, property);
  },
};

// Creating a proxy with the handler
const user = { name: 'Lin' };
const proxy = new Proxy(user, handler);
```

现在，任何时候修改 `name` 的值时，我们都会得到一个 `name is being updated.` 的消息。

你可能会想，这有什么的，我可以通过对象的 `set` 方法来实现这一点，但是我想跟你说：

- 代理方法更加通用，handlers方法可以被重用，这意味着……
- 你在对象上设置的任何值，都可以被递归的转换为代理，这意味着……
- 你将拥有一个神奇的对象，任何嵌套的数据更新，都是会被监听的。

除了这些，你还可以处理其他一些访问事件，比如当一个属性被读取、更新、删除等。现在我们已经有能力监听操作，我们需要实际响应这些事件了。

## 更新UI

如果你还记得，响应式的第二部分是自动更新UI。为些，我们需要知道被更新的UI元素。在这之前，我们需要首先标记这个UI元素，当数据更新的时候可以找到这个UI元素。为了做到这一点，我们将使用数据属性，这是一个允许我们在元素上设置任意值的特性：

```html
<div>
  <!-- Mark the h1 as appropriate for when "name" changes -->
  <h1 data-mark="name"></h1>
</div>
```

数据属性的便利之处在于，我们可以使用以下方法找到所有适当的元素：

```jsx
document.querySelectorAll('[data-mark="name"]');
```

现在我们只需设置所有适当元素的 innerText：

```jsx
const handler = {
  set(user, value, property) {
    const query = `[data-mark="${property}"]`;
    const elements = document.querySelectorAll(query);

    for (const el of elements) {
      el.innerText = value;
    }

    return Reflect.set(user, value, property);
  },
};

// Regular object is omitted cause it's not needed.
const user = new Proxy({ name: 'Lin' }, handler);
```

就这些，这就是响应式的核心！

由于我们的处理程序具有通用性，用户设置的任何属性都会更新所有适当的 UI 元素。

这就是 JavaScript 代理功能如此强大的原因，不需要任何依赖，只需一些巧妙的方法，它就可以为我们提供这些神奇的反应性对象。

现在我们来说第二个主要内容…

## 组合式（Composibility）

结果发现，浏览器已经为这个功能专门推出了一个叫做 Web Components 的功能，谁能想到！

很少有人使用它是因为它有点让人头疼（而且也因为大多数人在开始一个项目时，默认选择常用的框架，而不考虑项目的范围）。

为了实现组件的可组合性，我们首先需要定义组件。

### 使用template和slot定义组件

`<template>` 标签用于包含浏览器不渲染的标记。例如，您可以在 HTML 中添加以下标记：

```html
<template>
  <h1>Will not render!</h1>
</template>
```

这个组件暂时不会渲染。你可以认为这是一个不可见的组件容器。

下一个构建块是 `<slot>` 元素，它定义了组件内容的放置位置。这使得组件可以重复使用不同的内容，也就是说，它变得可组合。

举个例子，这是一个红色内容的`h1`元素：

```html
<template>
  <h1 style="color: red">
    <slot />
  </h1>
</template>
```

在我们使用组件之前 - 比如上面的红色 h1，我们需要先注册它们。

### 注册组件

在我们注册红色 h1 组件之前，我们需要一个名称来注册它。我们可以使用 name 属性来实现这一点：

```html
<template name="red-h1">
  <h1 style="color: red">
    <slot />
  </h1>
</template>
```

现在，使用一些JavaScript代码来获取组件及它的名称：

```jsx
const template = document.getElementsByTagName('template')[0];
const componentName = template.getAttribute('name');
```

然后使用 `[customElements.define](https://developer.mozilla.org/en-US/docs/Web/API/CustomElementRegistry/define)` 来注册：

```jsx
customElements.define(
  componentName,
  class extends HTMLElement {
    constructor() {
      super();
      const component = template.content.children[0].cloneNode(true);
      this.attachShadow({ mode: 'open' }).appendChild(component);
    }
  }
);
```

在上面的代码块中发生了很多事情：

- 我们正在调用 customElements.define 函数并传入两个参数。
- 第一个参数是组件名称（即 "red-h1"）。
- 第二个参数是一个定义我们自定义组件为 HTMLElement 的类。

在类构造函数中，我们使用 red-h1 模板的副本来设置 shadow DOM 树。

<aside>
💡 什么是 Shadow Dom？
Shadow DOM 用于设置一些默认元素（如范围输入或视频元素）的样式。
元素的Shadow DOM 默认情况下是隐藏的，这就是为什么我们在开发者控制台中看不到它，但是我们这里将其设置为 'open' 模式。
这使我们可以检查元素，并看到红色 h1 与 #shadow-root 相关联。

</aside>

调用 `customElements.define` 将使我们能够像使用普通 HTML 元素一样使用定义好的组件。

```html
<red-h1>This will render in red!</red-h1>
```

接下来，我们将把这两个概念结合起来！

## 响应式 + 组合式

快速回顾一下，我们做了两件事：

1. 我们创建了一个响应式数据结构，即代理对象，当设置一个值时，可以更新我们标记为合适的任何元素。
2. 我们定义了一个自定义组件 red-h1，它将渲染其内容为一个红色 h1。

现在我们可以把它们放在一起：

```html
<div>
  <red-h1 data-mark="name"></red-h1>
</div>

<script>
  const user = new Proxy({}, handler);
  user.name = 'Lin';
</script>
```

让自定义组件渲染我们的数据，并在我们更改数据时更新界面。

## 总结

当然，常规的前端框架不只做到这一步，它们有专门的语法，如 Vue 中的模板语法和 React 中的 JSX，这些语法使得编写复杂的前端页面相对于其他方式更加简洁。

由于这种专门的语法不是常规的 JavaScript 或 HTML，浏览器无法解析它们，因此它们都需要专门的工具将它们编译成常规的 JavaScript、HTML 和 CSS，以便浏览器能够理解。因此，现在没有人再写 JavaScript 了。

即使没有专门的语法，你仍然可以通过使用`Proxy`和 `Web Components` 来实现常规前端框架的大部分功能，并且具有类似的简洁性。
