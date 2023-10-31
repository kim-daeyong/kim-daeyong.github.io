---
layout: post 
title: Querydsl count 결과 값 distinct
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Java, Querydsl]
categories : [Java]
toc: true
---


### TRY  
Querydsl에서 `count()` 할떄  `distinct()` 되지않았다.  
결과값은 10개가 나와야하는데 그 이상나왔다.  

### CATCH  
* 해결
    이럴땐
    ```Java
        .countDistinct()
    ```
    가 따로 있었다.  


### FINALLY  
QueryDsl!

끝

---
