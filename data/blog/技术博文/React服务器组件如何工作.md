---
title: React服务器组件如何工作:深入指南
date: '2022-01-30'
tags: ['react', '博文翻译']
draft: false
summary: ''
---

原文地址：[How React server components work: an in-depth guide](https://blog.plasmic.app/posts/how-react-server-components-work)

- [React 服务器组件如何工作:深入指南](#react-服务器组件如何工作深入指南)
  - [什么是 React 服务器组件](#什么是-react-服务器组件)
    - [这不是服务器端渲染吗？](#这不是服务器端渲染吗)
    - [为什么我们会想要这个？](#为什么我们会想要这个)
  - [类比直观的认识](#类比直观的认识)
  - [服务器-客户端组件划分](#服务器-客户端组件划分)
  - [RSC 渲染的生命](#rsc-渲染的生命)
    - [1.服务器接收渲染的请求](#1服务器接收渲染的请求)
    - [2.服务器将根组件元素序列化为 JSON](#2服务器将根组件元素序列化为-json)
    - [3.浏览器重建 React 树](#3浏览器重建-react-树)
    - [他是否支持和 Suspense 一起运作](#他是否支持和-suspense-一起运作)
  - [RSC 线格式](#rsc-线格式)
    - [使用 RSC 格式](#使用-rsc-格式)
    - [为什么不仅仅是输出普通的 HTML？](#为什么不仅仅是输出普通的-html)
    - [这比仅仅从客户端组件获取数据更好吗](#这比仅仅从客户端组件获取数据更好吗)
    - [但是......服务器端渲染怎么样？](#但是服务器端渲染怎么样)
  - [更新服务器组件呈现的内容](#更新服务器组件呈现的内容)
  - [为什么我需要使用 RSC 的元框架？](#为什么我需要使用-rsc-的元框架)
  - [RSC 准备好了吗？](#rsc-准备好了吗)

# React 服务器组件如何工作:深入指南

React 服务器组件（RSC）是一个令人兴奋的新功能，在不久的将来会对页面加载性能、包的大小以及我们如何编写 React 应用程序产生巨大影响。尽管 RSC 在 React 18 中仍然是一个早期的实验性功能，我们一直在挖掘它在引擎盖下的工作原理。在这篇博文中，我们很高兴地与大家分享我们所学到的东西。

## 什么是 React 服务器组件

React 服务器组件允许服务器和客户端（浏览器）在渲染你的 React 应用程序时进行协作，我们开发的页面一般是 React Dom 树渲染的结果，React Dom 树一般是由很多 Component 组成的，RSC 使得 Tree 中的一些 Component 可以由服务器渲染，而一些由浏览器渲染。

这是 React 团队提供的一个快速插图，显示了最终目标是什么：一棵 React 树，其中橙色组件在服务器上渲染，而蓝色组件在客户端渲染。

![A React tree with server components (orange) and client components (blue)](https://blog.plasmic.app/static/images/react-server-components.png)

### 这不是服务器端渲染吗？

**RSC 不是服务端渲染（SSR）！**它有点令人困惑，因为它们都有名称中的“服务器”，它们都在服务器上工作。但它更容易理解它们作为两个独立和正交的功能。使用 RSC 不需要使用 SSR，反之亦然！SSR 模拟一个环境，用于将 React 树渲染为原始 HTML;它不会区分服务器和客户端组件，并且它使它们相同的方式！

不过，也可以将 SSR 和 RSC 结合使用，这样您就可以使用服务器组件进行服务器端呈现，并在浏览器中适当地将它们合并起来。在以后的文章中，我们将更多地讨论它们是如何一起工作的。

但是现在，让我们忽略 SSR，只关注 RSC。

### 为什么我们会想要这个？

在 React Server 组件之前，所有 React 组件都是“客户端”组件 - 它们都在浏览器中运行。当您的浏览器访问反应页面时，它会下载所有必要的 React 组件的代码，构造 React Element 树，并将其呈现给 DOM（或者如果您使用的 SSR，则水解 DOM）。浏览器是一个很好的地方，因为它允许您的 React 应用程序是交互式 - 您可以安装事件处理程序，跟踪状态，响应事件的响应和更新 DOM 的响应。那么我们为什么要在服务器上呈现任何东西？

在服务器上呈现比在浏览器上呈现有一些优势

- 服务器可以更直接地访问您的数据源 - 是他们的数据库，GraphQL 端点或文件系统。服务器可以直接获取所需的数据，而不通过一些公共 API 端点跳跃，通常与您的数据源更紧密地汇总，因此它可以比浏览器更快地获取数据。

- 服务器可以廉价地使用大量的代码模块，比如用于将标记转换为 html 的 npm 包，因为服务器不需要像浏览器那样每次使用时都需要下载这些依赖项，而浏览器必须将所有使用的代码作为 javascript 包下载。

简而言之，**React Server 组件使服务器和浏览器成为可能做到最好的事情。**服务器组件可以专注于获取数据和渲染内容，并且客户端组件可以专注于有状态交互，导致较快的页面加载，较小的 JavaScript 捆绑尺寸以及更好的用户体验。

## 类比直观的认识

_让我们先对它的工作原理有一些直观的认识。_

我的孩子喜欢装饰蛋糕，但他们不是那么抱怨它们。要求他们从头开始制作和装饰杯形蛋糕将是一个（可爱的）噩梦。我需要用手向他们的面粉和糖，棒的黄油，让他们进入烤箱，读它们一吨的指示，并花一整天。但嘿，我可以更快地做烘烤部分;如果我通过第一次烘烤蛋糕并使糖果烘烤，并将那些人交给我的孩子，而不是原料的作品 - 他们可以更快地装饰乐趣！更好，我不需要担心他们所有人都用烤箱担心。赢！

![](https://blog.plasmic.app/static/images/cake.jpg)

React 服务器组件就是为了实现这种分工，让服务器先做它能做的更好，然后再把剩下的交给浏览器完成。这样一来，比起一整袋面粉和一个该死的烤箱，服务员要给的东西更少，12 个小纸杯蛋糕运输起来更有效率

考虑对您的页面的 React 树，有一些要在服务器上呈现的组件以及客户端的一些组件。这是考虑高级策略的一种简化方法：服务器可以像往常一样“渲染”服务器组件，将您的 React 组件转换为 Div 和 P 等本机 HTML 元素。但是，每当它遇到 Client 组件时意味着要在浏览器中呈现，它就刚刚输出占位符，其中包含填充此孔的指令，其中包含右 Client 组件和道具。然后，浏览器采用该输出，填充 Client 组件。

这不是它如何工作，我们即将很快跳入那些真正的粗糙细节;但它是一个有用的高级别画面！

## 服务器-客户端组件划分

但首先，什么是服务器组件?如何确定哪些组件用于服务器，哪些组件用于客户端

React Team 根据组件写入的文件的扩展名：如果文件以`.server.jsx`结尾，则它包含服务器组件;如果它以`.client.jsx`结尾，它包含客户端组件。如果它没有，那么它包含可以用作服务器和客户端组件的组件。

这种定义是务实的 - 开发者和 Bundler 很容易告诉他们分开。专为 Bundler，他们现在能够通过检查文件名来处理不同的客户组件。因为你很快就会看到，Bundler 在制定 RSC 工作方面发挥着重要作用。

由于服务器组件在服务器上运行，而 Client 件在客户端上运行，因此每个都可以执行许多限制。**但要记住的最重要的是 Client 组件无法导入 Server 组件！**这是因为服务器组件无法在浏览器中运行，并且可能有代码在浏览器中不起作用;如果客户端组件依赖于服务器组件，那么我们将最终将这些非法依赖项拉到浏览器包中。

最后一点可能会让人挠头;这意味着像这样的客户端组件是非法的

```typescript
// ClientComponent.client.jsx
// NOT OK:
import ServerComponent from './ServerComponent.server'
export default function ClientComponent() {
  return (
    <div>
      <ServerComponent />
    </div>
  )
}
```

但是，如果客户端组件无法导入服务器组件，因此无法实例化服务器组件，那么我们如何使用这样的反应树结束，使用服务器和客户端组件在一起交错？如何在客户端组件（蓝点）下有服务器组件（橙色点）？

![](https://blog.plasmic.app/static/images/react-server-components.png)

虽然您无法从客户端组件导入和渲染服务器组件，但仍然可以使用组合 - 即，客户端组件仍然可以采用仅是不透明的 ReactNodes，并且可能遇到这些 ReactNodes 由服务器组件呈现。例如：

```typescript
// ClientComponent.client.jsx
export default function ClientComponent({ children }) {
  return (
    <div>
      <h1>Hello from client land</h1>
      {children}
    </div>
  )
}

// ServerComponent.server.jsx
export default function ServerComponent() {
  return <span>Hello from server land</span>
}

// OuterServerComponent.server.jsx
// OuterServerComponent can instantiate both client and server
// components, and we are passing in a <ServerComponent/> as
// the children prop to the ClientComponent.
import ClientComponent from './ClientComponent.client'
import ServerComponent from './ServerComponent.server'
export default function OuterServerComponent() {
  return (
    <ClientComponent>
      <ServerComponent />
    </ClientComponent>
  )
}
```

这种限制将对您如何组织组件以更好地利用 RSC 来重大影响。

## RSC 渲染的生命

让我们潜入当您尝试渲染 React Server 组件时实际发生的内容的细节。您不需要了解这里能够使用服务器组件的所有内容，但它应该为您提供一些直觉如何工作！

### 1.服务器接收渲染的请求

因为服务器需要做一些渲染工作，所以使用 RSC 的页面的生命总是从服务器开始，响应一些 API 调用来渲染一个 React 组件。这个 "根 "组件总是一个服务器组件，它可以渲染其他服务器或客户端组件。服务器根据请求中传递的信息来确定使用哪个服务器组件和什么 props。这个请求通常是以特定 url 的页面请求的形式出现的，尽管 Shopify Hydrogen 有[更精细的方法](https://shopify.dev/custom-storefronts/hydrogen/framework/server-state)，React 团队的官方演示也有一个[原始实现](https://github.com/reactjs/server-components-demo/blob/main/server/api.server.js)。

### 2.服务器将根组件元素序列化为 JSON

这里的最终目标是将最初的根服务器组件渲染成一棵由基本 html 标签和客户端组件 "占位符 "组成的树。然后，我们可以将这棵树序列化，并将其发送给浏览器，而浏览器可以对其进行反序列化，用真正的客户端组件填充客户端占位符，并渲染出最终结果。

所以，在上面的示例之后 - 假设我们想要渲染\<OuterServerComponent />。我们可以刚刚做 json.stringify（\<OuterserverComponent />）来获取序列化元素树吗？

几乎，但不太好！😅 恢复实际的反应元素是 - 一个对象，具有类型字段为字符串 - 对于基本 HTML 标记，如“div” - 或函数 - 对于反应组件实例。

```typescript
// React element for <div>oh my</div>
> React.createElement("div", { title: "oh my" })
{
  $$typeof: Symbol(react.element),
  type: "div",
  props: { title: "oh my" },
  ...
}

// React element for <MyComponent>oh my</MyComponent>
> function MyComponent({children}) {
    return <div>{children}</div>;
  }
> React.createElement(MyComponent, { children: "oh my" });
{
  $$typeof: Symbol(react.element),
  type: MyComponent  // reference to the MyComponent function
  props: { children: "oh my" },
  ...
}
```

当您有一个组件元素时 - 不是基本 HTML 标记元素 - 类型字段引用组件函数，并且功能不是 json-serializable！

要正确 json-stryify 所有内容，React 将[特殊的替换功能](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify#the_replacer_parameter)传递给`JSON.Stringify（）`，该函数适当地处理这些组件函数参考;您可以在[ReastFlightServer.js 中找到它作为 resolvemodeltojson（）](https://github.com/facebook/react/blob/42c30e8b122841d7fe72e28e36848a6de1363b0c/packages/react-server/src/ReactFlightServer.js#L368)。

具体地，每当它看到才能序列化的反应元素时，

- 如果它是基础 HTML 标记（类型字段是一个像“'`div`'）的字符串，那么它已经是序列化的！没什么特别的。

- 如果是服务器组件，则将服务器组件函数（存储在类型字段中）与其道具调用，并序列化结果。这有效地“渲染”服务器组件;这里的目标是将所有服务器组件转换为基础 HTML 标记。

- 如果它是客户组件，那么...实际上已经序列化了！类型字段实际上已经指向模块引用对象，而不是组件函数。等等，什么？！

**什么是“模块参考”对象？**

RSC 引入了 React 元素类型字段的新可能值，称为“模块引用”;而不是组件函数，它是一个可序列化的“引用”。

例如，`ClientComponent`可能看起来像这样：

```typescript
{
  $$typeof: Symbol(react.element),
  // The type field  now has a reference object,
  // instead of the actual component function
  type: {
    $$typeof: Symbol(react.module.reference),
    // ClientComponent is the default export...
    name: "default",
    // from this file!
    filename: "./src/ClientComponent.client.js"
  },
  props: { children: "oh my" },
}
```

但这种手在哪里发生了发生 - 在那里我们正在将客户组件函数的引用转换为可序列化的“模块引用”对象？

事实证明，这是捆绑的是表演这个神奇的技巧！React Team 已发布对`react-server-dom-webpack`的 WebPack 作为[webpack-loader](https://github.com/facebook/react/blob/main/packages/react-server-dom-webpack/src/ReactFlightWebpackNodeLoader.js)或[node-register](https://github.com/facebook/react/blob/main/packages/react-server-dom-webpack/src/ReactFlightWebpackNodeRegister.js)的官方 RSC 支持。当服务器组件从`* .client.jsx`文件导入某些内容时，而不是实际获取该件事，它只获取模块参考对象，其中包含该件事的文件名和导出名称。没有客户端组件函数是在服务器上构建的 React 树的一部分！

再次考虑上面的示例，我们正在尝试序列化\<OuterSuperComponent />;我们最终会享受 json 树：

```typescript
{
  // ClientComponent元素占位符，具有“模块参考”
  $$typeof: Symbol(react.element),
  type: {
    $$typeof: Symbol(react.module.reference),
    name: "default",
    filename: "./src/ClientComponent.client.js"
  },
  props: {
    // children passed to ClientComponent, which was <ServerComponent />.
    children: {
      // ServerComponent gets directly rendered into html tags;
      // notice that there's no reference at all to the
      // ServerComponent - we're directly rendering the `span`.
      $$typeof: Symbol(react.element),
      type: "span",
      props: {
        children: "Hello from server land"
      }
    }
  }
}
```

**可序列化的 React 树**

在此过程结束时，我们希望最终能够使用一个反应树，在服务器上看起来更像是这样的东西，将被发送到浏览器“完成”：

![](https://blog.plasmic.app/static/images/react-server-components-placeholders.png)

**所有 props 都必须是可序列化的**

因为我们正在序列化整个 React 树到 JSON，所以您传递给客户端组件或基本 HTML 标记的所有道具也必须是序列化的。这意味着来自服务器组件，您无法将 event handler 作为 prop 传递！

```typescript
//注意：服务器组件无法将函数传递为一个prop
// to its descendents, because functions are not serializable.
function SomeServerComponent() {
  return <button onClick={() => alert('OHHAI')}>Click me!</button>
}
```

然而，这里要注意的一件事是在 RSC 过程中，当我们遇到客户端组件时，我们从不调用客户端组件函数，或者将“descend”进入客户端组件。因此，如果您有一个实例化另一个客户组件的客户组件：

```typescript
function SomeServerComponent() {
  return <ClientComponent1>Hello world!</ClientComponent1>;
}

function ClientComponent1({children}) {
  // It is okay to pass a function as prop from client to
  // client components
  return <ClientComponent2 onChange={...}>{children}</ClientComponent2>;
}
```

ClientComponent2 在此 RSC JSON 树中根本不会出现;相反，我们将只能看到具有模块引用和 ClientComponent1 的 Props 的元素。因此，ClientComponent1 是完全合法的，将事件处理程序传递给 CliencPonent2。

### 3.浏览器重建 React 树

浏览器从服务器接收 JSON 输出，现在必须启动重建在浏览器中呈现的 React 树。每当我们遇到类型是模块引用的元素时，我们都希望将其替换为正确的客户端组件函数。

这项工作再次需要我们 bundler 的帮助;它是我们的 bundler 替换了客户端组件函数，使用模块在服务器上引用，现在我们的 bundler 知道如何使用浏览器中的真实客户端组件函数替换这些模块引用。

重建的 React 树将看起来像这样 - 只需在：

![](https://blog.plasmic.app/static/images/react-server-components-client.png)

然后我们只是像往常一样渲染并将这棵树提交到 DOM！

### 他是否支持和 Suspense 一起运作

可以一起运作

我们故意在本文中提到 Suspense，因为悬疑是一个巨大的主题，值得自己的博文。但非常短暂的 - Suspense 允许您在需要尚未准备的内容（获取数据，懒惰导入组件等）时从反应组件中抛出 Promise。这些 Promise 被捕获到“Suspense boundary” - 每当从渲染悬念子树中抛出 Promise 时，会暂停渲染该子树直到 Promise Resolved，然后再次尝试。

当我们调用服务器上的服务器组件函数以生成 RSC 输出时，这些功能可能会在获取所需的数据时抛出 Promise。当我们[遇到这样的抛出 Promise 时](https://github.com/facebook/react/blob/42c30e8b122841d7fe72e28e36848a6de1363b0c/packages/react-server/src/ReactFlightServer.js#L416)，我们输出占位符;一旦 Promise 得到解决，我们尝试再次调用服务器组件功能，如果我们成功，请输出已完成的 Chunk。我们实际上是创建 RSC 输出的 Stream，暂停作为 Promise 被抛出，并在解决它们时流汇集额外的 Chunk。

同样，在浏览器中，我们正在将 RSC JSON 输出从我们的`fetch（）`呼叫中流下来。此过程也可能最终遇到输出中的 Suspense 占位符（服务器遇到抛出的 Promise），并且尚未看到 Steam 中的占位符内容[（这里有些细节）](https://github.com/facebook/react/blob/main/packages/react-client/src/ReactFlightClientStream.js)的情况结束，并尚未见到占位符，或者，如果它遇到客户端组件模块引用，则可能还抛出 Promise，但在浏览器中尚未加载该客户端组件函数 - 在这种情况下，bundler 运行时必须[动态获取必要的 chunks](https://github.com/facebook/react/blob/main/packages/react-server-dom-webpack/src/ReactFlightClientWebpackBundlerConfig.js)。

由于 Suspense，您可以将服务器流式传输 RSC 输出作为服务器组件获取数据，并且您将浏览器逐步呈现数据，并在其变得可用时呈现数据，并在必要时动态获取客户端组件包。

## RSC 线格式

但是服务器究竟是什么？如果你读到“json”和“流”时抬起眉毛，你是对持怀疑态度的权利！那么，服务器流传输到浏览器的数据是什么？

这是一种简单格式，每行上有一个 json blob，标记为一个 id。以下是我们\<OuterServerComponent/>：

```
M1:{"id":"./src/ClientComponent.client.js","chunks":["client1"],"name":""}
J0:["$","@1",null,{"children":["$","span",null,{"children":"Hello from server land"}]}]
```

在上面的代码段中，以`M`开头的行定义了客户端组件模块的参考，其中需要查找客户端捆绑包中的组件函数所需的信息。以`J`开头的行定义了一个实际的 React Element 树，其中包含`@1`引用由`M`行定义的客户端组件。

此格式非常符合 stream 的方式 - 一旦客户端读取整行，它就可以解析了一个 json 片段并进行了一些 progress。如果服务器在渲染时遇到 Suspense Boundaries，则会看到与已解决的每个 Chunk 对应的多个`J`行。

例如，让我们的示例更有趣......

```typescript
// Tweets.server.js
import { fetch } from 'react-fetch' // React's Suspense-aware fetch()
import Tweet from './Tweet.client'
export default function Tweets() {
  const tweets = fetch(`/tweets`).json()
  return (
    <ul>
      {tweets.slice(0, 2).map((tweet) => (
        <li>
          <Tweet tweet={tweet} />
        </li>
      ))}
    </ul>
  )
}

// Tweet.client.js
export default function Tweet({ tweet }) {
  return <div onClick={() => alert(`Written by ${tweet.username}`)}>{tweet.body}</div>
}

// OuterServerComponent.server.js
export default function OuterServerComponent() {
  return (
    <ClientComponent>
      <ServerComponent />
      <Suspense fallback={'Loading tweets...'}>
        <Tweets />
      </Suspense>
    </ClientComponent>
  )
}
```

在这种情况下，RSC 流是什么样的？

```
M1:{"id":"./src/ClientComponent.client.js","chunks":["client1"],"name":""}
S2:"react.suspense"
J0:["$","@1",null,{"children":[["$","span",null,{"children":"Hello from server land"}],["$","$2",null,{"fallback":"Loading tweets...","children":"@3"}]]}]
M4:{"id":"./src/Tweet.client.js","chunks":["client8"],"name":""}
J3:["$","ul",null,{"children":[["$","li",null,{"children":["$","@4",null,{"tweet":{...}}}]}],["$","li",null,{"children":["$","@4",null,{"tweet":{...}}}]}]]}]
```

`J0`线现在有一个额外的 Children - 新的`Suspense boundary`，children 指向引用`@3`。有趣的是在这里注意到`@3`目前尚未定义！当服务器完成加载推文时，它会输出`M4`的行 - 它定义了对`Tweet.Client.js`组件 - 和`J3`的模块引用 - 它定义了另一个应将其交换到`@3`所在的其他反应元素树（且再次请注意，`J3`children 正在引用`M4`中定义的`Tweet组件`）。

这里有另一件事要注意，那是 Bundler 将 ClientComponent 和 Tweet 自动将 ClientComponent 和 Tweet 分为两个单独的捆绑包，这允许浏览器推迟到以后推出推文 bundle 包！

### 使用 RSC 格式

如何将此 RSC 流转换为浏览器中的实际反应元素？`React-server-dom-webpack`包含 rsc 响应并重新创建 React Element 树的[入口](https://github.com/facebook/react/blob/main/packages/react-server-dom-webpack/src/ReactFlightDOMClient.js)点。以下是您的根客户端组件可能如下所示的简化版本：

```typescript
import { createFromFetch } from 'react-server-dom-webpack'
function ClientRootComponent() {
  // fetch() from our RSC API endpoint.  react-server-dom-webpack
  // can then take the fetch result and reconstruct the React
  // element tree
  const response = createFromFetch(fetch('/rsc?...'))
  return <Suspense fallback={null}>{response.readRoot() /* Returns a React element! */}</Suspense>
}
```

您要求`React-Server-DOM-WebPack`从 API 端点读取 RSC 响应。然后，`Response.Readroot（）`返回更新的反应元素，因为响应流被处理过！在读取任何流之前，它将立即抛出 Promise - 因为尚未准备好内容。然后，当它处理第一个`J0`时，它会创建相应的 React Element 树并解析抛出的 Promise。React Resumes 渲染，但是当遇到未且准备就绪`@3`引用时，另一个 Promise 被抛出。一旦它读取`J3`，该 Promise 得到解决，并再次恢复渲染，这次完成。因此，随着我们将 RSC 响应流流，我们将继续更新和呈现我们在 Suspense boundary 定义的 Chunk 中的元素树，直到我们完成。

### 为什么不仅仅是输出普通的 HTML？

为什么要发明全新的线材格式？客户端上的目标是重建 React Element 树。从这种格式完成这一格式比 HTML 更容易，其中我们必须解析 HTML 以创建 React Elements。请注意，React Element 树的重建是重要的，因为这允许我们将后续更改合并到 DOM 的最小提交。

### 这比仅仅从客户端组件获取数据更好吗

如果我们需要向服务器进行 API 请求以获取此内容，这比提出要获取数据的请求，然后在客户中完全渲染，正如我们今天所做的那样？

最终，它取决于您在屏幕上呈现的内容。使用 RSC，您获得了非常规的能力，“处理过的”数据直接展示给用户，因此如果您只渲染您将要获取的小型数据，或者渲染本身需要一个许多您想要避免下载到浏览器的 javascript。如果渲染需要很多后台数据获取，那么在服务端运行相关数据会更好，其中数据延迟远低于浏览器。

### 但是......服务器端渲染怎么样？

我知道我知道我知道。通过 React 18，可以将 SSR 和 RSC 组合起来，以便您可以在服务器上生成 HTML，然后在浏览器中使用 RSC 的 HTML 水合物。在这个主题上留下来的更多关注！

## 更新服务器组件呈现的内容

如果您需要服务器组件呈现新的内容，例如，例如，如果要在将一个产品视为其他产品之间的页面之间切换，则为：

同样，由于渲染发生在服务器上，这需要另一个 API 调用服务器以获得 RSC 线格式的新内容。好消息是，一旦浏览器收到新内容，它可以构造一个新的 React 元素树，并执行与上一个 React 树的常用协调，以找出 DOM 所需的最小更新，而驻留状态和事件。您的客户端组件中的处理程序。对于客户端组件，如果它完全在浏览器中发生，则此更新将与其不同。

`现在，您必须从根服务器组件重新渲染整个root server component`，但在将来，可能会为子树执行此操作。

## 为什么我需要使用 RSC 的元框架？

React 团队表示，RSC 最初是通过[Next.js](https://github.com/vercel/next.js/issues/30994)或[Shopify Hydrogen](https://github.com/Shopify/hydrogen/issues/250)的[Meta-Frameworks](https://github.com/josephsavona/rfcs/blob/server-components/text/0000-server-components.md#adoption-strategy)采用，而不是直接用于普通的反应项目。但为什么？元框架为你做了什么？

你不必，但它会让你的生活更轻松。元框架提供友好的包装器和抽象，因此您永远不必考虑在服务器中生成 RSC 流，并在浏览器中消耗它。元框架也支持服务器端渲染，并且他们正在进行工作，以确保如果您使用的是服务器组件，则可以正确保密服务器生成的 HTML。

如您所见，您还需要从 Bundler 中的合作才能在浏览器中妥善发货和使用客户端组件。已经有一个 WebPack 集成，而且 shopify 正在研究[Vite 集成](https://github.com/facebook/react/pull/22952)。这些插件需要成为 React Repo 的一部分，因为 RSC 所需的许多件未作为公共 NPM 包发布。但是，一旦开发，应该可以使用这些碎片而没有涉及的元框架。

## RSC 准备好了吗？

[React Server 组件现在可作为 Next.js 下一个实验功能](https://nextjs.org/docs/advanced-features/react-18)，[并在当前的开发人员预览中用于 Shopify Hydrogen](https://hydrogen.shopify.dev/)，但也没有准备好生产使用。在未来的博客文章中，我们将潜入其中每个框架如何使用 RSC。

但是，毫无疑问，React Server 组件将是反应未来的重要组成部分。它是 React 的回答，以更快的页面加载，较小的 JavaScript 捆绑包，更短的时间交互式 - 更全面的论点是如何使用 React 构建多页应用程序。它可能还没有准备好，但很快就会开始关注。
