---
title: tech-notes
date: '2022-03-03'
tags: ['资源整理']
draft: false
summary: ''
---

- [FixedForwardRef](#fixedforwardref)
- [渐变 border 的 tailwind 实现](#渐变-border-的-tailwind-实现)
- [ios H5 history.back() 返回上一路由 页面白屏问题](#ios-h5-historyback-返回上一路由-页面白屏问题)
- [Vite 配置 ws 转发](#vite-配置-ws-转发)
- [在屏幕缩放比例发生改变页面不变的处理方案](#在屏幕缩放比例发生改变页面不变的处理方案)
- [antd-mobile css 变量覆盖](#antd-mobile-css-变量覆盖)
- [nginx 常规路径代理配置](#nginx-常规路径代理配置)
- [nginx 网段禁止配置](#nginx-网段禁止配置)
- [nginx-openresty 查看日志位置](#nginx-openresty-查看日志位置)
- [Canvas 工具函数](#canvas-工具函数)
- [nginx 配置正则反向代理](#nginx-配置正则反向代理)
- [FFmpeg 脚本](#ffmpeg-脚本)
- [在 webpack4 中添加强本地缓存](#在-webpack4-中添加强本地缓存)
- [在 nextjs 中支持项目外部的.ts,.tsx 文件编译](#在-nextjs-中支持项目外部的tstsx-文件编译)
- [H5 的使用技巧](#h5-的使用技巧)
- [vite 配合 whistle 跑本地环境的 server-hmr 配置](#vite-配合-whistle-跑本地环境的-server-hmr-配置)

## FixedForwardRef

```tsx
import React, { forwardRef } from " react";
// Declare a type that works with
// generic components
type FixedForwardRef = <T, P= {}>(
  render: (props: P, ref: React.Ref<T>）=> React.ReactNode
）=> (props: P & React.RefAttributes<T>）=> React.ReactNode;

// cast the old forwardRef to the new one
export const fixedForwardRef = forwardRef as FixedForwardRef;
```

## 渐变 border 的 tailwind 实现

```tsx
export const AnimatedGradientBorderTW: React.FC<{
  children: React.ReactNode
}> = ({ children }) => {
  const boxRef = useRef<HTMLDivElement>(null)

  useEffect(() => {
    const boxElement = boxRef.current

    if (!boxElement) {
      return
    }

    const updateAnimation = () => {
      const angle = (parseFloat(boxElement.style.getPropertyValue('--angle')) + 0.5) % 360
      boxElement.style.setProperty('--angle', `${angle}deg`)
      requestAnimationFrame(updateAnimation)
    }

    requestAnimationFrame(updateAnimation)
  }, [])

  return (
    <div
      ref={boxRef}
      style={
        {
          '--angle': '0deg',
          '--border-color': 'linear-gradient(var(--angle), #070707, #687aff)',
          '--bg-color': 'linear-gradient(#131219, #131219)',
        } as CSSProperties
      }
      className="flex h-[400px] w-[400px] items-center justify-center rounded-lg border-2 border-[#0000] p-3 [background:padding-box_var(--bg-color),border-box_var(--border-color)]"
    >
      {children}
    </div>
  )
}
```

## ios H5 history.back() 返回上一路由 页面白屏问题

改变 history.scrollRestoration

使用 history.back 返回上一页的时候，浏览器会记录页面的滚动位置，而在 iOS 上面，滚动后返回的时候页面渲染会出现问题，导致白屏。可以利用 scrollRestoration 属性，它默认是 auto，也就是会记录滚动位置（这是 H5 新增的属性，所以需要判断浏览器是否支持，我实践的是可以兼容大部分移动端机型）

```js
if ('scrollRestoration' in history) {
  history.scrollRestoration = 'manual' //改为 manual 之后，就不会记录滚动位置
}
```

## Vite 配置 ws 转发

![L3PbXs](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/L3PbXs.jpg)

## 在屏幕缩放比例发生改变页面不变的处理方案

1. 首页第一个模块所有的 100vh 全部要动态的改变为 calc（100vh \* 2）
2. 视频的宽度需要调整为 100%（视频动效会没）
3. 登录模态框内部的逻辑调整为 zoom 处理

- 暂定有问题的动效模块直接不展示
- 另外注入 URS 模态框的 css 可以区分 Win 还是 Mac

## antd-mobile css 变量覆盖

```css
:root:root {
  --adm-button-border-radius: 2px;
}
```

## nginx 常规路径代理配置

```
location /docs/zh {
  proxy_pass https://your.domain.com/zh/;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "upgrade";
}
```

## nginx 网段禁止配置

```
server {
  listen       80;
  server_name  domain1.com www.domain1.com;
  access_log   logs/domain1.access.log  main;
  root         html;

  deny 61.135.0.0/16;
}
```

## nginx-openresty 查看日志位置

![rahGOt](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/rahGOt.jpg)

## Canvas 工具函数

```typescript
// 调整图片的尺寸大小
function imageUrl2ExactCanvas(image: HTMLImageElement, imageSize: LimitImageSize) {
  const cvs = document.createElement('canvas')
  const ctx = cvs.getContext('2d')
  const width = imageSize[0]
  const height = imageSize[1]
  cvs.width = width
  cvs.height = height
  ctx!.drawImage(image, 0, 0, cvs.width, cvs.height)
  image2white(ctx!, width, height)
  return cvs
}

// 图片透明部分转换成白色
function image2white(ctx: CanvasRenderingContext2D, width: number, height: number) {
  let imageData = ctx!.getImageData(0, 0, width, height)
  for (let i = 0; i < imageData.data.length; i += 4) {
    // 当该像素是透明的，则设置成白色
    if (imageData.data[i + 3] === 0) {
      imageData.data[i] = 255
      imageData.data[i + 1] = 255
      imageData.data[i + 2] = 255
      imageData.data[i + 3] = 255
    }
  }
  ctx!.putImageData(imageData, 0, 0)
}

/**
 * @param image 图片
 * @param backType 需要返回的类型blob,file
 * @param quality 图片压缩比 0-1,数字越小，图片压缩越小
 * @returns
 */
function compressorImage(image: File, backType?: 'blob' | 'file', quality?: number) {
  return new Promise((resolve, reject) => {
    new Compressor(image, {
      quality: quality || 0.8,
      success(result) {
        let file = new File([result], image.name, { type: image.type })

        if (!backType || backType === 'blob') {
          resolve(result)
        } else if (backType === 'file') {
          resolve(file)
        } else {
          resolve(file)
        }
      },
      error(err) {
        reject(err)
      },
    })
  })
}
```

## nginx 配置正则反向代理

```
resolver 8.8.8.8;

location ~ /xx/([0-9a-z]+).html {
  proxy_pass http://your.domain/xx/$1/
}
```

## FFmpeg 脚本

```shell
ffmpeg -i video.mp4 -i audio.mp4 -c:v copy -c:a aac -strict experimental output.mp4
```

## 在 webpack4 中添加强本地缓存

```js
const HardSourceWebpackPlugin = require('hard-source-webpack-plugin')

plugins: [
  new HardSourceWebpackPlugin({
    cacheDirectory: path.resolve(__dirname, '../cache'),
  }),
]
```

## 在 nextjs 中支持项目外部的.ts,.tsx 文件编译

```javascript
const nextConfig = {
  webpack: (config, options) => {
    config.module.rules.forEach((rule) => {
      if (rule.test && rule.test.toString().includes('tsx|ts')) {
        rule.include = [...rule.include, path.resolve(__dirname, '../common-package/src')]
      }
      if (Array.isArray(rule.oneOf)) {
        rule.oneOf.forEach((rule_inner) => {
          if (rule_inner.test && rule_inner.test.toString().includes('tsx|ts')) {
            rule_inner.include = [
              ...rule_inner.include,
              path.resolve(__dirname, '../common-package/src'),
            ]
          }
        })
      }
    })
    return config
  },
}
```

## H5 的使用技巧

```html
// 1、捕获摄像头，user 表示前置摄像头，environment 表示后置摄像头
<input type="file" capture="user" accept="image/*" />

// 2、每 10s 刷新一次
<meta http-equiv="refresh" content="10" />

// 3、开启 input 输入框的拼写检测
<input type="text" spellcheck="true" lang="en" />

// 4、上传文件时指定允许的文件格式
<input type="file" accept=".jpeg,.png" />

// 5、阻止浏览器翻译，适用场景比如 Logo 和品牌名
<p translate="no">Brand name</p>

// 6、允许选择多个文件
<input type="file" multiple />

// 7、为 video 标签添加缩略图
<video poster="picture.png"></video>

// 8、声明资源文件的下载
<a href="image.png" download></a>
```

## vite 配合 whistle 跑本地环境的 server-hmr 配置

```typescript
server: {
  hmr: {
    protocol: 'ws',
    host: 'localhost',
    },
  },
```
