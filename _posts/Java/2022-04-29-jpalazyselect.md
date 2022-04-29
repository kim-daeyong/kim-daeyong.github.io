---
layout: post
title: Spring Data JPA에서 연관관계 있는 엔티티 select 하기
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Jpa, spring]
comments : true
categories : [Programming]
---

## 상황

만약 다음과 같은 엔티티가 있다고 가정하자.  

공동구매..?  

```
@Entity
@Getter @Setter
public class Member {

    @Id @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    private String name;

    @Embedded
    private Address address;

    @ManytoOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    Order order;

}

```

그리고 해당 엔티티를 조회해본다면

```
// 1
List<Member> members = memberRepository.findAll();

// 2
members.get(0).getOrder.getClass();

// 3
members.get(0).getOrder.getName();
```

어떻게 될까??  


먼저 1번에서 select를 하여 멤버 리스트를 조회하고.  
2번에서 member에 해당하는 order의 getClass를 찍어보면 실제 객체가 아닌 하이버네이트 프록시 객체가 찍혀 있을 것이다.  
3번에서 getName을 요청하면 프록시 객체라 값이 없으므로 해당 Order에 대해 다시 Select문을 사용해 조회 실제 값으로 채운다.  


이처럼 각각의 member 리스트에 대한 조회를 한변 요청했을뿐인데 각각의 member의 order에 대해 추가 쿼리가 요청될 것이다.  
이 문제를 n+1의 문제라 하고 이를 방지하려면 어떻게 할까.  

---

## 이처럼 여러번 조회하지않고 한번에 조회하려면??

1. fetch를 이용하여 조회한다.

```
@Query("select m from Member m left join fetch m.order")
List<Member> findMemberFetch

```

member를 조회할때 연관된 order에 대한 걸 한번에 조회해온다.  
스프링 데이터 JPA에서 지원하는 method로 하고싶다면?  

2. @EntityGraph

```
@EntityGraph(attributePaths = {"order"})
List<Member> findByName(String name);

@Override
@EntityGraph(attributePaths = {"order"})
List<Member> findAll();
```

혹은 이런 것도 된다.  

```
@EntityGraph(attributePaths = {"order"})
@Query("select m from Member m")
List<Member> findMemberFetch

```

@EntityGraph 를 이용하면 편하게 fetch 할 수 있다.  

---