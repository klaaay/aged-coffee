---
title: React Hooks 是一个错误吗
date: '2023-03-11'
tags: ['react', '博文翻译']
draft: false
summary: ''
---

原文地址：[Were React Hooks a Mistake?](https://jakelazaroff.com/words/were-react-hooks-a-mistake/)

Web 开发社区最近几周一直在谈论信号，这是一种反应式编程模式，可以实现非常高效的 UI 更新。Devon Govett 写了一个引人深思的 [Twitter 线程](https://twitter.com/devongovett/status/1629540226589663233)，讨论了信号和可变状态。Ryan Carniato 回应了一篇[优秀的文章](https://dev.to/this-is-learning/react-vs-signals-10-years-later-3k71)，比较了信号与

讨论表明的一件事是，有很多人对 React 编程模型并不适应。为什么会这样？

我认为问题在于人们对组件的心理模型与使用钩子的函数组件在 React 中的工作方式不匹配。我要做一个大胆的声明：人们喜欢信号，因为基于信号的组件更类似于类组件而不是使用钩子的函数组件。

让我们倒回一点。React 组件以前长成这样

```tsx
class Game extends React.Component {
  state = { count: 0, started: false }

  increment() {
    this.setState({ count: this.state.count + 1 })
  }

  start() {
    if (!this.state.started) setTimeout(() => alert(`Your score was ${this.state.count}!`), 5000)
    this.setState({ started: true })
  }

  render() {
    return (
      <button
        onClick={() => {
          this.increment()
          this.start()
        }}
      >
        {this.state.started ? 'Current score: ' + this.state.count : 'Start'}
      </button>
    )
  }
}
```

每个组件都是 React.Component 类的实例。状态保存在 state 属性中，回调只是实例上的方法。当 React 需要渲染一个组件时，它会调用 render 方法。

你仍然可以编写像这样的组件。语法并没有被删除。但是在 2015 年，React 引入了一些新东西：无状态函数组件（stateless function components）.

```tsx
function CounterButton({ started, count, onClick }) {
  return <button onClick={onClick}>{started ? 'Current score: ' + count : 'Start'}</button>
}

class Game extends React.Component {
  state = { count: 0, started: false }

  increment() {
    this.setState({ count: this.state.count + 1 })
  }

  start() {
    if (!this.state.started) setTimeout(() => alert(`Your score was ${this.state.count}!`), 5000)
    this.setState({ started: true })
  }

  render() {
    return (
      <CounterButton
        started={this.state.started}
        count={this.state.count}
        onClick={() => {
          this.increment()
          this.start()
        }}
      />
    )
  }
}
```

当时，这些组件没有添加状态的方法——必须将其保留在类组件中并作为 props 传递。想法是大多数组件都是无状态的，由树顶附近的一些有状态组件提供支持。

然而，在编写类组件时，情况有点尴尬。有状态逻辑的构成特别棘手。比如说你需要多个不同的类来监听窗口调整大小事件。如果不重写每个实例方法，该怎么办？如果您需要它们与组件状态交互呢？React 试图通过 mixin 解决此问题，但团队很快意识到了缺点。

此外，人们真的很喜欢函数式组件！甚至还有库可以向其中添加状态。因此也许并不奇怪 React 提出了一个内置解决方案：hooks.

```tsx
function Game() {
  const [count, setCount] = useState(0)
  const [started, setStarted] = useState(false)

  function increment() {
    setCount(count + 1)
  }

  function start() {
    if (!started) setTimeout(() => alert(`Your score was ${count}!`), 5000)
    setStarted(true)
  }

  return (
    <button
      onClick={() => {
        increment()
        start()
      }}
    >
      {started ? 'Current score: ' + count : 'Start'}
    </button>
  )
}
```

当我第一次尝试使用 hooks 时，它们真的是一个启示。它们确实使得封装行为和重用有状态逻辑变得容易。我毫不犹豫地跳了进去；自那以后，我写过的唯一类组件就是错误边界。

话虽如此 - 尽管乍一看这个组件与上面的类组件相同，但存在一个重要区别。也许你已经发现了：UI 中的分数将会更新，但当警报出现时，它总是会显示 0。因为 setTimeout 只在第一次调用 start 时发生，并且关闭初始计数值，所以它永远只能看到那个值。

你可能认为可以使用 useEffect 来解决这个问题：

```tsx
function Game() {
  const [count, setCount] = useState(0)
  const [started, setStarted] = useState(false)

  function increment() {
    setCount(count + 1)
  }

  function start() {
    setStarted(true)
  }

  useEffect(() => {
    if (started) {
      const timeout = setTimeout(() => alert(`Your score is ${count}!`), 5000)
      return () => clearTimeout(timeout)
    }
  }, [count, started])

  return (
    <button
      onClick={() => {
        increment()
        start()
      }}
    >
      {started ? 'Current score: ' + count : 'Start'}
    </button>
  )
}
```

这个警报将显示正确的计数。但是有一个新问题：如果您不断点击，游戏永远不会结束！为了防止效果函数闭包变得“陈旧”，我们将 count 和 started 添加到依赖项数组中。每当它们改变时，我们都会获得一个看到更新值的新效果函数。但是该新效果还设置了一个新的超时时间。每次单击按钮时，您都可以在警报出现之前获得五秒钟的新鲜时间。

在类组件中，方法始终可以访问最新状态，因为它们具有对类实例的稳定引用。但是，在函数组件中，每次呈现都会创建关闭其自身状态的新回调。每次调用该函数时，它都会获得自己的闭包。未来渲染无法更改过去渲染的状态。

换句话说：类组件每个已挂载的组件只有一个实例，但函数组件有多个“实例”——每次渲染都会创建一个。Hooks 进一步巩固了这种约束。这是你在使用它们时遇到所有问题的根源：

- 每次渲染都会创建自己的回调函数，这意味着任何在运行副作用之前检查引用相等性的东西——如 useEffect 及其兄弟们——将过于频繁地触发。
- 回调函数封闭了它们所属渲染中的状态和属性，这意味着由于 useCallback、异步操作、超时等原因而持久存在于多次渲染之间的回调函数将访问陈旧数据。

React 给你提供了一个应对这种情况的逃生口：useRef，它是一个可变对象，在渲染之间保持稳定的身份。我认为它是一种在同一已挂载组件的不同实例之间来回传送值的方法。有了这个想法，下面是使用 hooks 的我们游戏可能看起来像什么：

```tsx
function Game() {
  const [count, setCount] = useState(0)
  const [started, setStarted] = useState(false)
  const countRef = useRef(count)

  function increment() {
    setCount(count + 1)
    countRef.current = count + 1
  }

  function start() {
    if (!started) setTimeout(() => alert(`Your score was ${countRef.current}!`), 5000)
    setStarted(true)
  }

  return (
    <button
      onClick={() => {
        increment()
        start()
      }}
    >
      {started ? 'Current score: ' + count : 'Start'}
    </button>
  )
}
```

这很笨重！我们现在要在两个不同的地方跟踪计数，并且我们的增量函数必须同时更新它们。它能够工作的原因是每个启动闭包都可以访问相同的 countRef；当我们在一个闭包中改变它时，所有其他闭包也可以看到已经改变的值。但是我们不能摆脱 useState 只依赖 useRef，因为更改引用不会导致 React 重新渲染。我们陷入了两个不同世界之间 - 用于更新 UI 的不可变状态和具有当前状态的可变引用。

类组件没有这种缺点。每个挂载组件都是类实例给了我们一种内置引用方式。Hooks 为我们提供了更好地组合有状态逻辑所需基础设施，但代价也随之而来。

如果我们使用 Solid 重写我们的小计数器游戏，那么它看起来会像这样：

```tsx
function Game() {
  const [count, setCount] = createSignal(0)
  const [started, setStarted] = createSignal(false)

  function increment() {
    setCount(count() + 1)
  }

  function start() {
    if (!started()) setTimeout(() => alert(`Your score was ${count()}!`), 5000)
    setStarted(true)
  }

  return (
    <button
      onClick={() => {
        increment()
        start()
      }}
    >
      {started() ? 'Current score: ' + count() : 'Start'}
    </button>
  )
}
```

它看起来几乎与第一个 hooks 版本相同！唯一可见的区别是我们调用 createSignal 而不是 useState，并且每当我们想要访问值时，count 和 started 都是我们调用的函数。然而，就像类组件和函数组件一样，外观上的相似性掩盖了一个重要的区别。

Solid 和其他基于信号的框架的关键在于组件只运行一次，并且框架设置了一个数据结构，在信号更改时自动更新 DOM。仅运行组件一次意味着我们只有一个闭包。仅有一个闭包再次为已安装的每个组件提供稳定实例，因为闭包等效于类。

什么？

这是真的！从根本上讲，它们都只是数据和行为的捆绑体。闭包主要是行为（函数调用），带有相关数据（封闭变量），而类主要是数据（实例属性）与相关行为（方法）。如果你真的想，你可以用其中一个来编写另一个。

想一想。使用类组件...

- 构造函数设置了组件渲染所需的所有内容（设置初始状态、绑定实例方法等）。
- 当您更新状态时，React 会改变类实例，调用 render 方法并对 DOM 进行任何必要的更改。
- 所有函数都可以访问存储在类实例上的最新状态。

而使用信号组件...

- 函数体设置了组件渲染所需的所有内容（设置数据流、创建 DOM 节点等）。
- 当您更新信号时，框架会改变存储的值，运行任何依赖信号，并对 DOM 进行任何必要的更改。
- 所有函数都可以访问存储在函数闭包中的最新状态。

从这个角度来看，更容易看到权衡。与类一样，信号是可变的。这可能有点奇怪。毕竟，Solid 组件没有分配任何东西——它调用了 setCount，就像 React 一样！但请记住，count 不是一个值本身——它是一个返回信号当前状态的函数。当调用 setCount 时，它会改变信号，并且进一步对 count() 的调用将返回新值。

尽管 Solid 的 createSignal 看起来像 React 的 useState，但信号实际上更像引用：对可变对象的稳定引用。区别在于，在围绕不可变性构建的 React 中，引用是一个逃生口子，在渲染上没有影响。但是像 Solid 这样的框架将信号放在首位。框架不会忽略它们，在其改变时做出反应，并仅更新使用其值的 DOM 特定部分。

这种情况带来的重大后果是 UI 不再是状态纯函数。这就是为什么 React 拥抱不可变性：它保证状态和 UI 一致。当引入突变时，您还需要一种方法使 UI 保持同步。信号承诺成为实现此目标可靠方式，并且他们成功与否取决于他们履行该承诺能力如何表现好坏

简要概述：

- 首先，我们有类组件，在渲染之间共享单个实例中保留状态。
- 然后，我们有带有钩子的函数组件，在其中每个渲染都具有其自己隔离的实例和状态。
- 现在，我们正在转向信号，再次将状态保留在单个实例中。

那么 React hooks 是一个错误吗？它们确实使得分解组件和重用有状态逻辑变得更容易。即使我打这些字的时候，如果你问我是否会放弃 hooks 并返回到类组件，我会告诉你不会。

同时，我也意识到信号的吸引力在于重新获得我们以前使用类组件时所具有的功能。React 对不可变性进行了大胆尝试，但人们一直在寻找既能保持数据不可变又方便操作的方法。这就是为什么存在像 immer 和 MobX 这样的库：事实证明，使用可变数据的人机交互体验非常方便。

信号比钩子更好吗？我认为这不是正确的问题。每件事都有权衡，我们对信号所做的权衡相当确定：它们放弃了状态的不可变性和 UI 作为纯函数，换取更好的更新性能和每个已安装组件的稳定、可变实例。

时间会告诉我们信号是否会带回 React 创建以解决的问题。但现在，框架似乎正在尝试在钩子的组合性和类别稳定性之间寻求一个舒适点。至少，这是值得探索的选项。
