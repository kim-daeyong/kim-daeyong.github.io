---
layout: post
title: 소스트리 사용법
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [SourceTree, git]
categories : [Other]
---

### 순서

1. 레포지토리 생성 및 콜라보, 클론등 설정 후 완료.   
2. 깃플로우 > 개발할 브랜치(develop) 생성(master는 배포용으로 하자!)
3. 깃플로우 > 새기능 생성(만들 기능으로 하자, 전에 했던 이름과 같은 이름으로하면 전에 꺼를 불러오게되니 이름은 다르게 해야한다.)
4. 코드 작성, commit
5. feature/~~~ 에 push
6. 깃레포짓에서 pullrequest 받고 develop에 merge
    - pullrequest창에서 베이스를 수정해야 한다.
7. pullrequest 받고 master에 merge(마스터에 merge안하면 깃 contribution에 안올라옴)
8. 소스트리에서 develop,master에서 pull 받는다.

---