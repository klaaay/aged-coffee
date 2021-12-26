---
title: 元构建工具vite初探
date: '2021-12-26'
tags: ['react', 'vite']
draft: false
summary: '基于webpack+react+antd项目的改造'
---

- [写在前面](#写在前面)
- [package.json 的大瘦身](#packagejson-的大瘦身)

## 写在前面

[vite](https://cn.vitejs.dev/)是一种全新的前端构建工具，在构建工具之前加上`元`这个字除了在这个元宇宙（metaverse）的浪潮下多一个 `meta` 就加持了某种神秘的魔力。

由于 react 在 18 中引入了[Server-Side Streaming](https://nextjs.org/blog/next-12#server-side-streaming), [React Server Components](https://nextjs.org/blog/next-12#react-server-components) 的特性，Vue 框架的作者尤雨溪同时也是 Vite 的作者就戏称 React 已经
是 Meta Framework。

![元框架-react](/static/images/react-meta.jpg)

当然我称 Vite 为元构建工具其实还有另外含义

- meta 这个词取自于希腊语本身带有`超越`的含义，而 vite 的出现本身就有`超越先阶段基本都基于 webpack 构建这样的现状`的意味。
- 在中文神经元之类的词语中元本身还代表着`最小功能单元`的意思，而 vite 相较于 webpack 的构建方案，最大的特点之一就是在开发阶段不需要对源码全部打包为一个 bundle 给浏览器运行，而是基于 [esm](https://nodejs.org/api/esm.html) 直接交给现代浏览器运行处理，而一个 `esm` 便可以理解为那个最小的功能单元。

![webpack的打包方案](/static/images/bundler.37740380.png)

![vite的打包方案](/static/images/esm.3070012d.png)

言归正传，之所以会有将项目改造为 vite 驱动的想法，是因为**在开发项目代码量和文件数目变多的情况下，明显感觉 webpack 下的 dev 环境每次冷启动和热加载的速度变慢**，另外自己早些时候也听闻过 vite `丝滑急速的开发体验`，便跃跃欲试。

接下来我们就主要讲一讲将现有的`基于 webpack 的 react+antd 的项目改造为 vite 驱动`的过程

## package.json 的大瘦身

![package大瘦身](/static/images/slim-package.jpeg)
