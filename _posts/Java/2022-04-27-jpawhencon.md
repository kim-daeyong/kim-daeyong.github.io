---
layout: post
title: JPA가 언제 DB와 connection을 가질까
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Jpa]
comments : true
categories : [Programming]
---

## 언제 가질까

DB의 트랜잭션이 시작할때 JPA가 영속성 컨텍스트가 커넥션을 가져온다.

---

## 반납은?

spring.jpa.open-in-view(osiv) 가 켜져있다면 (기본적으로 true이다.) 트랜잭션이 서비스에서 끝나도 반환하지않다가(영속성 컨텍스트가 물고있다.) 요청에 대한 Response가 될때 영속성 컨텍스트와 함께 사라진다.  
이때문에 컨트롤러 단에서 지연로딩이 가능했던 것이다.  
이처럼 편하지만 실시간 트래픽이 많은 서비스에선 DB 커넥션이 다 소비될 수 있다.  


spring.jpa.open-in-view(osiv) 가 꺼져있다면 트랜잭션이 끝나면 영속성컨텍스트를 닫고 커넥션을 반환한다.  
이때문에 컨트롤러단에선 준영속성상태로 DB와 커넥션이 없기 때문에 지연로딩할 수 없다.  
지연로딩은 트랜잭션 내부에서 처리해야 한다.  

---