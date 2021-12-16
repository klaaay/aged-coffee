---
title: zshrc
date: '2021-12-17'
tags: ['常用配置']
draft: false
summary: ''
---

```bash
export PATH=$HOME/bin:/usr/local/bin:$PATH
export PATH=/usr/local/Cellar/postgresql@9.6/9.6.19/bin:$PATH

export ZSH=$HOME/.oh-my-zsh


ZSH_THEME="ys"


plugins=(git)

source $ZSH/oh-my-zsh.sh

prompt_context() {}

eval $(thefuck --alias)

alias ga="git add -A"
alias gpull="git pull origin"
alias gpush="git push origin"
alias gclr="git checkout ./"
alias gcmit="git commit -m"

alias cfgzsh="open ~/.zshrc"
alias apyzsh="source ~/.zshrc"

alias clr='clear'

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
