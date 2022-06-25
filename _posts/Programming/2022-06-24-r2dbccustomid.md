---
layout: post
title: r2dbc에서 Custom id 사용하기.
subtitle: 
gh-repo: 
gh-badge: [star, fork, follow]
tags: [Mysql, R2dbc, Spring]
categories : [Programming]
---


### 문제 
* save 시 @Id 컬럼을 기준으로 이 객체가 isNew인지 아닌지 체크  
* repository의 기본 save를 이용 시 새로운 객체로 insert를 해야하는데 update를 하려고 하고, 이때 없는 데이터라 update가 되지않음.  

### 해결

```java
    @Table
    public class TestClass {

        @Id
        private Long id;

        private Long value;
    }
```

기본 클래스라고 치고..  

* 1번째 방법 예)
    - @Query에 insert문을 직접 사용하여 사용.

* 2번째 방법 예)
    ``` java
        @Table
        public class TestClass implements Persistable {

            @Id
            private Long id;

            private Long value;

            @Transient
            private boolean isNew;

            public Point(String id, Long value) {
                this.id = id;
                this.value = value;
                this.isNew = false;
            }

            @Override
            public Serializable getId() {
                return id;
            }

            @Override
            public boolean isNew() {
                return isNew;
            }
        }

    ```
    Persistable를 상속받아 getId()와 isNew() 를 @Override한다.  
    isNew()를 통해 새로운 엔티티인지 update를 해야하는지 확인하기 때문이다.  



---
