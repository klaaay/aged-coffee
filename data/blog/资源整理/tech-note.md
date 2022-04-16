---
title: tech-notes
date: '2022-03-03'
tags: ['资源整理', '技术笔记']
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
