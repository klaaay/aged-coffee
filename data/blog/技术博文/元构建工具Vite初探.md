---
title: 元构建工具vite初探
date: '2021-12-26'
tags: ['react', 'vite', '原创']
draft: false
summary: '基于webpack+react+antd项目的改造'
---

- [写在前面](#写在前面)
- [package.json 的大瘦身](#packagejson-的大瘦身)
- [入口 index.html 文件调整](#入口-indexhtml-文件调整)
- [react 以及 typescript 支持](#react-以及-typescript-支持)
- [less 以及 module-css 支持](#less-以及-module-css-支持)
- [项目的基础 server 及代理配置](#项目的基础-server-及代理配置)
- [绝对路径的 alisa 配置](#绝对路径的-alisa-配置)
- [静态文件的引入](#静态文件的引入)
- [环境变量的配置](#环境变量的配置)
- [manual-chunk 打包优化逻辑](#manual-chunk-打包优化逻辑)
- [奇怪的踩坑](#奇怪的踩坑)
- [完整的 vite 配置](#完整的-vite-配置)
- [写在最后](#写在最后)

## 写在前面

[vite](https://cn.vitejs.dev/)是一种全新的前端构建工具，在构建工具之前加上`元`这个字在这个元宇宙（metaverse）的浪潮下多一个 `meta` 就好似加持了某种神秘的魔力。

由于 react 在 18 中引入了[Server-Side Streaming](https://nextjs.org/blog/next-12#server-side-streaming), [React Server Components](https://nextjs.org/blog/next-12#react-server-components) 的特性，Vue 框架的作者尤雨溪同时也是 Vite 的作者就戏称 React 已经
是 Meta Framework。

![元框架-react](/static/images/react-meta.jpg)

当然我称 Vite 为元构建工具也有名副其实的含义

- meta 这个词取自于希腊语，本身带有`超越`的含义，而 vite 的出现本身就有`超越先阶段项目都基于 webpack 构建这样的现状`的意味。
- 在中文神经元之类的词语中元本身还有`最小功能单元`的意思，而 vite 相较于 webpack 的构建方案，最大的特点之一就是在开发阶段不需要对源码全部打包为一个 bundle 给浏览器运行，而是基于 [esm](https://nodejs.org/api/esm.html) 直接交给现代浏览器运行处理，而一个 `esm` 便可以理解为那个最小的功能单元。

![webpack的打包方案](/static/images/bundler.37740380.png)

![vite的打包方案](/static/images/esm.3070012d.png)

言归正传，之所以会有将项目改造为 vite 驱动的想法，是因为**在开发项目代码量和文件数目变多的情况下，明显感觉 webpack 下的 dev 环境每次冷启动和热加载的速度变慢**，另外自己早些时候也听闻过 vite `丝滑急速的开发体验`，便跃跃欲试。

接下来我们就主要讲一讲将现有的`基于 webpack 的 react+antd 的项目改造为 vite 驱动`的过程

## package.json 的大瘦身

![package大瘦身](/static/images/slim-package.jpeg)

## 入口 index.html 文件调整

`index.html 在项目最外层而不在 public 文件夹内`

webpack

```html
<!DOCTYPE html>
<html>
  <head>
    <link rel="icon" href="%PUBLIC_URL%/logo.ico" />
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root" style="height: 100%"></div>
  </body>
</html>
```

vite

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <link rel="icon" href="/src/assets/logo.ico" />
  </head>
  <body>
    <div id="root"></div>
    <!-- vite 特有的 esm 入口文件 -->
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

## react 以及 typescript 支持

官方插件`@vitejs/plugin-react`提供完整的 react 支持

```ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

// https://cn.vitejs.dev/config/
export default defineConfig({
  plugins: [react()],
})
```

vite 内置了 [esbuild](https://esbuild.github.io/) 对 ts 文件的编译处理`，只需要在项目提供自己的 tsconfig 文件

## less 以及 module-css 支持

vite 内置对 less 以及 module-css 的支持，安装 less 预处理依赖即可
由于 antd 的 less 使用了 less 的函数功能需要在 preprocessorOptions 开启 javascriptEnabled 配置

```ts
export default defineConfig({
  css: {
    preprocessorOptions: {
      less: {
        javascriptEnabled: true,
      },
    },
  },
})
```

## 项目的基础 server 及代理配置

```ts
export default defineConfig({
  server: {
    port: 3000,
    host: 'localhost',
    open: true,
    https: true,
    hmr: {
      host: 'localhost',
    },
    proxy: {
      '/api': "https://tartget-server",
      '/special/api': {
        target: "https://special-tartget-server",,
        changeOrigin: true,
        rewrite: path => path.replace(/^\/special/, ''),
      },
    },
  },
});
```

## 绝对路径的 alisa 配置

- 在项目中我们一般使用绝对路径引入模块，用@来替换根文件夹下的 src 文件夹路径
- 由于部分依赖的 node_modules 本身含有@符，这样的 import 不应该被匹配
- 部分老的 less 的 import 使用了~表示绝对引入的方式，这样的引入需要做特殊的处理

```ts
export default defineConfig({
  resolve: {
    alias: [
      {
        find: /^@\//,
        replacement: `${path.resolve(__dirname, 'src')}${path.sep}`,
      },
      { find: /^~/, replacement: '' },
    ],
  },
})
```

## 静态文件的引入

[一般的静态资源引入 vite 本身支持了多种方案](https://cn.vitejs.dev/guide/assets.html#importing-asset-as-url)
在迁移过程中主要处理了一种需要根据变量拼接静态资源的引用处理逻辑  
由于不在 webpack 环境下无法使用 require 方法，需要用到 vite 这边的 esm 原生功能[import.meta.url](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import.meta)

```ts
// 根据short动态拼接出国旗资源的图片路径
const getFlag = (short: string) => {
  return new URL(`./flags/${short.toLowerCase()}.png`, import.meta.url).href
}
```

## 环境变量的配置

引入 dotenv 相关依赖，启动命令添加 dotenv

```json
{
  "scripts": {
    "dev": "dotenv -e .env.dev vite",
    "build": "tsc && dotenv -e .env.dev vite build",
    "serve": "vite preview"
  },
  "devDependencies": {
    "dotenv": "^8.2.0",
    "dotenv-cli": "^4.0.0",
    "dotenv-expand": "^5.1.0"
  }
}
```

在 vite 中需要遵守约定的前缀规范，`VITE_`

```shell
VITE_VAR=SOME_VALUE
```

在 src 中定义用于静态校验的 ts 文件`env.d.ts`

```ts
interface ImportMetaEnv extends Readonly<Record<string, string>> {
  readonly VITE_VAR: string
}

interface ImportMeta {
  readonly env: ImportMetaEnv
}
```

## manual-chunk 打包优化逻辑

在项目中使用懒加载 React-Router 路由让代码自动 split 一般可以解决大部分场景
在代码分割任然不合理的情况下可以在配置中添加手动处理 chunk 的生成逻辑

```ts
import path from 'path'
import { dependencies } from './package.json'

function renderChunks(deps) {
  let chunks = {}
  Object.keys(deps).forEach((key) => {
    if (key.includes('@types')) return
    if (['react', 'react-router-dom', 'react-dom'].includes(key)) return
    chunks[key] = [key]
  })
  return chunks
}

export default defineConfig({
  // 在 build 之下的逻辑只有生产打包会运行
  build: {
    sourcemap: false,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-router-dom', 'react-dom'],
          ...renderChunks(dependencies),
        },
      },
    },
  },
})
```

## 奇怪的踩坑

- moment 国际化包需要按照不一样的方式引入生效

```ts
// webpack
import 'moment/locale/zh-cn'
// vite
import 'moment/dist/locale/zh-cn'
```

- 部分依赖包在 webpack 生产环境没有问题的包，在 vite 会报错
  react-response 这个包在生产环境下会报错 global 不存在，需要在 script 标签中添加容错逻辑

```html
<script>
  if (global === undefined) {
    var global = window
  }
</script>
```

## 完整的 vite 配置

```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';
import { dependencies } from './package.json';

function renderChunks(deps) {
  let chunks = {};
  Object.keys(deps).forEach(key => {
    if (key.includes('@types')) return;
    if (['react', 'react-router-dom', 'react-dom'].includes(key)) return;
    chunks[key] = [key];
  });
  return chunks;
}

export default defineConfig({
  plugins: [
    react(),
  ],
  css: {
    preprocessorOptions: {
      less: {
        javascriptEnabled: true,
      },
    },
  },
  server: {
    port: 3000,
    host: 'localhost',
    open: true,
    https: true,
    hmr: {
      host: 'localhost',
    },
    proxy: {
      '/api': "https://tartget-server",
      '/special/api': {
        target: "https://special-tartget-server",,
        changeOrigin: true,
        rewrite: path => path.replace(/^\/special/, ''),
      },
    },
  },
  resolve: {
    alias: [
      {
        find: /^@\//,
        replacement: `${path.resolve(__dirname, 'src')}${path.sep}`,
      },
      { find: /^~/, replacement: '' },
    ],
  },
  build: {
    sourcemap: false,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-router-dom', 'react-dom'],
          ...renderChunks(dependencies),
        },
      },
    },
  },
});
```

## 写在最后

1. 迁移过后的项目在最新的 chrome 跑已经没有问题，但对于一些老浏览器的支持还需要验证。  
   (vite 本身提供了老浏览器的支持[插件](https://github.com/vitejs/vite/tree/main/packages/plugin-legacy))

2. `由于vite在开发环境下和生产环境下两种构筑方式上还是存在一定的差异性，对于最终的线上部署可能会带来一定的困扰。`

3. 另外针对迁移的项目对于 antd 的样式引用方式是全量的引入了 less 的源码，加上了自己的 less 主题变量覆盖，`由于浏览器不支持less文件的运行，vite任然需要对less文件进行完整的编译之后项目才可以启动，在less文件多的情况下编译速度较慢` ，之后可以配置 less 的按需引用，以及 less 主题变量的插件注入。
