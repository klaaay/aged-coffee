---
title: tools-collection
date: '2022-08-21'
tags: ['资源整理']
draft: false
summary: ''
---

- [Mac 常用软件](#mac-常用软件)
  - [通用](#通用)
  - [效率](#效率)
  - [工具](#工具)
  - [开发](#开发)
  - [娱乐](#娱乐)
- [命令行工具](#命令行工具)
  - [常见替代](#常见替代)
  - [功能](#功能)
- [VSCODE 插件](#vscode-插件)
  - [视觉](#视觉)
  - [效率](#效率-1)
  - [风格](#风格)
  - [提示](#提示)
  - [功能](#功能-1)
  - [AI](#ai)
- [快捷键](#快捷键)
  - [control + option + command](#control--option--command)
  - [control + option](#control--option)
  - [command + shift](#command--shift)
  - [command](#command)
  - [alt](#alt)
- [zshrc](#zshrc)

## Mac 常用软件

### 通用

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

### 效率

- Notion - 内容管理
- Paste - 粘贴板工具
- Alfred 4 - 搜索增强
- [Fluent Reader - RSS 阅读](https://github.com/yang991178/fluent-reader)
- Ego Reader - RSS 阅读
- [Eagle - 图片管理](https://eagle.cool/)

### 工具

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

### 开发

- [orbstack](https://orbstack.dev/)
- [sequel-ace](https://sequel-ace.com/)
- [VirtualBuddy](https://github.com/insidegui/VirtualBuddy)
- [virtualbox](https://www.virtualbox.org/)
- [Apifox](https://apifox.com/)

### 娱乐

- IINA - 视频播放器

## 命令行工具

### 常见替代

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

### 功能

- [rg](https://github.com/BurntSushi/ripgrep)  
   正则查找文件夹或者文件中的内容

```shell
rg 'fast\w*' README.md
```

- [ctop](https://github.com/bcicen/ctop)  
   运行程序资源分布查看

- [mac-cleanup](https://github.com/mac-cleanup/mac-cleanup-sh)
  Mac OS Cleanup

- [upscayl](https://www.upscayl.org/)  
  AI 提升图片画质

## VSCODE 插件

### 视觉

- [Peacock](https://marketplace.visualstudio.com/items?itemName=johnpapa.vscode-peacock)  
   给 vscode 上色
- [Bracket Pair Colorizer](https://marketplace.visualstudio.com/items?itemName=CoenraadS.bracket-pair-colorizer)
- [Material Icon Theme](https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme)
- [Night Owl](https://marketplace.visualstudio.com/items?itemName=sdras.night-owl)
- [One Dark Pro](https://marketplace.visualstudio.com/items?itemName=zhuangtongfa.Material-theme)
- [styled-jsx](https://marketplace.visualstudio.com/items?itemName=blanu.vscode-styled-jsx)

### 效率

- [Auto Close Tag](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-close-tag)
- [Auto Rename Tag](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-rename-tag)
- [ES7 React/Redux/GraphQL/React-Native snippets](https://marketplace.visualstudio.com/items?itemName=dsznajder.es7-react-js-snippets)
- [VS Code JavaScript ES6 snippets](https://marketplace.visualstudio.com/items?itemName=xabikos.JavaScriptSnippets)
- [Less IntelliSense](https://marketplace.visualstudio.com/items?itemName=mrmlnc.vscode-less)
- [IntelliSense for CSS class names in HTML](https://marketplace.visualstudio.com/items?itemName=Zignd.html-css-class-completion)
- [Turbo Console Log](https://marketplace.visualstudio.com/items?itemName=ChakrounAnas.turbo-console-log)
- [Better Comments](https://marketplace.visualstudio.com/items?itemName=aaron-bond.better-comments)

### 风格

- [Codelf](https://marketplace.visualstudio.com/items?itemName=unbug.codelf)
- [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)
- [Document This](https://marketplace.visualstudio.com/items?itemName=oouo-diogo-perdigao.docthis)
- [koroFileHeader](https://marketplace.visualstudio.com/items?itemName=OBKoro1.korofileheader)
- [Project Manager](https://marketplace.visualstudio.com/items?itemName=alefragnani.project-manager)  
   git 项目的归类管理

### 提示

- [Code Spell Checker](https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker)
- [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
- [VersionLens](https://marketplace.visualstudio.com/items?itemName=pflannery.vscode-versionlens)

### 功能

- [GitLens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens)
- [Import Cost VSCode Extension](https://marketplace.visualstudio.com/items?itemName=wix.vscode-import-cost)
- [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer)
- [Markdown All in One](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one)
- [MDX](https://marketplace.visualstudio.com/items?itemName=silvenon.mdx)
- [REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client)
- [TODO+](https://marketplace.visualstudio.com/items?itemName=fabiospampinato.vscode-todo-plus)
- [TODO Highlight](https://marketplace.visualstudio.com/items?itemName=wayou.vscode-todo-highlight)
- [i18n Ally](https://marketplace.visualstudio.com/items?itemName=Lokalise.i18n-ally)

### AI

- [codeium](https://codeium.com/)
- [GitHub Copilot](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot)

## 快捷键

### control + option + command

用 openInTerminal 在当前目录打开 terminal  
`control + option + command + t`

用 openInTerminal 在当前目录打开 vscode  
`control + option + command + v`

用 openInTerminal 复制当前的文件地址  
`control + option + command + c`

### control + option

用 DropOver 打开一个空的文件暂存区域  
`alt + option + blank-space`

控制 Hidden Bar 的显示与否  
`alt + option + h`

### command + shift

打开剪贴板管理工具 Paste  
`command + shift + v`

使用 uPic 选择文件上传  
`command + shift + y`

使用 uPic 上传剪贴板的图片内容
`command + shift + u`

使用 uPic 选择裁剪区域并上传  
`command + shift + i`

### command

使用 input source pro 切换输入法搜狗中文  
`command + ]`

使用 input source pro 切换输入法英文  
`command + [`

### alt

打开 Alfred 全局搜索框  
`alt + blank-space`

## zshrc

```bash

export PATH=$HOME/bin:/usr/local/bin:$PATH
export PATH=/usr/local/Cellar/postgresql@9.6/9.6.19/bin:$PATH

export ZSH=$HOME/.oh-my-zsh


ZSH_THEME="ys"

# git clone https://github.com/paulirish/git-open.git
# git clone https://github.com/zsh-users/zsh-autosuggestions
# git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
# git clone https://github.com/agkozak/zsh-z
plugins=(
  git
  dirhistory
  git-open
  zsh-autosuggestions
  zsh-syntax-highlighting
  zsh-z
)

source $ZSH/oh-my-zsh.sh

prompt_context() {}

eval $(thefuck --alias)

# -------------------------------- #

alias clr='clear'
alias ..="cd .."

# -------------------------------- #

alias ga="git add"
alias gA="git add -A"


alias gck="git checkout"
alias gckb="git checkout -b"

alias go="git open"

alias gcl="git clone"
alias gclr="git checkout ./"
alias gp="git pull origin"
alias gps="git push origin"
alias gcm="git commit -m"
alias gch="git cherry-pick"

alias gsv="git stash save"
alias gsl="git stash list"
alias gsp="git stash pop"
alias gsclr="git stash clear"


# npm install -g czg
# czg --openai-token=sk-xxxxx

alias aicm="czg ai"

# -------------------------------- #

alias cfgzsh="open ~/.zshrc"
alias sourcezsh="source ~/.zshrc"

# -------------------------------- #

# where proxy
proxy () {
  export http_proxy="http://127.0.0.1:1087"
  export https_proxy="http://127.0.0.1:1087"
  echo "HTTP Proxy on"
}

# where noproxy
noproxy () {
  unset http_proxy
  unset https_proxy
  echo "HTTP Proxy off"
}
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion

```
