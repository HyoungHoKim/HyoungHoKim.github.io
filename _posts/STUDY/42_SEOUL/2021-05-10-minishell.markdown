---
layout: post
title:  "42_minishell"
date:   2021-05-10 15:20:21 +0900
categories: [STUDY, STUDY/42_SEOUL]
---

나만 볼려고 적는 minishell 정리!

### 환경변수 
이번 과제에서는 환경변수도 처리할 수 있어야한다. 기존의 환경변수를 처리할 수 있어야하고, 새로운 환경변수도 추가할 수 있어야한다. 그러려면 첫번째 환경변수를 받고 초기화할 수 있어야한다. 

1. 환경변수 받기 

환경변수를 받는 법은 쉽다. main문애 매개변수를 하나 추가하면 된다. 원래 써왔던 2개의 매개변수 이외에도, char **envp를 붙이면 문자열 형태로 환경 변수를 받을 수 있다. 받은 환경변수는 아래와 같이 envp에 입력된다.
```
USER=hyoukim
__CFBundleIdentifier=com.microsoft.VSCode
COMMAND_MODE=unix2003
LOGNAME=hyoukim
PATH=/usr/local/opt/llvm/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/opt/llvm/bin
SSH_AUTH_SOCK=/private/tmp/com.apple.launchd.xt0y8YA8Qj/Listeners
SHELL=/bin/zsh
HOME=/Users/hyoukim
__CF_USER_TEXT_ENCODING=0x1F5:0x3:0x33
TMPDIR=/var/folders/63/33s2c4t546n_6h4ckyzfrp3h0000gn/T/
XPC_SERVICE_NAME=0
XPC_FLAGS=0x0
ORIGINAL_XDG_CURRENT_DESKTOP=undefined
SHLVL=1
...
```

### Parsing
