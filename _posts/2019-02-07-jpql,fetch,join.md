---
layout: post
title: JPQL, fetch, join?
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [SpringBoot, Jpa]
categories : [Spring]
---

* JPQL 특징

1. 테이블이 아닌 객체를 검색하는 객체지향 쿼리

2. SQL을 추상화 했기 때문에 특정 벤더에 종속적이지 않음

3. JPA는 JPQL을 분석하여 SQL을 생성한 후 DB에서 조회


* fetch join - 일반적인 조인은 연관관계는 고려하지않으나 fetch는 연관된 엔티티도 함께 조회한다.

JPQL fetch join 사용법
1) 공식문서 명령어 , [LEFT [OUTER] INNER] JOIN FETCH 조인경로

2) 엔티티 패치 조인

ex) select m from parent p join fetch p.child
연관된 엔티티나 컬렉션을 함께 조회한다. p와 p.child를 모두 검색한다.
참고로 일반적인 JPQL은 별칭을 허용하지 않는다.
하이버네이트는 페치 조인에도 별칭을 허용한다.

~~~
select
    parent0_.id as id1_1_0_,
    child1_.id as id1_0_1_,
    parent0_.child_id as child_id3_1_0_,
    parent0_.name as name2_1_0_,
    child1_.name as name2_0_1_ 
from
    Parent parent0_ 
inner join
    Child child1_ 
        on parent0_.child_id=child1_.id
~~~

3) 컬렉션 패치 조인

일 대 다 관계를 패치 조인 할 경우
ex) select p from Parent p join fetch p.child where p.name = ‘child_name’
연관된 child의 p.name의 값이 child_name과 동일한 모든값이 나온다
또한 fetch join으로 인해서 지연로딩이 발생하지 않는다.

~~~
select
    parent0_.id as id1_1_0_,
    child2_.id as id1_0_1_,
    parent0_.name as name2_1_0_,
    child2_.name as name2_0_1_,
    child1_.Parent_id as Parent_i1_2_0__,
    child1_.child_id as child_id2_2_0__ 
from
    Parent parent0_ 
inner join
    Parent_Child child1_ 
    on parent0_.id=child1_.Parent_id 
inner join
    Child child2_ 
    on child1_.child_id=child2_.id 
where
    parent0_.name=?
~~~
~~~
String name = "testName1";
Parent parents = em.createQuery("" +
    "SELECT p " +
    "FROM Parent p " +
    "JOIN FETCH p.child " +
    "where p.name = :name ", Parent.class) 
    .setParameter("name", name) // :name에 값 매핑
    .getSingleResult();
System.out.println(parents);
~~~
3. fetch join의 단점
페치 조인 대상에는 별칭을 줄 수 없다.
둘 이상의 컬렉션을 페치 할 수 없다.
컬렉션을 페치 조인하면 페이징 API를 사용 할 수 없다.
