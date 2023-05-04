---
title: shell-tool
date: '2022-01-29'
tags: ['资源整理']
draft: false
summary: ''
---

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
