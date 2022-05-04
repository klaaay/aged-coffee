---
title: tech-notes
date: '2022-03-03'
tags: ['笔记', '技术笔记']
draft: false
summary: ''
---

## vite 配合 whistle 跑本地环境的 server-hmr 配置

```typescript
server: {
  hmr: {
    protocol: 'ws',
    host: 'localhost',
    },
  },
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
