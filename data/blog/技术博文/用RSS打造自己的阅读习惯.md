---
title: 用 RSS 打造自己的阅读习惯
date: '2022-10-30'
tags: ['通用', '原创']
draft: false
summary: '用 RSS 打造自己的阅读习惯'
---

- [RSS 是什么 & 有什么好处](#rss-是什么--有什么好处)
- [RSS 阅读器推荐](#rss-阅读器推荐)
- [如何找 RSS 订阅源](#如何找-rss-订阅源)
  - [RSSHub](#rsshub)
  - [辅助查找网站上的 RSS 订阅源](#辅助查找网站上的-rss-订阅源)
    - [谷歌插件](#谷歌插件)
    - [安卓 APP](#安卓-app)
    - [IOS APP](#ios-app)
  - [看自己喜欢的博客有没有 RSS 订阅](#看自己喜欢的博客有没有-rss-订阅)
- [添加 RSS 订阅源 & 使用 RSS 阅读器](#添加-rss-订阅源--使用-rss-阅读器)
- [部署自己的 RSShub](#部署自己的-rsshub)
- [在自己的博客上添加 RSS 订阅](#在自己的博客上添加-rss-订阅)

## RSS 是什么 & 有什么好处

简易信息聚合（也叫聚合内容）是一种基于 XML 的标准，在互联网上被广泛采用的内容包装和投递协议。RSS(Really Simple Syndication) 是一种描述和同步网站内容的格式，是使用最广泛的 XML 应用。RSS 搭建了信息迅速传播的一个技术平台，使得每个人都成为潜在的信息提供者。

- 本质是一个 xml 文件
- xml 中含有当前网站的一些摘要信息
- 可以被订阅接收
- 不受具体平台的约束
- 可以通过 RSS 阅读器来将不同的信息源聚合
- 无广告

## RSS 阅读器推荐

[Fluent-Reader 电脑端](https://hyliu.me/fluent-reader/)

![fluent-reader](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/UnyAP1.jpg)

[Ego-Reader 移动端](https://egorss.com/zh/)

![ego-reader](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/fyoPvc.jpg)

## 如何找 RSS 订阅源

### RSSHub

RSSHub 是一个开源、简单易用、易于扩展的 RSS 生成器，可以给任何奇奇怪怪的内容生成 RSS 订阅源。RSSHub 借助于开源社区的力量快速发展中，目前已适配数百家网站的上千项内容

![XiTNAl](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/XiTNAl.jpg)

### 辅助查找网站上的 RSS 订阅源

#### 谷歌插件

[rss-reader-chrome](https://chrome.google.com/webstore/detail/rsshub-radar/kefjpfngnndepjbopdmoebkipbgkggaa)

![ID50BU](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/ID50BU.jpg)

#### 安卓 APP

[RSSAid](https://github.com/LeetaoGoooo/RSSAid)

#### IOS APP

[RSSBud](https://github.com/Cay-Zhang/RSSBud)

### 看自己喜欢的博客有没有 RSS 订阅

![7Qk29t](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/7Qk29t.jpg)

前端优质博客推荐

- [Antfu](https://antfu.me/)
- [Robin Wieruch](https://www.robinwieruch.de)
- [Josh Comeau](https://www.joshwcomeau.com/)
- [Dan Abramov](https://overreacted.io/)

## 添加 RSS 订阅源 & 使用 RSS 阅读器

Fluent-Reader  
![wBUg6e](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/wBUg6e.jpg)

![sHQMCh](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/sHQMCh.jpg)

Ego-Reader
![GvAWAS](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/GvAWAS.jpg)

![fpXTH1](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/fpXTH1.jpg)

## 部署自己的 RSShub

官网提供了很多的部署方式，建议使用 fork 代码仓库使用 Vercel 提供的免费云平台部署自己的服务

[vercel](https://docs.rsshub.app/install/#bu-shu-dao-vercel-zeit-now)

![YFynGM](https://cdn.jsdelivr.net/gh/klaaay/pbed@main/uPic/YFynGM.jpg)

## 在自己的博客上添加 RSS 订阅

```ts
// 生成 RSS 订阅条目需要的信息
const generateRssItem = (post) => `
  <item>
    <guid>${siteMetadata.siteUrl}/blog/${post.slug}</guid>
    <title>${escape(post.title)}</title>
    <link>${siteMetadata.siteUrl}/blog/${post.slug}</link>
    ${post.summary && `<description>${escape(post.summary)}</description>`}
    <pubDate>${new Date(post.date).toUTCString()}</pubDate>
    <author>${siteMetadata.email} (${siteMetadata.author})</author>
    ${post.tags && post.tags.map((t) => `<category>${t}</category>`).join('')}
  </item>
`

const generateRss = (posts, page = 'feed.xml') => `
  <rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
    <channel>
      <title>${escape(siteMetadata.title)}</title>
      <link>${siteMetadata.siteUrl}/blog</link>
      <description>${escape(siteMetadata.description)}</description>
      <language>${siteMetadata.language}</language>
      <managingEditor>${siteMetadata.email} (${siteMetadata.author})</managingEditor>
      <webMaster>${siteMetadata.email} (${siteMetadata.author})</webMaster>
      <lastBuildDate>${new Date(posts[0].date).toUTCString()}</lastBuildDate>
      <atom:link href="${siteMetadata.siteUrl}/${page}" rel="self" type="application/rss+xml"/>
      ${posts.map(generateRssItem).join('')}
    </channel>
  </rss>
`

// 以 next.js 为例在生成页面的时候生成 feed.xml RSS 订阅源文件
export async function getStaticProps({ params }) {
  const allPosts = await getAllFilesFrontMatter('blog')
  const postIndex = allPosts.findIndex((post) => formatSlug(post.slug) === params.slug.join('/'))
  const prev = allPosts[postIndex + 1] || null
  const next = allPosts[postIndex - 1] || null
  const post = await getFileBySlug('blog', params.slug.join('/'))
  const authorList = post.frontMatter.authors || ['default']
  const authorPromise = authorList.map(async (author) => {
    const authorResults = await getFileBySlug('authors', [author])
    return authorResults.frontMatter
  })
  const authorDetails = await Promise.all(authorPromise)

  // rss
  const rss = generateRss(allPosts)
  fs.writeFileSync('./public/feed.xml', rss)

  return { props: { post, authorDetails, prev, next } }
}
```
