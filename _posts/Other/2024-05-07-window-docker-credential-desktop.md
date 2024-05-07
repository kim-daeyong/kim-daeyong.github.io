---
layout: post 
title: "Windows 윈도우에서 docker-credential-desktop: executable file not found in PATH"S
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Windows, docker]
categories : [Other]
toc: true
---


### TRY  
윈도우 환경에서 WSL2로 Ubuntu 환경을 만들어 도커를 테스트 하는 도중 발생한 에러입니다.  
aws에 로그인하려 했으나 로그인이 안되는 문제가 발생했습니다.  

다음과 같은 에러가 발생하였습니다.

> docker-credential-desktop": executable file not found in %PATH%

![image](https://github.com/kim-daeyong/kim-daeyong.github.io/assets/45562285/cac5cbea-9f1c-4d7f-91a6-398c5ae50999)


### CATCH  

* 에러 해결 방법

~~~

C:\Users\[YOUR.USER]\.docker\config.json

~~~

`credsStore` 라고 되어 있는 것을 `credStore` 로 변경  

그 다음에 다시 로그인을 실행하니 정상동작 합니다.  

혹시 리눅스라면

~~~

$HOME/.docker/config.json

~~~

해당 경론에 있습니다.  


### FINALLY  
기존에 개발용으로 사용하던 mac os 혹은 linux os와 달리 windows os에서 개발은 다른점이 정말 많습니다.  
이런점을 인식하고 접근하는게 중요한 것 같습니다.  


끝

---
