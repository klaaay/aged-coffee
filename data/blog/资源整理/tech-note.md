---
title: tech-notes
date: '2022-03-03'
tags: ['资源整理']
draft: false
summary: ''
---

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
