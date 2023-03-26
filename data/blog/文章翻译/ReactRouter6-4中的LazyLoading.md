---
title: ReactRouter6-4中的LazyLoading
date: '2023-03-26'
tags: ['react', '博文翻译']
draft: false
summary: ''
---

原文地址：[Lazy Loading Routes in React Router 6.4+](https://remix.run/blog/lazy-loading-routes)

# React Router 6.4+中的懒加载路由

[React Router 6.4](https://remix.run/blog/react-router-v6.4) 引入了[“数据路由器”](https://reactrouter.com/en/main/routers/picking-a-router)的概念，主要关注点是将数据获取与渲染分离，以消除渲染 + 获取链和随之而来的旋转加载图标。

这些链条通常被称为“瀑布”，但我们正在重新思考这个术语，因为大多数人听到“瀑布”就会想象尼亚加拉大瀑布，所有的水都会落在一个漂亮的大瀑布中。但是，“一次性”似乎是加载数据的好方法，那么为什么要讨厌瀑布呢？也许我们应该追求它们？

实际上，我们想要避免的“瀑布”看起来更像上面的标题图片，并且类似于楼梯。水流下来一点，然后停止，再下来一点，然后停止，依此类推。现在想象一下，在那个楼梯中每一个步骤都是一个加载旋转器。这不是我们想给用户的 UI 类型！因此，在本文（以及希望之外），我们使用“链”这个术语来表示固有顺序的获取，并且每个获取都被前面的获取所阻塞。

## Render + Fetch Chains

如果你还没有阅读过[“Remixing React Router”](https://remix.run/blog/remixing-react-router)一文或者去年 Reactathon 上 Ryan 的[“When to Fetch”](https://www.youtube.com/watch?v=95B8mnhzoCM)演讲，那么在深入阅读本文之前，建议先查看这些内容。它们涵盖了我们引入数据路由概念背后的很多背景知识。

简而言之，当你的路由器不知道你的数据需求时，你最终会得到链接请求，并且随着渲染子组件，“发现”了后续数据需求。

![将数据获取与组件耦合会导致渲染 + 获取链](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/MMGWa8.jpg)

但是引入数据路由器可以让您并行获取数据并一次性渲染所有内容：

![路由获取并行化请求，消除了缓慢的渲染 + 获取链](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/OYqYzx.jpg)

为了实现这一点，数据路由器将您的路由定义从渲染周期中提取出来，以便我们的路由器可以提前识别嵌套数据需求。

```tsx
// app.jsx
import Layout, { getUser } from `./layout`
import Home from `./home`
import Projects, { getProjects } from `./projects`
import Project, { getProject } from `./project`

const routes = [
  {
    path: '/',
    loader: () => getUser(),
    element: <Layout />,
    children: [
      {
        index: true,
        element: <Home />,
      },
      {
        path: 'projects',
        loader: () => getProjects(),
        element: <Projects />,
        children: [
          {
            path: ':projectId',
            loader: ({ params }) => getProject(params.projectId),
            element: <Project />,
          },
        ],
      },
    ],
  },
]
```

但这也有一个缺点。到目前为止，我们已经讨论了如何优化数据获取，但我们还必须考虑如何优化 JS 捆绑包的获取！通过上面的路由定义，虽然我们可以并行地获取所有数据，但我们阻塞了数据获取的开始，因为要下载包含所有加载器和组件的 Javascript 捆绑包。

考虑一个用户在 / 路由上进入您的网站：

![单个 JS 捆绑包阻止了数据获取](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/cJ4SjZ.jpg)

## React.lazy 能拯救我们吗？

[React.lazy ](https://beta.reactjs.org/reference/react/lazy)提供了一种优秀的原语来分块组件树，但它遭受着与数据路由器试图消除的获取和渲染紧密耦合的相同问题 😕。这是因为当你使用 React.lazy() 时，你会为你的组件创建一个异步块，但 React 直到渲染懒惰组件才开始获取该块。

```tsx
// app.jsx
const LazyComponent = React.lazy(() => import('./component'))

function App() {
  return (
    <React.Suspense fallback={<p>Loading lazy chunk...</p>}>
      <LazyComponent />
    </React.Suspense>
  )
}
```

![React.lazy() 调用会产生类似的渲染 + 获取链。](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/EwyafS.jpg)

因此，虽然我们可以在数据路由器中利用 React.lazy()，但最终会引入一个链来下载组件。Ruben Casas 撰写了[一篇很棒的文章](https://www.infoxicator.com/en/react-router-6-4-code-splitting)，介绍了一些使用 React.lazy() 在数据路由器中进行代码拆分的方法。但是从这篇文章中可以看出，手动进行代码拆分仍然有点冗长和繁琐。因为 DX 不够好，所以我们收到了[@rossipedia 提出的建议](https://github.com/remix-run/react-router/discussions/9826)（和初始 [POC 实现](https://github.com/remix-run/react-router/pull/9830)）。这个建议很好地概述了当前面临的挑战，并让我们开始思考如何在 RouterProvider 中引入一流的代码拆分支持。我们要向这两位（以及其他优秀社区成员）致以巨大的赞扬，感谢他们积极参与 React Router 演进 🙌。

## 介绍 Route.lazy

如果我们希望懒加载与数据路由器良好地配合，我们需要能够在渲染周期之外引入惰性。就像我们将数据获取从渲染周期中提取出来一样，我们也希望将路由获取提取出来。

如果你退后一步，看待路由定义，它可以分为三个部分：

- 路径匹配字段，例如路径、索引和子节点
- 数据加载/提交字段，如加载器和操作
- 渲染字段，如元素和错误元素

数据路由器在关键路径上真正需要的是匹配路径字段，因为它需要能够识别给定 URL 匹配的所有路由。匹配后，我们已经有了异步导航正在进行中，所以没有理由不能在该导航期间获取路由信息。然后，在完成数据获取之前，我们不需要渲染方面的内容，因为直到数据获取完成之前我们才会呈现目标路线。是的，这可能会引入“链”的概念（加载路线，然后加载数据），但这是一个可选的杠杆作用，在需要时可以拉动以解决初始加载速度和随后导航速度之间的权衡问题。

以下是使用上面的路由结构和在一个路由定义中使用新的 lazy() 方法（在 React Router v6.9.0 中可用）的示例：

```tsx
// app.jsx
import Layout, { getUser } from `./layout`;
import Home from `./home`;

const routes = [{
  path: '/',
  loader: () => getUser(),
  element: <Layout />,
  children: [{
    index: true,
    element: <Home />,
  }, {
    path: 'projects',
    lazy: () => import("./projects"), // 💤 Lazy load!
    children: [{
      path: ':projectId',
      lazy: () => import("./project"), // 💤 Lazy load!
    }],
  }],
}]

// projects.jsx
export function loader = () => { ... }; // formerly named getProjects

export function Component() { ... } // formerly named Projects

// project.jsx
export function loader = () => { ... }; // formerly named getProject

export function Component() { ... } // formerly named Project
```

你问的是导出功能组件吗？从这个惰性模块中导出的属性会逐字添加到路由定义中。因为导出一个元素很奇怪，所以我们增加了在路由对象上定义组件而不是元素的支持（但别担心，元素仍然可以使用！）。

在这种情况下，我们选择将布局和主页路由留在主要捆绑包中，因为这是我们用户最常用的入口点。但是，我们已经将项目导入和：projectId 路由的导入移动到它们自己的动态导入中，在没有导航到那些路由时不会被加载。

初始加载时，生成的网络图大致如下：

![lazy() 方法允许我们缩减关键路径捆绑包](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/EgKROK.jpg)

现在我们的关键路径捆绑包仅包括我们认为对于进入网站最为关键的那些路由。然后，当用户点击链接到 /projects/123 时，我们通过 lazy() 方法并行获取这些路由，并执行它们返回的加载器方法：

![我们在导航时并行地惰性加载路由](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/wcKLHR.jpg)

这让我们在某种程度上兼顾了两全其美的效果，因为我们能够将关键路径捆绑到相关的主页路由中。然后在导航时，我们可以匹配路径并获取所需的新路由定义。

## 高级用法和优化

一些敏锐的读者可能会感到一些 🕷️ 蜘蛛侠般的直觉，认为这里隐藏着一些链接。这是最优网络图吗？事实证明不是！但考虑到我们没有编写多少代码就得到了它，它还是相当不错的 😉。

在上面的例子中，我们的路由模块包括我们的加载器和组件，这意味着在开始加载程序之前，我们需要下载两者的内容。实际上，在 React Router SPA 中，您的加载程序通常非常小，并且会访问外部 API，其中大部分业务逻辑都存在。另一方面，组件定义了整个用户界面，包括所有与其相关联的用户交互 - 它们可能会变得相当大。

![单一路由文件会阻止组件下载后的数据获取](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/nLUSHE.jpg)

阻止加载程序（可能正在对某个 API 进行 fetch() 调用）通过 JS 下载大型组件树似乎很愚蠢。如果我们能把这个 👆 变成这个 👇 会怎样？

![我们可以通过将组件提取到它自己的文件中来解除数据获取的阻塞](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/K6H8JY.jpg)

好消息是，您只需进行最少量的代码更改即可实现！如果在路由上静态定义了一个加载器/操作，则它将与 lazy() 并行执行。这使我们可以通过将加载器和组件分开成不同的文件来解耦加载器数据获取和组件块下载：

```tsx
const routes = [
  {
    path: 'projects',
    async loader({ request, params }) {
      let { loader } = await import('./projects-loader')
      return loader({ request, params })
    },
    lazy: () => import('./projects-component'),
  },
]
```

在路由上静态定义的任何字段都将始终优先于从 lazy 返回的任何内容。因此，虽然您不应该同时定义静态加载程序并从 lazy 返回加载程序，但如果这样做，则会忽略懒惰版本，并获得控制台警告。

这个静态定义的加载器概念还为直接内联代码打开了一些有趣的可能性。例如，也许你有一个单独的 API 端点，它知道如何基于请求 URL 获取给定路由的数据。你可以以最小捆绑成本内联所有加载器，并在数据获取和组件（或路由模块）块下载之间实现完全并行化。

```tsx
const routes = [
  {
    path: 'projects',
    loader: ({ request }) => fetchDataForUrl(request.url),
    lazy: () => import('./projects-component'),
  },
]
```

![看啊，妈妈，没有加载器块！](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/1blpUr.jpg)
