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
