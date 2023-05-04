# tools-collection

# mac-software

# Mac 常用软件

## 通用

- 网易邮箱大师
- 微信
- 谷歌浏览器
- 有道云笔记
- 网易云音乐
- QQ 音乐
- VSCODE
- Docker
- uTorrent
- 阿里云盘
- Eagle

## 效率

- Notion - 内容管理
- Paste - 粘贴板工具
- Alfred 4 - 搜索增强
- [Fluent Reader - RSS 阅读](https://github.com/yang991178/fluent-reader)
- Ego Reader - RSS 阅读
- [Eagle - 图片管理](https://eagle.cool/)

## 工具

- Beyond-Compare - 文件对比
- Magnet - 分屏工具
- eZip - 压缩工具
- zsh - 命令行
- [wrap - 命令行扩展](https://www.warp.dev/)
- Shottr - 截屏工具
- Jump-Desktop - 远程连接
- [OpenInTerminal - terminal 工具](https://github.com/Ji4n1ng/OpenInTerminal)
- [SwitchKey - 输入法自动切换](https://github.com/itsuhane/SwitchKey)
- [TinyPNG4Mac - 图片压缩](https://github.com/kyleduo/TinyPNG4Mac)
- [uPic - 图床](https://github.com/gee1k/uPic)
- [ShadoShadowsocksX-NG-R8 - 翻墙](https://github.com/shadowsocks/ShadowsocksX-NG)
- [KeepingYouAwake - 电脑休眠](https://github.com/newmarcel/KeepingYouAwake)
- [Input Source pro - 输入法切换](https://inputsource.pro/zh-CN?utm_source=appinn.com)
- [Dropover - 文件拖拽中转](https://apps.apple.com/cn/app/dropover/id1355679052?mt=12)
- [Bartender - Mac 菜单栏管理](https://www.macbartender.com/)
- [Omi 录屏专家](https://apps.apple.com/cn/app/+%BD%95%E5%B1%8F%E4%B8%93%E5%AE%B6omi-%E5%B1%8F%E5%B9%95%E5%BD%95%E5%88%B6%E5%B7%A5%E5%85%B7/id1592987853?mt=12)
- [Downie 4 - 视频下载](https://software.charliemonroe.net/downie/)
- [Cursor-pro - 光标优化](https://apps.apple.com/us/app/cursor-pro/id1447043133?mt=12)
- [stats - 系统监控](https://github.com/exelban/stats)

## 开发

- [orbstack](https://orbstack.dev/)
- [sequel-ace](https://sequel-ace.com/)
- [VirtualBuddy](https://github.com/insidegui/VirtualBuddy)
- [virtualbox](https://www.virtualbox.org/)
- [Apifox](https://apifox.com/)

## 娱乐

- IINA - 视频播放器

# 命令行工具

- [常见替代](#常见替代)
- [功能](#功能)

## 常见替代

- [procs](https://github.com/dalance/procs)  
   查看系统运行的进程

```shell
procs vscode
```

- [bat](https://github.com/sharkdp/bat)  
   cat 的高亮替代版本用于查看文件

```shell
bat README.md
```

- [fd](https://github.com/sharkdp/fd)  
   find 的替代版本用于查找文件

```shell
fd passwd /etc
/etc/default/passwd
/etc/pam.d/passwd
/etc/passwd
```

- [exa](https://github.com/ogham/exa)  
   ls 的替代版本查看文件夹文件信息

```shell
exa -l
exa --tree --level=2
```

## 功能

- [rg](https://github.com/BurntSushi/ripgrep)  
   正则查找文件夹或者文件中的内容

```shell
rg 'fast\w*' README.md
```

- [ctop](https://github.com/bcicen/ctop)  
   运行程序资源分布查看

- [mac-cleanup](https://github.com/mac-cleanup/mac-cleanup-sh)
  Mac OS Cleanup

# VSCODE 插件

## 视觉

- [Peacock](https://marketplace.visualstudio.com/items?itemName=johnpapa.vscode-peacock)  
   给 vscode 上色
- [Bracket Pair Colorizer](https://marketplace.visualstudio.com/items?itemName=CoenraadS.bracket-pair-colorizer)
- [Material Icon Theme](https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme)
- [Night Owl](https://marketplace.visualstudio.com/items?itemName=sdras.night-owl)
- [One Dark Pro](https://marketplace.visualstudio.com/items?itemName=zhuangtongfa.Material-theme)
- [styled-jsx](https://marketplace.visualstudio.com/items?itemName=blanu.vscode-styled-jsx)

## 效率

- [Auto Close Tag](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-close-tag)
- [Auto Rename Tag](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-rename-tag)
- [ES7 React/Redux/GraphQL/React-Native snippets](https://marketplace.visualstudio.com/items?itemName=dsznajder.es7-react-js-snippets)
- [VS Code JavaScript ES6 snippets](https://marketplace.visualstudio.com/items?itemName=xabikos.JavaScriptSnippets)
- [Less IntelliSense](https://marketplace.visualstudio.com/items?itemName=mrmlnc.vscode-less)
- [IntelliSense for CSS class names in HTML](https://marketplace.visualstudio.com/items?itemName=Zignd.html-css-class-completion)
- [Turbo Console Log](https://marketplace.visualstudio.com/items?itemName=ChakrounAnas.turbo-console-log)
- [Better Comments](https://marketplace.visualstudio.com/items?itemName=aaron-bond.better-comments)

## 风格

- [Codelf](https://marketplace.visualstudio.com/items?itemName=unbug.codelf)
- [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)
- [Document This](https://marketplace.visualstudio.com/items?itemName=oouo-diogo-perdigao.docthis)
- [koroFileHeader](https://marketplace.visualstudio.com/items?itemName=OBKoro1.korofileheader)
- [Project Manager](https://marketplace.visualstudio.com/items?itemName=alefragnani.project-manager)  
   git 项目的归类管理

## 提示

- [Code Spell Checker](https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker)
- [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
- [VersionLens](https://marketplace.visualstudio.com/items?itemName=pflannery.vscode-versionlens)

## 功能

- [GitLens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens)
- [Import Cost VSCode Extension](https://marketplace.visualstudio.com/items?itemName=wix.vscode-import-cost)
- [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer)
- [Markdown All in One](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one)
- [MDX](https://marketplace.visualstudio.com/items?itemName=silvenon.mdx)
- [REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client)
- [TODO+](https://marketplace.visualstudio.com/items?itemName=fabiospampinato.vscode-todo-plus)
- [TODO Highlight](https://marketplace.visualstudio.com/items?itemName=wayou.vscode-todo-highlight)
- [i18n Ally](https://marketplace.visualstudio.com/items?itemName=Lokalise.i18n-ally)

## AI

- [codeium](https://codeium.com/)
- [GitHub Copilot](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot)
