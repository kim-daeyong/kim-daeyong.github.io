---
layout: post
title:  맥에서 파이썬 버전 설정
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Python, Mac]
categories : [Python]
---

맥에서 brewdhk pyenv를 통해 파이썬 버전을 3.6.8버전을 받았다.  
근데 파이썬 버전을 확인해보면 2.7.10 버전으로 되어있고 바뀌지 않았다.  

---

> 방법

맥은 기본적으로 파이썬이 설치되어있고 버전은 2.7.10 을 탑재하고있다.

이를 변경해주려면

~~~

Pyenv로 원하는 버전을 받아서  //pyenv는 파이썬 버전관리

vi ~/.zshrc

 eval "$pyenv init -)"

Source.profile 


~~~