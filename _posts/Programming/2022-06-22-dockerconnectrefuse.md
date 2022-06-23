---
layout: post
title: Docker-compose로 구성된 컨테이너 connection refuse(m1)
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Mysql, Docker, Spring]
categories : [Programming]
---


### 문제 
* docker 컨테이너로 띄워둔 mysql에서 connection refuse가 난다
* 개발 환경은 M1 mac

### 해결

* 기존에 하던 것 처럼 먼저 host명을 compose에 정의한 mysql container의 이름으로 적었다.
    - 안됨 똑같이 connection refuse  
  
* MYSQL HOST, ROOT, port등 확인
    - 문제 없음   
  
* host를 container 명에서 localhost로 변경
    - local 개발환경에선 mysql container에 접속되지만 container안에선 안됨, 컨테이너 끼리만 안되는게 확실
  
* host를 host.docker.internal로 변경
    - datasource의 url을 다음과 같이 변경하였다.
    - url: r2dbc:mysql://host.docker.internal:13306/test url을 다음과 같이 변경하였다.

---

다른 환경에서도 동일한지 테스트는 안해봤다.

[참고](https://docs.docker.com/desktop/mac/networking/)
