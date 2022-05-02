---
layout: post
title: @Transactional readOnly 옵션이 빠른 이유
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Jpa, spring]
comments : true
categories : [Programming]
---

## readOnly = true

기본적으로 트랙잭션안의 조회된 entity는 변경이 읽어나면 dirty checking에 의해 변겅이 감지되고 커밋될때 플러시된다.  
하지만 이 옵션을 사용하면 하이버네이트 세션의 플러시 모드를 MANUAL로 설정하여 entity를 수정해도 강제로 플러시를 호출하지 않는 한 플러시가 일어나지 않는다.   
따라서 트랜잭션을 커밋해도 영속성 컨텍스트를 플러시하지 않고 엔티티의 등록， 수정， 삭제는 동작하지 않는다.  
그래서 플러시할때 일어나는 스냅삿 비교와 같은 무거운 로직들이 수행되지않아 성능이 향상된다.  

---