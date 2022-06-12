---
title: tech-notes
date: '2022-03-03'
tags: ['笔记', '技术笔记']
draft: false
summary: ''
---

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
