---
layout: post
title: custom-commands-in-zshrc
date: 2023-03-01
category: memo
---

```shell
### plugins
plugins=(git
	zsh-autosuggestions
	zsh-syntax-highlighting
	z
)


###terminal###
alias ll="ls -al --color=auto"

function fzfselect() { find * -type f | fzf > selected }

function killport() { kill -9 $(lsof -t -i :$1) }

###neofetch###
#Neofetch is a super-convenient command-line utility used to fetch system information within a few seconds.
neofetch

###git###
alias gtree="git log --pretty=oneline --graph --decorate --all"

alias gbDl="gcm && git branch | grep -v main | xargs git branch -D"

function gsync {
    git fetch origin
    git reset --hard origin/$(git branch --show-current)
}

###homebrew###
alias b="brew"

alias bs="brew services"

###python###
alias python="python3"
alias pip="pip3"

###postgres###
function fix-postgres()
{ rm -f /usr/local/var/postgres/postmaster.pid }

###iterm2###
bindkey "\e\e[D" backward-word
bindkey "\e\e[C" forward-word

###k8s###
alias k="kubectl"

# usage: kredeploy {deployment name}
function kredeploy () { kubectl patch deployment "$1" -p "{\"spec\":{\"template\":{\"metadata\":{\"annotations\":{\"date\":\"$(date +%s)\"}}}}}"}

function kcontext(){ kubectl config get-contexts }

function kkillpod() { kubectl delete pod $1 --grace-period=0 --force }

function kkillpodns() { kubectl delete pod $1 --grace-period=0 --force --namespace $2 }

function st() { stern -t --since $1 $2 }

function kbash() { kubectl exec -it $1 -c $2 bash }

function kportforward() { kubectl port-forward deployment/$1 $2:$3 }

function klocal() { k config use-context docker-desktop }
```