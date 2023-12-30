---
layout: post 
title: QueryDsl fetchJoin()과 Paging시 applying in memory 경고
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Java, Spring, JPA, QueryDsl]
categories : [Programming]
toc: true
---


### TRY  
QueryDsl 에서 페이지를 조회할때 `applying in memory` 경고가 발생했다.  
내용은 다음과 같다.  

```
WARN  - org.hibernate.orm.query - firstResult/maxResults specified with collection fetch; applying in memory

```

### CATCH  

* 원인   
    이 경고의 원인은 `fetchJoin()`과 `Paging`을 같이 할때 발생한다고 한다.  
    이 경고의 뜻은 조건에 해당하는 모든 데이터를 가져와 메모리에서 페이징 처리를 하니까 경고를 해주는 것이다.  
    실제로 쿼리를 확인해보면 `limit`가 빠져있는걸 확인할 수 있다.  
    물론 페이징이 안되는 건 아니다.  
    하지만 데이터가 적다면 문제없겠지만 만약 데이터가 많아지면 서버의 부하가 심해질 것이다.  


* 이런상황을 미리 방지하기   
    설정파일에 다음과 같이 추가해주자.
    ```
        jpa:
            ...
            hibernate.query.fail_on_pagination_over_collection_fetch: true
    ```

    이렇게 해두면 만약 저 경고를 로그에 찍어줄 상황이 되면 다음과 같이 막아버린다.
    ```
    firstResult/maxResults specified with collection fetch. In memory pagination was about to be applied. Failing because 'Fail on pagination over collection fetch' is enabled.
    
    org.springframework.orm.jpa.JpaSystemException: firstResult/maxResults specified with collection fetch. In memory pagination was about to be applied. Failing because 'Fail on pagination over collection fetch' is enabled.

    Caused by: org.hibernate.HibernateException: firstResult/maxResults specified with collection fetch. In memory pagination was about to be applied. Failing because 'Fail on pagination over collection fetch' is enabled.
    ```

    로컬 혹은 테스트등의 프로필에 추가해주면 배포 전 확인할 수 있을 것이다.  

* 해결법  
    다양한 방법이 있겠지만 나는 fetchJoin()을 하지않은 대상 ids를 먼저 불러오고 In절을 통해 fetchJoin()하는 방식으로 바꿨다.  
    만약 Join된 테이블이 많거나 데이터양이 많다면 쿼리를 분리하거나 뭐 다양한 방법들이 있을 것이다.  

    * 기존 예제
        ```java
            JPAQuery<Member> contentQuery = factory.select(member)
                .from(member)
                .leftJoin(member.team, team).fetchJoin();


            List<Member> content = contentQuery
                .where(searchOption())
                .offset(page.getOffset())
                .limit(page.getPageSize())
                .orderBy(orderByOption())
                .distinct()
                .fetch();

        ```

    * 변경 예제
        ```java
            JPAQuery<Long> contentQuery = factory.select(member.id)
                .from(member)
                .leftJoin(member.team, team);

            List<Long> contentId = contentQuery
                .where(searchOption())
                .offset(page.getOffset())
                .limit(page.getPageSize())
                .orderBy(orderByOption())
                .distinct()
                .fetch();

            List<Member> content = from(member)
                .leftJoin(member.team, team).fetchJoin()
                .where(member.id.in(contentId))
                .orderBy(orderByOption())
                .fetch();
        ```


### FINALLY  
다양한 방법이 있을 것이다.

이는 한가지 방법일 뿐이다.  

끝

---
