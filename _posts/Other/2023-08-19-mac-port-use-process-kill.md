---
layout: post 
title: Mac port 사용중인 process kill
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [AWS, Shell, Infra, Other]
categories : [Other]
toc: true
---


### TRY  
가끔 mac에서  port를 사용하려 할때 누가 사용하고있는지 모를때가 있다.  
그때 삭제 하기위해..  

### CATCH  

```shell
# port 찾기
$ lsof -i :{port}


#예
lsof -i :8080

COMMAND  PID  USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
java    8186  user   65u  IPv6 0xf19e8675aba1dbcd      0t0  TCP *:http-alt (LISTEN)


# kill -9
$ kill -9 {PID}
```

### FINALLY  
간단하다.

끝

---
