---
layout: post
title: jpa의 distinct
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Jpa]
comments : true
categories : [Programming]
---

## Distinct

fetch join을 사용 시 중복된 데이터가 생성된다.  
왜냐면 db의 입장에선 fetch join도 그냥 join이기 때문에 join한 중복된 데이터가 나온다.  
이를 제거해줘야 하는데 그중 한가지 방법은 distinct가 있다.


DB의 distinct는 한 row과 같아 중복될때 제거해준다.  


하지만 jpa의 distinct는 쿼리에 distinct를 추가함과 더불어 entity의 id가 같을때(중복된 엔티티) 중복제거를 해준다.  


---