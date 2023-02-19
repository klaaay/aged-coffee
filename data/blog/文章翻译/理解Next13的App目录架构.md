---
title: 理解Next13的App目录架构
date: '2023-02-20'
tags: ['Next', '博文翻译']
draft: false
summary: ''
---

原文地址：[Understanding App Directory Architecture In Next.js](https://www.smashingmagazine.com/2023/02/understanding-app-directory-architecture-next-js/)

- [理解 Next13 的 App 目录架构](#理解-next13-的-app-目录架构)
  - [什么是 App 目录？](#什么是-app-目录)
    - [快速比较](#快速比较)
  - [App 目录的构建模块](#app-目录的构建模块)
    - [SERVER COMPONENTS 简介](#server-components-简介)
  - [扩展的 Fetch API](#扩展的-fetch-api)
  - [迁移从 Pages 到 App](#迁移从-pages-到-app)
    - [React 上下文的情况](#react-上下文的情况)
    - [Typescript 和异步 React 元素](#typescript-和异步-react-元素)
    - [客户端获取数据的工作](#客户端获取数据的工作)
  - [那么，它准备好了吗？](#那么它准备好了吗)

# 理解 Next13 的 App 目录架构

概要：新的 App 目录架构是最近 Next.js 发布的主要内容，它引发了很多讨论。在本文中，Atila Fassina 探讨了这种新策略的优点和缺陷，并思考了您现在是否应该在生产中使用它。

自 [Next.js 13](https://nextjs.org/blog/next-13) 发布以来，就有一些关于其公告中包含的新功能的稳定性的争议。在[“Next.js 13 有哪些新功能？”](https://www.smashingmagazine.com/2022/11/whats-new-nextjs-13/)这篇文章中，我们介绍了发布的内容，并确定了 Next.js 13 虽然包含一些有趣的实验，但绝对是稳定的。从那时起，对于新的 \<Link> 和 \<Image> 组件，甚至还有（仍处于 beta 阶段的）@next/font，我们大多数人都看到了非常清晰的情况；这些都是可以直接使用的，具有即时收益。如公告中明确说明的那样，Turbopack 仍处于 alpha 阶段，仅针对开发构建，并且仍在积极开发中。您是否可以在日常工作中使用它取决于您的技术栈，因为仍有一些集成和优化正在进行中。本文的范围仅限于公告的主角：新的应用目录架构（简称 AppDir）。

由于应用目录与 React 生态系统中的一个重要演进 - React 服务器组件 - 以及边缘运行时配合使用，因此它一直引起了问题。它显然是我们 Next.js 应用程序未来的形态。虽然它是实验性的，但其路线图不是我们可以认为会在接下来的几周内完成的。因此，您现在是否应该在生产中使用它？您可以从中获得什么优势，以及您可能会遇到的陷阱是什么？像往常一样，软件开发中的答案都是相同的：这取决于不同情况。

## 什么是 App 目录？

这是在 Next.js 中处理路由和渲染视图的新策略。它是由几个不同的功能组合在一起实现的，并且它是为了最大程度地利用 React 并发特性而构建的（是的，我们正在谈论 React Suspense）。它带来了一个大的范式转变，改变了你在 Next.js 应用程序中思考组件和页面的方式。这种构建应用程序的新方式对架构有很多非常受欢迎的改进。以下是一个简短的、非详尽的列表：

- 部分路由。
  - 路由分组。
  - 并行路由。
  - 拦截路由。
- 服务器组件 vs 客户端组件。
- Suspense 边界。
- 还有更多，请查看新文档中的功能概述。

### 快速比较

当谈到当前路由和渲染架构（在 Pages 目录中）时，开发者需要考虑每个路由的数据获取方式。

- getServerSideProps: 服务端渲染；
- getStaticProps: 服务端预渲染和/或增量静态再生；
- getStaticPaths + getStaticProps: 服务端预渲染或静态站点生成。

历史上，还没有办法在每个页面上选择渲染策略。大多数应用程序要么全面采用服务端渲染，要么全面采用静态站点生成。Next.js 创建了足够的抽象层，使得在其架构内单独考虑路由成为标准。

一旦应用程序到达浏览器，就会启动 hydration 过程，并且通过将\_app 组件包装在 React Context Provider 中，可以使路由共享数据。这给了我们工具来将数据置于渲染树的顶部并向下级传递数据。

```tsx
import { type AppProps } from 'next/app';

export default function MyApp({ Component, pageProps }: AppProps) {
  return (
        <SomeProvider>
            <Component {...pageProps} />
        </SomeProvider>
}
```

能够在每个路由中渲染和组织所需数据，使得这种方法成为必要时绝佳的工具，以便在整个应用中数据得以全局传播。然而，将所有内容包装在 Context Provider 中会将 hydration 捆绑到应用程序的根部，不再能够在该树上渲染任何分支（在该 Provider 上下文中的任何路由）。

这里就是 Layout 模式的作用了。通过在页面周围创建包装器，我们可以选择针对每个路由的渲染策略，而不是一次性做出应用级决策。有关如何在 Pages Directory 中管理状态，请参阅[“Next.js 中的状态管理”](https://www.smashingmagazine.com/2021/08/state-management-nextjs/)文章以及 [Next.js 文档](https://nextjs.org/docs/basic-features/layouts)。

Layout 模式被证明是一个很好的解决方案。能够精细定义渲染策略是一个非常受欢迎的功能。因此，App 目录突出了 Layout 模式的重要性。作为 Next.js 架构的一等公民，它在性能、安全性和数据处理方面提供了巨大的改进。

通过 React concurrent 特性，现在可以将组件流式传输到浏览器，并让每个组件处理其自己的数据。因此，渲染策略现在更加精细，不再是基于页面而是基于组件。默认情况下，Layout 是嵌套的，这使得开发人员更加清楚地了解基于文件系统架构每个页面的影响。最重要的是，在使用 Context 之前必须显式地将组件转换为客户端组件（通过 "use client" 指令）。

## App 目录的构建模块

这种架构是围绕着每个页面一个布局的架构建立的。现在没有了\_app 和\_document 组件。它们都被根 layout.jsx 组件所取代。正如你所期望的那样，这是一个特殊的布局，将整个应用程序包装起来。

```tsx
export function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  )
}
```

根布局是我们一次性操作返回给整个应用程序的 HTML 的方式。它是一个服务器组件，在导航时不会重新渲染。这意味着布局中的任何数据或状态都将在应用程序的生命周期内持续存在。

虽然根布局是我们整个应用程序的一个特殊组件，但我们还可以为其他构建块定义根组件：

- loading.jsx: 定义整个路由的 Suspense 边界;
- error.jsx: 定义整个路由的错误边界;
- template.jsx: 类似于布局，但在每次导航时重新渲染。特别适用于处理路由之间的状态，例如进入或退出过渡效果。

所有这些组件和约定默认都是嵌套的。这意味着/about 将自动嵌套在/的包装器中。

最后，我们还需要为每个路由定义一个 page.jsx，它将定义与该 URL 段（即你放置组件的位置）对应的主要组件。这些显然不是默认嵌套的，并且只有在与它们相对应的 URL 段存在精确匹配时才会显示在我们的 DOM 中。

当然，还有更多的架构（甚至还有更多即将到来！），但这应该足以在考虑将生产中的 Pages 目录迁移到 App 目录之前正确理解你的思维模型。务必查看官方[升级指南](https://beta.nextjs.org/docs/upgrade-guide)。

### SERVER COMPONENTS 简介

React 服务器组件允许应用程序利用基础设施来提高性能和整体用户体验。例如，立即的改进在于捆绑大小方面，因为 RSC 不会将它们的依赖项带入最终捆绑包中。因为它们在服务器上呈现，任何类型的解析、格式化或组件库都将保留在服务器代码上。其次，由于它们的异步性质，服务器组件会流式传输到客户端。这允许在浏览器上逐步增强呈现的 HTML。

因此，服务器组件导致最终捆绑包的更可预测、可缓存和恒定大小，打破了应用程序大小和捆绑包大小之间的线性相关性。这立即将 RSC 作为最佳实践与传统的 React 组件（现在称为客户端组件以便于消歧义）进行比较。

在服务器组件中，获取数据也非常灵活，而且在我看来，更接近原生 JavaScript-这总是使学习曲线更平滑。例如，了解 JavaScript 运行时使得定义数据获取为并行或顺序成为可能，因此可以更细粒度地控制资源加载瀑布。

- **并行数据获取**，等待所有：

```tsx
import TodoList from './todo-list'

async function getUser(userId) {
  const res = await fetch(`https://<some-api>/user/${userId}`)
  return res.json()
}

async function getTodos(userId) {
  const res = await fetch(`https://<some-api>/todos/${userId}/list`)
  return res.json()
}

export default async function Page({ params: { userId } }) {
  // Initiate both requests in parallel.
  const userResponse = getUser(userId)
  const todosResponse = getTodos(username)

  // Wait for the promises to resolve.
  const [user, todos] = await Promise.all([userResponse, todosResponse])

  return (
    <>
      <h1>{user.name}</h1>
      <TodoList list={todos}></TodoList>
    </>
  )
}
```

- **并行**，等待一个请求，流式传输另一个请求：

```tsx
async function getUser(userId) {
  const res = await fetch(`https://<some-api>/user/${userId}`)
  return res.json()
}

async function getTodos(userId) {
  const res = await fetch(`https://<some-api>/todos/${userId}/list`)
  return res.json()
}

export default async function Page({ params: { userId } }) {
  // Initiate both requests in parallel.
  const userResponse = getUser(userId)
  const todosResponse = getTodos(userId)

  // Wait only for the user.
  const user = await userResponse

  return (
    <>
      <h1>{user.name}</h1>
      <Suspense fallback={<div>Fetching todos...</div>}>
        <TodoList listPromise={todosResponse}></TodoList>
      </Suspense>
    </>
  )
}

async function TodoList({ listPromise }) {
  // Wait for the album's promise to resolve.
  const todos = await listPromise

  return (
    <ul>
      {todos.map(({ id, name }) => (
        <li key={id}>{name}</li>
      ))}
    </ul>
  )
}
```

在这种情况下，\<TodoList> 接收到一个正在进行中的 Promise，需要在渲染之前等待它。应用程序将呈现 suspense 回退组件，直到所有操作都完成。

- **顺序数据获取**一次只会触发一个请求并等待每个请求：

```tsx
async function getUser(username) {
  const res = await fetch(`https://<some-api>/user/${userId}`)
  return res.json()
}

async function getTodos(username) {
  const res = await fetch(`https://<some-api>/todos/${userId}/list`)
  return res.json()
}

export default async function Page({ params: { userId } }) {
  const user = await getUser(userId)

  return (
    <>
      <h1>{user.name}</h1>
      <Suspense fallback={<div>Fetching todos...</div>}>
        <TodoList userId={userId} />
      </Suspense>
    </>
  )
}

async function TodoList({ userId }) {
  const todos = await getTodos(userId)

  return (
    <ul>
      {todos.map(({ id, name }) => (
        <li key={id}>{name}</li>
      ))}
    </ul>
  )
}
```

现在，Page 将会在获取 getUser 数据并等待它之后开始渲染。一旦到达\<TodoList>，它将获取 getTodos 数据并等待它。这仍然比我们在 Pages 目录中使用的方法更加细化。

需要注意的重要事项：

- 在同一组件范围内发出的请求将并行发出（更多关于此的信息请参见下面的扩展获取 API）。

- 在同一服务器运行时发出的相同请求将被去重（只有一个实际发生，缓存过期时间最短的那个）。

- 对于不使用 fetch 的请求（例如第三方库，如 SDK、ORM 或数据库客户端），除非通过段缓存配置手动配置，否则路由缓存不会受到影响。

```tsx
export const revalidate = 600; // revalidate every 10 minutes

export default function Contributors({
  params
}: {
  params: { projectId: string };
}) {
    const { projectId }  = params
    const { contributors } = await myORM.db.workspace.project({ id: projectId })

  return <ul>{*/ ... */}</ul>;
}
```

这说明开发人员可以更加精细地控制页面的渲染。在 pages 目录下，直到所有数据都准备好后，页面渲染才会被解除阻塞。而使用 getServerSideProps 时，用户仍然会看到加载动画，直到整个路由的数据都准备好。为了在 App 目录下模拟这种行为，需要在该路由的 layout.tsx 中发出 fetch 请求，因此应该避免这样做。一种 "全有或全无" 的方法很少是你所需要的，相对于这种细粒度的策略，它会导致更差的感知性能。

## 扩展的 Fetch API

语法仍然保持不变：fetch(route, options)。但是根据 [Web Fetch Spec](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)，options.cache 将决定该 API 如何与浏览器缓存进行交互。但在 Next.js 中，它将与[框架的服务器端 HTTP 缓存](https://beta.nextjs.org/docs/data-fetching/fundamentals#caching-data)进行交互。

在谈论 Next.js 的扩展 Fetch API 和其缓存策略时，有两个值需要理解：

- force-cache: 默认选项，查找最新匹配并返回它。
- no-store 或 no-cache: 每次请求都从远程服务器获取。
- next.revalidate: 与 ISR 相同的语法，设置一个硬阈值来考虑资源是否新鲜。

```tsx
fetch(`https://route`, { cache: 'force-cache', next: { revalidate: 60 } })
```

缓存策略允许我们对我们的请求进行分类：

- 静态数据：保留时间更长。例如，博客文章。
- 动态数据：经常更改和/或是用户交互的结果。例如，评论部分，购物车。

默认情况下，每个数据都被视为静态数据。这是由于 force-cache 是默认的缓存策略。要完全排除动态数据的影响，可以定义 no-store 或 no-cache。

如果使用动态函数（例如设置 cookies 或 headers），默认值将从 force-cache 切换到 no-store！

最后，要实现与增量静态再生更相似的内容，您需要使用 next.revalidate。好处是它只定义了它所在的组件，而不是整个路由。

## 迁移从 Pages 到 App

将逻辑从 Pages 目录迁移到 App 目录可能看起来需要做很多工作，但是 Next.js 已经准备好让这两种架构共存，因此迁移可以逐步进行。此外，官方文档中有一个非常好的[迁移指南](https://beta.nextjs.org/docs/upgrade-guide#migrating-from-pages-to-app)；在开始重构之前，我建议您完全阅读它。

引导您完成迁移路径超出了本文的范围，并且会使其与文档重复。相反，为了在官方文档提供的基础上增加价值，我将尝试提供一些经验，以帮助您避免摩擦点。

### React 上下文的情况

为了提供本文中提到的所有优点，RSC 不能是交互式的，这意味着它们没有 hooks。因此，我们决定尽可能推迟将客户端逻辑推向渲染树的叶子节点；一旦添加交互性，该组件的子代将是客户端端的。

在一些情况下，推迟推送某些组件将不可能（特别是如果某些关键功能依赖于 React 上下文，例如）。由于大多数库都准备好保护其用户免受 Prop Drilling 的影响，因此许多库创建上下文提供程序，以跳过从根到远处的后代组件。因此，完全放弃 React 上下文可能会导致一些外部库无法正常工作。

作为临时解决方案，有一种针对此问题的客户端封装程序：

```tsx
// /providers.jsx
‘use client’

import { type ReactNode, createContext } from 'react';

const SomeContext = createContext();

export default function ThemeProvider({ children }: { children: ReactNode }) {
  return (
    <SomeContext.Provider value="data">
      {children}
    </SomeContext.Provider>
  );
}

```

因此，布局组件不会抱怨跳过一个客户端组件的渲染。

```tsx
// app/.../layout.jsx
import { type ReactNode } from 'react';
import Providers from ‘./providers’;

export default function Layout({ children }: { children: ReactNode }) {
    return (
    <Providers>{children}</Providers>
  );
}
```

重要的是要意识到，一旦这样做，整个分支将变成客户端渲染。该方法将使 \<Providers> 组件中的所有内容不会在服务器端呈现，因此仅在万不得已的情况下使用。

### Typescript 和异步 React 元素

在 Layouts 和 Pages 之外使用 async/await 时，TypeScript 会基于其 JSX 定义所期望匹配的响应类型产生错误。尽管在运行时它仍然可用并且支持，但根据 Next.js 文档，这需要在 TypeScript 上游进行修复。

目前的解决方案是在上面一行添加一个注释`{/* @ts-expect-error Server Component */}`。

### 客户端获取数据的工作

历史上，Next.js 没有内置的数据变异（mutation）功能。从客户端发出的请求是开发人员自己决定如何处理。使用 React Server Components 后，这种情况就有所改变了。React 团队正在开发一个 use hook，该 hook 将接受一个 Promise，然后处理该 Promise 并直接返回其值。

在未来，这将替换大部分在使用中的 useEffect（更多相关内容请参阅[“再见，UseEffect”](https://www.youtube.com/watch?v=bGzanfKVFeU&ab_channel=BeJS)这个出色的演讲），并且可能成为处理客户端 React 中的异步操作（包括获取）的标准。

目前，仍然建议依赖于 React-Query 和 SWR 等库来满足您的客户端获取需求。尤其要注意获取的行为！

## 那么，它准备好了吗？

实验是前进的本质，我们不能不打破蛋就做出美味的煎蛋卷。我希望本文可以帮助你回答自己特定用例的这个问题。

如果是新项目，我可能会尝试使用 App 目录，并将 Page 目录作为后备或关键业务功能。如果是重构，那就取决于我有多少客户端获取数据。如果很少，就可以开始重构；如果很多，可能要等待完整的解决方案。
