---
title: 为什么在React中不需要信号
date: '2023-05-02'
tags: ['React', '博文翻译']
draft: false
summary: ''
---

原文地址：[Why You Don't Need Signals in React](https://blog.axlight.com/posts/why-you-dont-need-signals-in-react/)

# 为什么在 React 中不需要信号

## 介绍

在 Web 前端开发的世界里，信号已经成为一个热门话题。信号本质上用于表示随时间而变化的状态。一些开发者已经讨论了在 React 中使用信号的潜力。

信号实际上是一个比较旧的概念，现代的 Web 开发者对它的理解还有些不确定。起初，我也对信号的特性感到困惑，但后来我意识到，它们可以归纳为两个主要方面：

a）响应式原语
b）绕过差异化处理

https://twitter.com/dai_shi/status/1628926485200523264

在这篇博客文章中，我们将探讨这两个方面以及它们在 React 上下文中的相关性。请注意，我们只讨论信号的这两个方面，而不考虑其他可能对某人更重要的潜在用途。

反应性原语

反应性是 React 的一个关键特性。组件在状态改变时会重新渲染。使用 useState，可以创建反应性原语，通过定义在更新时触发重新渲染的状态。这使得您的组件变得反应性。此外，您可以在渲染函数中定义派生状态。

以下是使用 useState 的示例：

```tsx
const Component = () => {
  const [count, setCount] = useState(0)
  const doubleCount = count * 2 // 派生状态
  // ...
}
```

但是，值得注意的是 useState 只会创建本地状态。这可能会使在组件之间共享状态变得困难，您可能需要使用 prop drilling 或 context propagation 来实现它。

为了简化定义和使用全局状态的过程，第三方库例如 Jotai 可能会很有用。使用 Jotai，您可以轻松地在组件之间共享状态，而不依赖于 prop drilling 或 context propagation。

要使用 Jotai 定义全局状态，可以使用 atoms。这些 atoms 表示您可以在组件中使用的状态片段的定义。例如，以下是如何定义一个基本 atom：

```tsx
const countAtom = atom(0)
```

您还可以定义依赖于其他 atoms 的派生 atoms，如下所示：

```tsx
const doubleCountAtom = atom((get) => get(countAtom) * 2)
```

虽然 atoms 的语法可能看起来与典型的 signal 语法有所不同，但其思维模型非常相似。我们定义原语并将它们组合成复杂的状态。您可以定义尽可能多的 atoms 来表示数据图，从而轻松地管理应用程序中的复杂状态。

以下是如何在组件中使用 Jotai atoms：

```tsx
const Component = () => {
  const [count, setCount] = useAtom(countAtom)
  const [doubleCount] = useAtom(doubleCountAtom) // 派生状态
  // ...
}
```

与 useState 不同，useAtom 不是本地状态，您可以在另一个组件中使用它来共享 atom 状态：

```tsx
const AnotherComponent = () => {
  const [count, setCount] = useAtom(countAtom)
  // ...
}
```

您可能已经注意到了，在反应性原语方面，Jotai atom 的工作方式与 signal 非常相似。但是，Jotai 通过 React 钩子（如 useAtom）提供了额外的好处，这些钩子遵循 React 约定。这些钩子允许您在组件之间共享状态，无需使用 prop drilling 或 context propagation，简化您的代码并使其更易于理解。此外，Jotai 还有 Provider 用于隔离子树的状态，在真正的全局信号中不可能实现。

## Bypassing diffing

React 的另一个关键特点是通过一个称为“diffing”的过程来更新 DOM。通过比较 UI 的先前和当前表示，React 确定了哪些部分发生了变化，只更新了 DOM 的这些部分，从而提高了性能和 UI 的响应能力。

然而，diffing 是有代价的，有些情况下，绕过 diffing 可能更有效率。例如，如果您只更新 UI 的一部分，并且可以确定其他所有内容都没有发生变化，直接更新 UI 可能更有效率。

为了证明绕过 diffing 是技术上可行的，我们有一个实验性库称为 jotai-signal。我们还撰写了一篇博客，探讨了其中的内部：Demystifying Create React Signals Internals。

然而，绕过 diffing 违背了 React 声明式编程的核心原则。虽然出于性能原因绕过 diffing 可能很诱人，但这样做可能会引入 UI 的不一致性，并使您的应用程序更难理解。

在决定绕过 diffing 之前，有必要仔细评估性能收益，并将其与潜在风险进行权衡。还值得考虑是否有其他方法来优化应用程序的性能。

总的来说，建议遵循 React 的最佳实践，并适当使用 diffing，以确保 UI 始终保持一致和可预测。

## 总结

总之，在 web 开发中，信号是一个有趣的概念，但 Jotai 提供了一种更简单、更直观的方式来管理 React 应用程序中的状态。通过使用 atom，在 Jotai 中，您可以轻松创建和使用全局状态，消除了需要进行 prop drilling 或者 context propagation 的需求。Atom 在反应原语方面在概念上非常类似于信号。

虽然从技术上讲可能会绕过 React 的差异算法，但这样做违背了声明式编程的原则，并且可能会在您的 UI 中引入不一致性。在决定绕过差异前，请仔细评估潜在的利益和风险。

通过遵循 React 的最佳实践并利用 Jotai 的功能，您可以创建可维护和高性能的 React 应用程序。
