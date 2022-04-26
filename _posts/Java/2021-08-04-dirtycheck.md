---
layout: post
title: jpa dirty checking
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [SpringBoot, JPA]
comments : true
categories : [Programming]
---

### 더티체킹이란 (dirty checking)

JPA는 엔티티 매니저가 엔티티의 저장, 조회, 삭제, 수정을 하는데 수정에 대한 메서드가 없습니다.

JPA 엔티티에 대한 변경을 dirty checking으로 체크 하여 자동으로 데이터 베이스에 update 합니다.

즉 따로 save() 하지않아도 변경을 감지하여 데이터베이스에 update 쿼리를 날려줍니다.

더티 체킹은 영속 상태에 있는 엔티티이고, Transaction 안에서 엔티티가 변경하는 경우 일어납니다.

---

@Transactional과 EntityTransaction를 이용하여 트랜잭션을 설정할 수 있는데

이와 같은 방법으로 트랜잭션을 사용할 경우 

엔티티 객체를 조회한 후, 필드 값을 변경하면 dirty checking이 되어 저장됩니다.

트랜잭션을 사용하지않은 경우 변경된 값은 저장되지않습니다.

이처럼 트랜잭션을 사용하지않았을때 save() 나 saveAndFlush를 사용하여 저장해줍니다.

---

update 쿼리를 확인하면 모든 필드에 대해 업데이트를 실행합니다.

JPA가 전체 필드를 업데이트 하는 방식이 default 이기 때문입니다.

이를 방지하려면 엔티티에 @DynamicUpdate를 붙여 변경 필드만 update 되도록 변경할 수 있습니다.

---

