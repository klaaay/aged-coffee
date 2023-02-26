---
title: React-Hooks 复合组件
date: '2023-02-25'
tags: ['react', '博文翻译']
draft: false
summary: ''
---

# React-Hooks 复合组件

[原文地址](https://kentcdodds.com/blog/compound-components-with-react-hooks?utm_source=reactdigest&utm_medium&utm_campaign=1527)

复合组件是你有两个或更多的组件一起工作来完成一项有用的任务。通常，其中一个组件是父级，另一个是子级。目标是提供更富表达性和灵活性的 API。

把它想象成`<select>`和`<option>`：

```html
<select>
  <option value="value1">key1</option>
  <option value="value2">key2</option>
  <option value="value3">key3</option>
</select>
```

如果你试图只用其中一个而不使用另一个，它就不会工作（或者没有意义）。此外，这实际上是一个非常出色的 API。让我们看看如果我们没有组合组件 API 来使用时会是什么样子（请记住，这是 HTML 而不是 JSX）

```html
<select options="key1:value1;key2:value2;key3:value3"></select>
```

我相信你可以想出其它表达这个意思的方式，但是太恶心了。那么用这种 API 如何表达 disabled 属性呢？有点疯狂。

所以，复合组件 API 为您提供了一种表达组件之间关系的好方法。

另一个重要方面是“隐式状态”的概念。`<select>`元素会隐式存储有关所选选项的状态，并将该状态与它的子元素分享，以便它们根据该状态来呈现自身。但这种共享是隐式的，因为我们的 HTML 代码中甚至无法访问此状态 (而且也不需要)。

好的，让我们看一下一个合法的 React 组件，它暴露了复合组件来进一步理解这些原则。这里是 Reach UI 中`<Menu />`组件的例子，它暴露了复合组件 API：

```tsx
function App() {
  return (
    <Menu>
      <MenuButton>
        Actions <span aria-hidden>▾</span>
      </MenuButton>
      <MenuList>
        <MenuItem onSelect={() => alert('Download')}>Download</MenuItem>
        <MenuItem onSelect={() => alert('Copy')}>Create a Copy</MenuItem>
        <MenuItem onSelect={() => alert('Delete')}>Delete</MenuItem>
      </MenuList>
    </Menu>
  )
}
```

在这个例子中，`<Menu>`建立了一些共享的隐式状态。 `<MenuButton>`、`<MenuList>`和`<MenuItem>`组件都可以访问或操作该状态，而且都是隐式实现的。这样就可以提供你想要的表达 API。

那么这是如何做到的呢？好吧，如果你观看我的课程，我会向你展示两种方法来实现。一个使用 React.cloneElement 在子元素上，另一个使用 React context。 （我的课程需要略微更新以显示如何使用 hooks 来实现此目的）。在本博客文章中，我将向您展示如何使用 context 创建一套简单的复合部件。

在教授一个新的概念时，我喜欢先使用简单的例子。所以我们将使用我最喜欢的`<Toggle>`组件例子来进行讲解。

这是我们如何使用`<Toggle>`复合组件的方式：

```tsx
function App() {
  return (
    <Toggle onToggle={(on) => console.log(on)}>
      <ToggleOn>The button is on</ToggleOn>
      <ToggleOff>The button is off</ToggleOff>
      <ToggleButton />
    </Toggle>
  )
}
```

好的，你们都在等待的时刻到了，实际上使用上下文和挂钩实现复合组件的全部代码。

```tsx
import * as React from 'react'
// this switch implements a checkbox input and is not relevant for this example
import { Switch } from '../switch'

const ToggleContext = React.createContext()

function useEffectAfterMount(cb, dependencies) {
  const justMounted = React.useRef(true)
  React.useEffect(() => {
    if (!justMounted.current) {
      return cb()
    }
    justMounted.current = false
  }, dependencies)
}

function Toggle(props) {
  const [on, setOn] = React.useState(false)
  const toggle = React.useCallback(() => setOn((oldOn) => !oldOn), [])
  useEffectAfterMount(() => {
    props.onToggle(on)
  }, [on])
  const value = React.useMemo(() => ({ on, toggle }), [on])
  return <ToggleContext.Provider value={value}>{props.children}</ToggleContext.Provider>
}

function useToggleContext() {
  const context = React.useContext(ToggleContext)
  if (!context) {
    throw new Error(`Toggle compound components cannot be rendered outside the Toggle component`)
  }
  return context
}

function ToggleOn({ children }) {
  const { on } = useToggleContext()
  return on ? children : null
}

function ToggleOff({ children }) {
  const { on } = useToggleContext()
  return on ? null : children
}

function ToggleButton(props) {
  const { on, toggle } = useToggleContext()
  return <Switch on={on} onClick={toggle} {...props} />
}
```

[在线展示](https://codesandbox.io/s/9yp5p2z7yr?from-embed)

所以这个工作原理是我们使用 React 创建一个上下文，在其中存储状态和更新状态的机制。然后`<Toggle>`组件负责将该上下文值提供给 React 树的其余部分。
