---
title: html标签和body标签在css的区别
date: '2022-11-20'
tags: ['html', '博文翻译']
draft: false
summary: ''
---

原文地址：[HTML vs Body in CSS](https://css-tricks.com/html-vs-body-in-css/)

`<html>`和`<body>`之间的区别很容易被忽略。这似乎是一件小事。必须承认，我有一个坏习惯，把所有全局样式都应用到`<body>`，当效果不理想时，我就会不加思考地移到`<html>`。

然而，还是有区别的，无论我们是 CSS 老手还是新手，都应该意识到它们。我们将在这篇文章中讨论这些，并考虑一些实际的应用，他们的差别可能是有意义的。

## HTML 和 body 是如何关联的

考虑一下基本 HTML 文档的标准结构

```html
<!DOCTYPE html>
<html lang="en">

  <head>
    <!-- Metadata and such -->
  </head>

  <body>
    <!-- Where the content begins -->
  <body>

</html>
```

规范将`<html>`定义为文档的根元素，在上面的例子中我们可以清楚地看到：`<html>`元素是所有其他元素的最顶层。责任就止于此，因为除了可以继承样式的级别之外，再没有别的级别了。

从那里开始，`<head>`和`<body>`构成了直接落在`<html>`中的两个元素。事实上，规范直接将`<body>`与`<head>`定义为对照，因为这是惟一需要区分的两个元素。

因此，这里的底线是`<html>`是文档的根元素，而`<body>`是包含在其中的子元素。事实上，在 CSS 中有一个:root 选择器。它们的目标完全相同

```css
:root {
}
html {
}
```

```
注意: 根具有更高的特异性:(0,0,1,0)vs(0,0,0,1)。
```

## 我们应该总是把全局样式放在`<html>`上？

很容易想到我们想要在整个台上继承的任何样式都应直接应用于`<html>`，因为它是文档的根元素。`<html>` 包含了 `<body>`的职责范围。

但事实并非如此。事实上，以下代码的内联属性最初在规范中被分配给`<body>`

- background
- bgcolor
- marginbottom
- marginleft
- marginright
- margintop
- text

虽然这些属性现在被认为是过时的，但建议使用 CSS 来代替它们对应的 CSS 属性：
| Inline Attribute | CSS Property |
| ---------------- | ----------------------------- |
| background | background |
| bgcolor | background / background-color |
| marginbottom | margin-bottom |
| margintop | margin-top |
| marginleft | margin-left |
| marginright | margin-right |
| text | font |

考虑到这些 CSS 属性来源于为`<body>`编写的内联属性，它们也应该直接应用于 CSS 中的`<body>`，而不是将它们推入`<html>`元素中，这似乎是合适的。

## 我们应该总是把全局样式放在`<body>`上？

嗯，不完全是。可能在某些情况下，将样式应用于`<html>`更有意义。让我们考虑其中的几个场景。

### 使用 rem 单位

rem 单位相对于文档根。例如，当写这样的东西时

```css
.module {
  width: 40rem;
}
```

rem 单位相对于`<html>`元素的字体大小，该元素是文档根。因此，在根级别设置为用户默认值的是 `.module` 类将伸缩到的值。

Jonathan Snook 有一篇经典的文章，很好地说明了如何设置`<html>`的字体大小为百分比，当使用 rem 单位时，可以将其用作重置。

### 古怪的背景颜色

在 CSS 中有一个奇怪的事情，`<body>`上的背景色淹没了整个视口，即使元素本身的度量并没有覆盖整个区域。除非在 html 元素上设置了背景色，否则不会。

如果不想要整个视口是目标，那么将其设置在 HTML 元素上可能是很明智的。

## 总结

希望这能让大家了解在 CSS 中`<html>`和`<body>`使用之间的关键区别。JavaScript 也有区别。例如，你不需要查询，html 是文档。rootElement 和 body 是 document.body。

我们当然可以在这两者之间画出更多的技术区别，但这里的重点是提高我们的理解，以便在编写 CSS 时做出更好的决策。

你有其他的例子，它更有意义的样式一个比另一个？或者也许你有一套不同的标准，知道什么时候设计一个？请在评论中告诉我们。
