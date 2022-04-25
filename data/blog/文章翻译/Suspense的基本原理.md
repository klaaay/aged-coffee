---
title: Suspense的基本原理
date: '2022-03-21'
tags: ['react', '博文翻译']
draft: false
summary: ''
---

原文地址：[A Fundamental Guide To React Suspense](https://www.chakshunyu.com/blog/a-fundamental-guide-to-react-suspense/)

另一个将在 React 18 中发布的大功能是 Suspense。如果你在 React 开发领域呆的时间比较长，那么你会知道 Suspense 功能并不是特别新。早在 2018 年，Suspense 是作为 React 16.6 版本的一部分，作为一个实验性功能发布的。然后，它主要是针对处理与 React.lazy 相结合的代码分割。

但现在，随着 React 18 的发布，Suspense 的正式发布摆在了我们面前。与[并发渲染](https://www.chakshunyu.com/blog/an-introductory-guide-to-concurrent-rendering/)的发布一起，Suspense 的真正力量终于被释放出来。Suspense 和并发渲染之间的互动为改善用户体验开辟了一个巨大的机会世界。

但是，就像所有的功能一样，就像并发的渲染一样，从基本原理开始是很重要的。到底什么是 Suspense？为什么我们一开始就需要 Suspense？Suspense 是如何解决这个问题的？有什么好处？为了帮助你理解这些基本原理，本文将确切地讨论这些问题，并为你提供有关 Suspense 主题的坚实知识基础。

## 什么是 Suspense

从本质上讲，Suspense 是一种机制，让 React 开发者向 React 表明，一个组件正在等待数据准备就绪。然后 React 知道它应该等待该数据被取走。同时，将向用户显示一个回退，React 将继续渲染应用程序的其余部分。在数据准备好后，React 将回到那个特定的用户界面，并相应地更新它。

从根本上说，这听起来与目前 React 开发者必须实现数据获取流程的方式没有太大区别：使用某种状态来指示组件是否仍在等待数据，一个开始获取数据的 useEffect，根据数据的状态显示一个加载状态，并在数据准备好后更新 UI。

但在实践中，Suspense 在技术上使这种情况发生，是完全不同的。与提到的数据获取流程相反，Suspense 与 React 深度整合，允许开发者更直观地协调加载状态，并避免了竞赛条件。为了更好地理解这些细节，重要的是要知道我们为什么需要 Suspense。

## 我们为什么需要 Suspense

在没有 Suspense 的情况下，有两种主要的方法来实现数据获取流：render 时获取和 fetch-then-render。然而，这些传统的数据获取流程存在一些问题。为了理解 Suspense，我们必须深入研究这些流程的问题和限制。

### Fetch-on-render

大多数人都会像之前提到的那样，使用 useEffect 和状态变量来实现数据的获取流程。这意味着只有当一个组件渲染时才开始获取数据。所有的数据获取都发生在组件的效果和生命周期方法中。

这种方法的主要问题是，组件只在渲染时触发数据获取，其异步性质迫使组件必须等待其他组件的数据请求。

假设我们有一个组件 ComponentA，它获取了一些数据并有一个加载状态。在内部，ComponentA 还渲染了另一个组件 ComponentB，它自己也执行了一些数据获取。但是由于数据获取的实现方式，ComponentB 只有在被渲染后才开始获取数据。这意味着它必须等待，直到 ComponentA 完成数据的获取，然后再渲染 ComponentB。

这导致了一种瀑布式的方法，即组件之间的数据获取是按顺序进行的，这基本上意味着它们是相互阻塞的。

```tsx
function ComponentA() {
  const [data, setData] = useState(null)

  useEffect(() => {
    fetchAwesomeData().then((data) => setData(data))
  }, [])

  if (user === null) {
    return <p>Loading data...</p>
  }

  return (
    <>
      <h1>{data.title}</h1>
      <ComponentB />
    </>
  )
}

function ComponentB() {
  const [data, setData] = useState(null)

  useEffect(() => {
    fetchGreatData().then((data) => setData(data))
  }, [])

  return data === null ? <h2>Loading data...</h2> : <SomeComponent data={data} />
}
```

### Fetch-then-render

为了防止组件之间的数据获取的顺序阻塞，一个替代方案是尽可能早地开始所有的数据获取工作。因此，与其让组件负责处理渲染时的数据获取和数据请求分别发生，不如在树开始渲染之前启动所有的请求。

这种方法的好处是，所有的数据请求都是一起发起的，因此组件 B 不必等待组件 A 完成。这就解决了组件依次阻断对方数据流的问题。然而，这也带来了另一个问题，那就是我们必须等待所有的数据请求完成后才能为用户呈现任何东西。可以想象，这不是一个最佳的体验。

```tsx
// 在渲染整个树之前开始获取数据
function fetchAllData() {
  return Promise.all([fetchAwesomeData(), fetchGreatData()]).then(([awesomeData, greatData]) => ({
    awesomeData,
    greatData,
  }))
}

const promise = fetchAllData()

function ComponentA() {
  const [awesomeData, setAwesomeData] = useState(null)
  const [greatData, setGreatData] = useState(null)

  useEffect(() => {
    promise.then(({ awesomeData, greatData }) => {
      setAwesomeData(awesomeData)
      setGreatData(greatData)
    })
  }, [])

  if (user === null) {
    return <p>Loading data...</p>
  }

  return (
    <>
      <h1>{data.title}</h1>
      <ComponentB />
    </>
  )
}

function ComponentB({ data }) {
  return data === null ? <h2>Loading data...</h2> : <SomeComponent data={data} />
}
```

## Suspense 是怎么解决这个问题的

从本质上讲，fetch-on-render 和 fetch-then-render 的主要问题归结为一个事实，即我们试图强行同步两个不同的流程，即数据获取流程和 React 生命周期。通过 Suspense，我们得到了一种不同的数据获取方法，即所谓的边获取边渲染的方法。

```tsx
const specialSuspenseResource = fetchAllDataSuspense()

function App() {
  return (
    <Suspense fallback={<h1>Loading data...</h1>}>
      <ComponentA />
      <Suspense fallback={<h2>Loading data...</h2>}>
        <ComponentB />
      </Suspense>
    </Suspense>
  )
}

function ComponentA() {
  const data = specialSuspenseResource.awesomeData.read()
  return <h1>{data.title}</h1>
}

function ComponentB() {
  const data = specialSuspenseResource.greatData.read()
  return <SomeComponent data={data} />
}
```

与之前的实现不同的是，它允许组件在 React 到达的那一刻启动数据获取。这甚至发生在组件渲染之前，而且 React 并没有就此停止。然后，它继续评估组件的子树，并继续尝试渲染它，同时等待数据获取的完成。

这意味着 Suspense 不会阻塞渲染，这意味着子组件不必等待父组件完成后再启动它们的数据获取请求。React 试图尽可能地渲染，同时启动适当的数据获取请求。在一个请求完成后，React 将重新访问相应的组件，并使用新收到的数据相应地更新用户界面。

## Suspense 的好处是什么

Suspense 有很多好处，特别是在用户体验方面。但其中一些好处也涵盖了开发者的体验。

- 尽早启动获取。Suspense 引入的 render-as-you-fetch 方法的最大和最直接的好处是，数据获取被尽可能早地启动。这意味着用户需要等待的时间更短，应用程序的速度更快，这对任何前端应用程序来说都是普遍有益的。

- 更直观的加载状态。有了 Suspense，组件不必再包括一大堆混乱的 if 语句，也不必再单独跟踪状态来实现加载状态。相反，加载状态被集成到它所属的组件本身。这使得组件更加直观，因为它将加载代码与相关代码保持在一起，并且由于加载状态被包含在组件中，因此更容易重复使用。

- 避免了竞赛条件。现有的数据获取实现的一个问题，我在这篇文章中没有深入介绍，就是竞赛条件。在某些情况下，传统的 "渲染时获取 "和 "渲染时获取 "的实现方式可能会导致竞赛条件，这取决于不同的因素，如时间、用户输入和参数化数据请求。主要的潜在问题是，我们试图强行同步两个不同的进程，即 React 的和数据获取的进程。但有了 Suspense，这就可以更优雅地进行整合，从而避免了上述问题。

- 更加综合的错误处理。使用 Suspense，我们基本上已经为数据请求流创建了边界。除此之外，由于 Suspense 使其与组件的代码整合得更直观，它允许 React 开发人员也为 React 代码和数据请求实现更多的集成错误处理。

## 最后总结

React Suspense 已经被关注了 3 年多了。但随着 React 18 的发布，正式发布的时间越来越近了。继并发渲染之后，它将是作为这个 React 版本的一部分而发布的最大功能之一。就其本身而言，它可以将数据获取和加载状态的实现提升到一个新的直观性和优雅的水平。

为了帮助你了解 Suspense 的基本原理，这篇文章涵盖了对它很重要的几个问题和方面。这涉及到了什么是 Suspense，为什么我们首先需要 Suspense 这样的东西，它是如何解决某些数据获取问题的，以及 Suspense 带来的所有好处。
